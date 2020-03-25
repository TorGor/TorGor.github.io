---
layout: post
title:  Springcloud + RocketMQ 解决分布式事务 
date:   2020-3-2 00:00:00 +0800
categories: document
tag: lock
---

>开篇思考
1. 为什么要分布式事务？
2. 分布式事务有哪些方式？
3. 分布式哪些环节会出问题？出了问题怎么应对？

# 站在巨人的肩膀观察和思考
随着互联网时代的高速发展，分布式成了大型系统的标配，这是时代发展的选择。大型分布式系统不是每个公司和开发人员都能够涉及的领域，因为大型系统后面都
隐藏着众多代名词：复杂，昂贵，高科技，人才云集，大战略。。。
大部分领头互联网公司甚至依托自己的分布式经验逐步建立自己的体系，并使用这套体系搭建自己的平台对内，甚至对外提供服务，
就像现在众多的云平台提供的服务，甚至有些大战略提出促进发展：大中台小前台、大炮台支援单兵作战等等。

这里提到了中台的概念，这个概念很广，都是以用户为中心的，分布式只是其中的一小部分运用，之所以强行和分布式挂钩，是想说明现在的发展趋势变了，
我们的眼界是有限的，但是完全可以站在巨人的肩膀上，利用他们的高度来提升自己的眼界，来思考，我们到底应该怎么做，怎么做适合
我们自身发展。我认为思考能力永远是一个程序员魅力所在，善于发现和思考，能够不断的帮助我们提升。

