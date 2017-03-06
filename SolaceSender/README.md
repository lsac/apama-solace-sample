# Solace Message Sender

This Java project is a sample project to demonstrate sending messages using Solace JCSMP libraries. The messages are eventually received and mapped into Apama events for real-time complex event processing.  

**Build requirement:**

1. Java 8
2. Eclipse Neon with Gradle support
3. CLI Gradle version 3 or above

**Build procedure:**

1. Clone from GIT using CLI or Eclipse 
2. Import the project into Eclipse
3. Run Gradle 'build task in Eclipse to do a build cycle 
4. Run Gradle 'eclipse' task to set up Eclipse project nature and refresh the project
5. Set up 'run' configuration to have Solace VMR IP address as argument and run
6. Log messages start scrolling in console 

**Log shows received original the solace message:** 

```text
Receiving latency : 1
Received message: 
Destination:                            Topic 'apamaTopic'
AppMessageID:                           appID-8224
Priority:                               0
Class Of Service:                       USER_COS_1
DeliveryMode:                           DIRECT
Message Id:                             8230
Eliding Eligible
User Property Map:                      2 entries
  Key 'MESSAGE_CREATED' (Long): 1488757225657
  Key 'MESSAGE_TYPE' (String): com.solace.sample.SampleTextMessage

Binary Attachment:                      len=21
  1c 15 6d 73 67 20 63 6f    75 6e 74 20 69 73 20 30    ..msg.count.is.0
  38 32 32 34 00                                        8224.


msg count is 08224
```


