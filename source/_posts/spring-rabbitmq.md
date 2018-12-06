---
title: "Spring MVC整合RabbitMQ实现延迟队列"
date: 2017-11-10
tags: RabbitMQ
---

> 这是一个Spring MVC整合RabbitMQ实现延迟队列的例子，包含可运行的完整代码。之前自己摸索了很久，希望能帮到你。

<!-- more -->

## 延迟队列简介
简单来说，延迟队列就是一个设置了等待时间的队列，消息到达队列后，会等待一段时间后再执行其他操作。主要的应用场景有订单超时取消、超时自动评价等等。

## 实现原理
RabbitMQ给我们提供了TTL（Time-To-Live）和DLX （Dead-Letter-Exchange）这两个特性，使用RabbitMQ实现延迟队列利用的正是这两个特性。TTL用于控制消息的生存时间，如果超时，消息将变成Dead Letter。DLX用于配置死信队列，可以通过配置x-dead-letter-exchange和x-dead-letter-routing-key这两个参数来指定消息变成死信后需要路由到的交换器和队列。关于TTL（Time-To-Live）和DLX （Dead-Letter-Exchange）的详细介绍可以参考文末的参考链接。

## 代码示例
简单介绍一下，项目首先引入Spring与RabbitMQ相关的依赖，然后在spring-rabbitmq.xml中配置一个常规队列（delay_queue）和一个死信队列（task_queue），在常规队列上配置x-message-ttl和x-dead-letter-exchange等参数指向死信队列。最后配置任务处理器（delayTask），并将任务处理器与死信队列绑定并配置到监听器上。然后就可以开开心心地实现业务逻辑了~

项目中有两个Service，ProducerService和ConsumerService，导入项目后可以直接运行这两个类的main方法查看延迟队列的效果。也可以部署到本地服务器后运行web项目来测试，但是测试中只使用到了ProducerService。

## 引入RabbitMQ
```xml
        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-amqp</artifactId>
            <version>2.0.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.amqp</groupId>
            <artifactId>spring-rabbit</artifactId>
            <version>2.0.0.RELEASE</version>
        </dependency>
```

## 配置
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/rabbit
           http://www.springframework.org/schema/rabbit/spring-rabbit.xsd
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="connectionFactory" class="org.springframework.amqp.rabbit.connection.CachingConnectionFactory">
        <property name="host" value="localhost"/>
        <property name="port" value="5672"/>
        <property name="username" value="guest"/>
        <property name="password" value="guest"/>
    </bean>

    <rabbit:admin connection-factory="connectionFactory"/>

    <rabbit:queue name="delay_queue" auto-declare="true">
        <rabbit:queue-arguments>
            <entry key="x-message-ttl" value="5000" value-type="java.lang.Long" />
            <entry key="x-dead-letter-exchange" value="exchange_delay" />
            <entry key="x-dead-letter-routing-key" value="task_queue" />
        </rabbit:queue-arguments>
    </rabbit:queue>

    <rabbit:queue name="task_queue" auto-declare="true"/>
    <rabbit:direct-exchange name="exchange_delay" durable="false" auto-delete="false" id="exchange_delay">
        <rabbit:bindings>
            <rabbit:binding queue="task_queue" />
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory" queue="delay_queue" routing-key="delay_queue"/>

    <!-- 配置线程池 -->
    <bean id ="taskExecutor"  class ="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor" >
        <!-- 线程池维护线程的最少数量 -->
        <property name ="corePoolSize" value ="5" />
        <!-- 线程池维护线程所允许的空闲时间 -->
        <property name ="keepAliveSeconds" value ="30000" />
        <!-- 线程池维护线程的最大数量 -->
        <property name ="maxPoolSize" value ="1000" />
        <!-- 线程池所使用的缓冲队列 -->
        <property name ="queueCapacity" value ="200" />
    </bean>

    <bean id="delayTask" class="com.shenofusc.task.DelayTask" />

    <!-- Queue Listener 当有消息到达时会通知监听在对应的队列上的监听对象-->
    <rabbit:listener-container connection-factory="connectionFactory" acknowledge="auto" task-executor="taskExecutor">
        <rabbit:listener queues="task_queue" ref="delayTask"/>
    </rabbit:listener-container>

</beans>
```

## DelayTask
```java
public class DelayTask implements MessageListener {
    public void onMessage(Message message) {
        try {
            String receivedMsg = new String(message.getBody(), "UTF-8");
            System.out.println("Received : " + receivedMsg);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

## ProducerService
```java
@Service
public class ProducerService {

    @Resource
    private AmqpTemplate amqpTemplate;

    public void send(String msg) {
        amqpTemplate.convertAndSend(msg);
        System.out.println("Sent: " + msg);
    }

    public static void main(String[] args) {
        ApplicationContext context = new GenericXmlApplicationContext("classpath:/spring-rabbitmq.xml");
        AmqpTemplate amqpTemplate = context.getBean(AmqpTemplate.class);
        amqpTemplate.convertAndSend("Hello World");
        System.out.println("Sent: Hello World");
    }

}
```

## ConsumerService
```java
@Service
public class ConsumerService {

	@Resource
	private AmqpTemplate amqpTemplate;

	public void recive() {
		System.out.println("Received: " + amqpTemplate.receiveAndConvert());
	}

    public static void main(String[] args) {
		ApplicationContext context = new GenericXmlApplicationContext("classpath:/spring-rabbitmq.xml");
		AmqpTemplate amqpTemplate = context.getBean(AmqpTemplate.class);
		System.out.println("Received: " + amqpTemplate.receiveAndConvert());
	}

}
```

## 完整代码
[Spring+RabbitMQ实现延迟队列](https://github.com/shenofusc/spring-rabbitmq-delay-queue)

## 注意事项
修改x-message-ttl参数后重启项目会报错，删除相关队列可以解决这个问题，系统启动时会自动重建队列。

## 参考资料
[RabbitMQ - Time-To-Live Extensions](https://www.rabbitmq.com/ttl.html)  
[RabbitMQ - Dead Letter Exchanges](https://www.rabbitmq.com/dlx.html)  
[Spring-AMQP With XML Configuration](https://docs.spring.io/spring-amqp/docs/2.0.0.RELEASE/reference/html/_introduction.html#_with_xml_configuration)  
[Spring-AMQP Sample Applications](https://docs.spring.io/spring-amqp/docs/2.0.0.RELEASE/reference/html/_reference.html#sample-apps)  
[RabbitMQ如何实现延迟队列？](http://blog.csdn.net/u013256816/article/details/55106401)  
[Spring-AMQP整合RabbitMQ消费者配置和代码](https://my.oschina.net/91jason/blog/488126)  
[使用Dead Letter(死信队列)进行延时发送](http://blog.csdn.net/qq315737546/article/details/66475743)  