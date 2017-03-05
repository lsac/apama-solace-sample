# apama-solace-sample

```java
SDTMap map = prod.createMap();
map.putString("MESSAGE_TYPE", "com.solace.sample.SampleTextMessage");
final String startTime = "" + System.currentTimeMillis();
TextMessage msg = JCSMPFactory.onlyInstance().createMessage(TextMessage.class);
for (int msgsSent = 0; msgsSent < 1000000; ++msgsSent) {

  msg.setText("Sending timestamp is " + startTime + ", msg count is " + String.format("%05d", msgsSent));
  msg.setDeliveryMode(DeliveryMode.DIRECT);
  msg.setApplicationMessageId("appID-" + msgsSent);
  msg.setElidingEligible(true);

  msg.setProperties(map);
  prod.send(msg, topic);
  Thread.sleep(500);
}
```

```go
event SampleTextMessage {
	string payload;
	dictionary<string, string> extraParam;
}
```

```xml
<bean id="globalReceiverSettings" class="com.apama.correlator.jms.config.JmsReceiverSettings">
		
  <property name="receiverFlowControl" value="false"/>

  <!-- These logging options are for testing/diagnostics only and should 
    not be enabled in a production system due to the possible 
    performance impact 
  -->
  <property name="logJmsMessages" value="false"/>
  <property name="logJmsMessageBodies" value="true"/>
  <property name="logProductMessages" value="false"/>

  <property name="logPerformanceBreakdown" value="true"/>
  <property name="logDetailedStatus" value="false"/>
</bean>
```