![中台齿轮图](https://torgor.github.io/styles/images/distribute/zhongtai-chiluntu.png)


# 微服务架构的优势和问题
优势：
* 扩展性强：可根据业务需要增加服务，不影响现有的服务架构
* 单一隔离：服务和服务之前通过远程调用，单个服务只提供单一职责的功能，开发人员可以一人一服务进行开发，互不影响
* 高可用：服务可以集群部署，单个应用失败通过重试和熔断等措施可以提高稳定性
* 技术选型灵活：可以根据团队特点、业务需求选择合适的技术栈，提高开发效率
* 突破性能瓶颈：可以集群方式部署，解决单体访问峰值等处理能力的性能瓶颈
* 降低运维成本：应对不同的场景，例如双十一，可以动态增减服务集群数量，做到按需付费，减少开支

问题：
* 复杂度高：整体架构设计十分复杂，需要考虑到整体性、经济性、稳定性
* 容易混肴服务界限：哪些服务需要划分微服务，哪些可以作为子模块，需要认真思考
* 难以确保一致性：因为是链路调用，一个请求会经过多个服务，难以确保执行结果达到一致性


# CAP & BASE 
因为写过这两个理论的具体介绍，这里简单再说明下。还有疑问的可以看介绍
[链接](https://mp.weixin.qq.com/s?__biz=MzA5MjM3ODE5MQ==&mid=2247483695&idx=1&sn=fb84901b9a63b17385d0ca5886bee2eb&chksm=906f440fa718cd1956c575cc156e8374bf028a8098ba24181a292cc7b86202d65453ecbff95b&token=2138559086&lang=zh_CN#rd)

![distribute-cap](https://torgor.github.io/styles/images/distribute/cap-circle.png) 

这两是分布式架构发展至今形成的理论基础，只要分布式都绕不开的原则。CAP 教会我们如何在设计的时候取舍，是要高可用，还是强一致性？
这个看业务的具体情况和需求分析，如果是关于资金的流转的，
例如：银行转账系统，肯定会要求保证一致性的场景更多，钱嘛，你懂得，不能多也不能少，少了错了就是大事。
如果只是一些简单的商品查询，可能用户更希望是可用性，也就是我随时查看都可以看到商品，但是商品信息可以在短时间内不一致。

也就是任何的架构设计，都是根据业务场景来分析的。通常 CAP 理论不够完善明确具体实现，这时候就可以用到 BASE 理论，简单的说就是
通过基本可用和软状态，来达到最终一致性的状态。

举个例子：秒杀服务，商品秒杀成功，商品库存 -1，但是订单入库需要调用订单服务，但是我们必须要先执行减库存操作成功后再执行，
这个时候可以先提交，然后发送到可靠消息系统 MQ，MQ 发送到订单服务进行入库操作，如果入库失败则进行重试

这里有个中间状态，库存信息入库而订单服务可能失败或者进行重试状态，这里要么重试成功，最终订单成功入库；要么彻底失败，回滚库存
入库信息改为初始状态。

上面的整体流程基本就是下面介绍的基于 RocketMQ 进行的分布式事务流程。


# 基于可靠消息（RocketMQ）最终一致性方案
![distribute-cap](https://torgor.github.io/styles/images/distribute/rocketmq-transaction-flow.png) 

RocketMQ 是一个来自阿里巴巴的分布式消息中间件，于 2012 年开源。
​RocketMQ 事务消息设计则主要是为了解决 Producer 端的消息发送与本地事务执行的原子性问题，RocketMQ 的设计中 broker 与
producer 端的双向通信能力，使得 broker 天生可以作为一个事务协调者存在；而 RocketMQ 本身提供的存储机制为事务消息提供了
持久化能力；RocketMQ 的高可用机制以及可靠消息设计则为事务消息在系统发生异常时依然能够保证达成事务的最终一致性。

## 使用场景分析
流程举例（中国银行 -- 转账 -- 人民银行）：
1. 中国银行要扣款 - 100 ,准备 HalfMsg，消息中携带中国银行 - 100 的信息
2. 中国银行 HalfMsg 成功发送后，执行数据库本地事务，在自己的库中 - 100 
3. 查看本地事务执行情况，成功则 commit 消息；失败则回滚不发送消息
4. 人民银行系统订阅了 MQ，MQ 确保消息发送到人民银行系统
5. 人民银行系统执行本地事务，数据库中 + 100 

问题分析：
1. half 消息没有发送成功：不会进入本地事务执行逻辑，且 MQ 没有消息
2. 本地事务失败，rollback half 消息：MQ 只有 half 消息，不会被其他服务消费，此时可以回滚清除消息
3. 本地事务成功，half 消息 commit 失败：MQ 定时查询本地事务状态，本地事务成功则继续 commit MQ 消息
4. 消费者消费 MQ 消息失败：事务没有执行，或者事务执行失败，基于 ACK 的重试机制重试，知道消费成功，如果还是不行，记录日志持久化，预警查看问题是执行回滚还是更新

## 如何在项目中使用
以上主要流程都是 RocketMQ 实现，对用户使用来说，用户需要实现的部分是：①本地事务执行 ；②本地事务回查方法
因此代码中的实现只需关注本地事务的执行状态即可。下面贴出具体实现方式

### pom 依赖
springcloud pom 的基础上，添加依赖，完整的 springcloud alibaba pom 可以参考
[这里](https://github.com/TorGor/seckill/blob/master/nacos-consumer-one/pom.xml) 

```xml

        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-stream-rocketmq</artifactId>
        </dependency>

```

### 模块添加

```java

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
@MapperScan("com.holy.nacosconsumerone.dao")
@EnableBinding({ MySource.class })
public class NacosConsumerOneApplication {

    public static void main(String[] args) {
        SpringApplication.run(NacosConsumerOneApplication.class, args);
    }
}
```

mysource
```java
public interface MySource {

		@Output("output1")
		MessageChannel output1();

		@Output("output2")
		MessageChannel output2();

		@Output("output3")
		MessageChannel output3();

		@Output("output4")
		MessageChannel output4();

	}
```

sendservice 
```java
/**
 * @author
 */
@Service
public class SenderService {

	@Autowired
	private MySource source;

	public void send(String msg) throws Exception {
		source.output1().send(MessageBuilder.withPayload(msg).build());
	}

	public <T> void sendTransactionalMsg(T msg, int num) throws Exception {
		MessageBuilder builder = MessageBuilder.withPayload(msg)
				.setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON);
		builder.setHeader("tx-header-num", String.valueOf(num));
		builder.setHeader(RocketMQHeaders.TAGS, "binder");
		Message message = builder.build();
		source.output2().send(message);
	}

}
```

划重点 ： MQ 事务监听类实现 ``RocketMQLocalTransactionListener``

```java
import org.apache.rocketmq.spring.annotation.RocketMQTransactionListener;
import org.apache.rocketmq.spring.core.RocketMQLocalTransactionListener;
import org.apache.rocketmq.spring.core.RocketMQLocalTransactionState;
import org.springframework.messaging.Message;

/**
 * 对需要使用分布式事务的消息发送接口监听
 * 根据事务消息分组来致性
 * ①本地事务先执行，根据业务情况执行提交、回滚操作
 * ②本地事务回查
 *
 * @author
 */
@RocketMQTransactionListener(txProducerGroup = "myTxProducerGroup", corePoolSize = 5,
        maximumPoolSize = 10)
public class TransactionListenerImpl implements RocketMQLocalTransactionListener {

    /**
     * 执行本地事务
     * ①事务执行成功，commit
     * ②事务执行失败，rollback
     * ③回查发送消息，unknown
     *
     * @param msg
     * @param arg
     * @return
     */
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
        Object num = msg.getHeaders().get("tx-header-num");
        try {
            // 本地业务代码，事务执行
            if ("1".equals(num)) {
                System.out.println("executer: " + new String((byte[]) msg.getPayload()) + " unknown");
                return RocketMQLocalTransactionState.UNKNOWN;
            } else if ("2".equals(num)) {
                throw new Exception("Exception for RocketMQ rollback");
            }
            System.out.println("executer: " + new String((byte[]) msg.getPayload()) + " commit");
            return RocketMQLocalTransactionState.COMMIT;
        } catch (Exception e) {
            System.out.println(e.getMessage());
            System.out.println("executer: " + new String((byte[]) msg.getPayload()) + " rollback");
            return RocketMQLocalTransactionState.ROLLBACK;
        }

    }

    /**
     * 执行本地事务回查，当状态为 UNKNOW 会执行这个方法，回查间隔时间差不多一分钟。
     *
     * 业务代码用来检查事务当前状态，是否执行完成，如果完成就执行 COMMIT
     *
     * @param msg
     * @return
     */
    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
        // 检查本地事务
        System.out.println("check: " + new String((byte[]) msg.getPayload()));
        return RocketMQLocalTransactionState.COMMIT;
    }

}
```

controller 测试：
```java
/**
 * @author holy
 */
@RestController
@RequestMapping("/rocketMQ")
public class RocketMQController {

	@Resource
	private SenderService senderService;

	@GetMapping(value = "/transactionMsg")
	public Object rocketMQTX(int num,String msg) {
		try {
			senderService.sendTransactionalMsg(msg,num);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return "OK" + num;
	}

}
	
```
配置文件：
```yaml
spring:
    application:
        name: service-consumer
    cloud:
        stream:
          bindings:
              output1:
                  content-type: application/json
                  destination: test-topic
              output2:
                  content-type: application/json
                  destination: TransactionTopic
              output3:
                  content-type: text/plain
                  destination: pull-topic
          rocketmq:
              bindings:
                  output1:
                      producer:
                          group: binder-group
                          sync: true
                  output2:
                      producer:
                          group: myTxProducerGroup
                          transactional: true
                  output3:
                      producer:
                          group: pull-binder-group
              binder:
                name-server: 192.168.244.89:9876
```

# 后续思考
> 本地事务复杂，执行查询时间太久如何处理？

针对不同的情况，其实我们只要认真分析场景，自然可以设计出对应的解决办法。
有些时候我们更希望通过一张事务执行情况表来判断事务的整体实行情况，比如业务比较复杂的时候，需要更新很多的表信息，这时候
使用事务表根据 TransactionID 来记录事务执行情况反而更贴合实际的使用场景。

> 如果我不是用 rocketMQ ，可以通过其他的 MQ 来实现分布式事务处理吗？

其实这个也是没有问题，大体思路就是封装新服务，专门用来检查事务执行情况，根据事务状态来决定是否发送消息到 MQ。

> 有好的建议欢迎在下方留言，大家一起讨论一起进步。优秀的评论会被选出赠送奖励


# 喜欢文章请关注我
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)








