# apama-solace-sample
Sample Apama project and Java project to demostrator JMS connection from Solace to Apama using Apama's Correlator-Integrated Messaging for JMS.

![Sequence Diagram](resources/solace2apama.png)



An Apama project with JMS bundle is created to receive messages from Solace VMR.

An Apama project is created using Software AG Designer. 

##Solace JCSMP project
- Create a session independent TextMessage - In a Session independent message ownership model, client applications can reuse messages between send operations. Messages are allocated on‑demand and are disposed explicitly by client applications when they are done with the messages.

        TextMessage msg = JCSMPFactory.onlyInstance().createMessage(TextMessage.class);

- Create a Structured Data Map for message properties

        SDTMap map = prod.createMap();

- Add Apama event name "com.solace.sample.SampleTextMessage" in MESSAGE_TYPE

        map.putString("MESSAGE_TYPE", "com.solace.sample.SampleTextMessage");
        
- Add additional message properies
 
        map.putLong("MESSAGE_CREATED", System.currentTimeMillis());
        
- Send off the message to a topic destination

        prod.send(msg, topic);

```java

    SDTMap map = prod.createMap();
    map.putString("MESSAGE_TYPE", "com.solace.sample.SampleTextMessage");
    TextMessage msg = JCSMPFactory.onlyInstance().createMessage(TextMessage.class);
    for (int msgsSent = 0; msgsSent < 1000000; ++msgsSent) {

        msg.reset();
    
        msg.setText("msg count is " + String.format("%05d", msgsSent));
        msg.setDeliveryMode(DeliveryMode.DIRECT);
        msg.setApplicationMessageId("appID-" + msgsSent);
        msg.setElidingEligible(true);
        map.putLong("MESSAGE_CREATED", System.currentTimeMillis());
    
        msg.setProperties(map);
        prod.send(msg, topic);
        Thread.sleep(500);
    }
```

##Solace router configuration

**Creating a Message VPN**

This section outlines how to create a message-VPN called “apama” on the Solace Message Router with authentication disabled and 200MB of message spool quota for Guaranteed Messaging. This VPN name is required in the Apama configuration when connecting to the Solace message router. Actual values for authentication, message spool and other message-VPN properties may vary depending on the end application’s use case.

```text
(config)# create message-vpn apama
(config-msg-vpn)# authentication
(config-msg-vpn-auth)# user-class client
(config-msg-vpn-auth-user-class)# basic auth-type none
(config-msg-vpn-auth-user-class)# exit
(config-msg-vpn-auth)# exit
(config-msg-vpn)# no shutdown
(config-msg-vpn)# exit
(config)#
(config)# message-spool message-vpn apama
(config-message-spool)# max-spool-usage 200
(config-message-spool)# exit
(config)#
```

**Setting up Solace JNDI References**

To enable the JMS clients to connect required by Apama, there is one JNDI object required on the Solace Message Router:

- A connection factory: /jms/cf/apama

```text
(config)# jndi message-vpn apama
(config-jndi)# create connection-factory /jms/cf/apama
(config-jndi-connection-factory)# property-list messaging-properties
(config-jndi-connection-factory-pl)# property default-delivery-mode persistent
(config-jndi-connection-factory-pl)# exit
(config-jndi-connection-factory)# property-list transport-properties
(config-jndi-connection-factory-pl)# property direct-transport false
(config-jndi-connection-factory-pl)# property "reconnect-retry-wait" "3000"
(config-jndi-connection-factory-pl)# property "reconnect-retries" "3"
(config-jndi-connection-factory-pl)# property "connect-retries-per-host" "5"
(config-jndi-connection-factory-pl)# property "connect-retries" "1"
(config-jndi-connection-factory-pl)# exit
(config-jndi-connection-factory)# exit
(config-jndi)# no shutdown
(config-jndi)# exit
(config)#
```

**Configuring Client Usernames & Profiles**

This section outlines how to update the default client-profile and how to create a client username for connecting to the Solace Message Router. For the client-profile, it is important to enable guaranteed messaging for JMS messaging, endpoint creation and transacted sessions if using transactions.

The test client username of “apama_user” will be required by the Apama when connecting to the Solace Message Router.

```text
(config)# client-profile default message-vpn apama
(config-client-profile)# message-spool allow-guaranteed-message-receive
(config-client-profile)# message-spool allow-guaranteed-message-send
(config-client-profile)# message-spool allow-guaranteed-endpoint-create
(config-client-profile)# message-spool allow-transacted-sessions
(config-client-profile)# exit
(config)#
(config)# create client-username apama_user message-vpn apama
(config-client-username)# acl-profile default   
(config-client-username)# client-profile default
(config-client-username)# no shutdown
(config-client-username)# exit
(config)#
```

**Apama JMS properties**

The Apama correlator-integrated messaging for JMS configuration consists of a set of XML files and .properties files.

A correlator that supports JMS has the following two files:

- jms-global-spring.xml
- jms-mapping-spring.xml

In addition, for each JMS connection added to the configuration, there will be an additional XML and .properties file :

- connectionId-spring.xml
- connectionId-spring.properties

The property file has all the custom settings and values in each use case.

```properties
connectionFactory.jndiName.solace=/jms/cf/apama
jndiContext.environment.solace=java.naming.factory.initial\=com.solacesystems.jndi.SolJNDIInitialContextFactory\njava.naming.provider.url\=${jndiContext.environment.provider.url.solace}\n
jndiContext.environment.provider.url.solace=smf\://192.168.4.167
jndiContext.jndiAuthentication.username.solace=apama_user@apama
jndiContext.jndiAuthentication.password.solace=
connectionAuthentication.username.solace=
connectionAuthentication.password.solace=
clientId.solace=
staticReceiverList.solace=topic\:apamaTopic;
defaultReceiverReliability.solace=BEST_EFFORT
defaultSenderReliability.solace=BEST_EFFORT
JMSProviderInstallDir.solace=C\:/tools/SoftwareAG/common/lib
classpath.solace=libs/sol-common-10.0.1.jar;libs/sol-jcsmp-10.0.1.jar;libs/sol-jms-10.0.1.jar;
```

**Sample Apama event definition**

```go
event SampleTextMessage {
    string payload;
    dictionary<string, string> extraParam;
}
```

**Apama Event Processing Language source**

```go
monitor SampleTopicReceiver {
    action getSolaceMessage() {
        on AppStarted() {
            JMS.onApplicationInitialized();
        }
    }
    action onload () {
        getSolaceMessage();
        on JMSReceiverStatus() as receiverStatus
        {
            log "Received receiverStatus from JMS: " + receiverStatus.toString() at INFO;
            on all SampleTextMessage() as sampleTextMessage {
                log "From JMS: " + sampleTextMessage.toString() at INFO;
            }
        }
        route AppStarted();
    }
}
```
**Apama JMS receiver properties**

Some of the JMS sender/receiver properties can be toggled during development to debug application and exam JMS messages and performance matrix
 
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

## License

This project is licensed under the Apache License, Version 2.0. - See the [LICENSE](LICENSE) file for details.


## Resources

For more information try these resources:

- The Solace Developer Portal website at: http://dev.solace.com
- Get a better understanding of [Solace technology](http://dev.solace.com/tech/).
- Check out the [Solace blog](http://dev.solace.com/blog/) for other interesting discussions around Solace technology
- Ask the [Solace community.](http://dev.solace.com/community/)
