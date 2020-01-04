---
layout: post
title: "Message level signing and encryption in soap web services!"
date: 2016-8-24 00:00:00 +0400
description: # Add post description (optional)
img: soap_signing_encryption.jpg # Add image post (optional)
tags: [soap, webservices, java, signing, encryption]

---
Recently I was involved in a project which I had to develop a soap web service. Web service development with maven and apache cxf sounds fun but the mysterious part was the message-level signing and encryption.

After a week searching and studying, finally I did the job and decided to write a tutorial about it.
In this tutorial I’m going to do some coding by using intellij, apache cxf, spring and Maven to develop the web service.
The code of the projects are available in Github and I just point out important parts of the projects.

First of all I’m going to describe the process of signing, encryption, verification and decryption for server side of the web service (actually it’s the same for the client) and then we will take a look at the configurations needed to implement webservice.

## Introduction
In order to apply security for exchanging data, three requirements have to be addressed:

First one is the Confidentiality and it means that only expected receiver has access to the message content (sender sends a message to the receiver, message is encrypted by the receiver’s public key and can be decrypted only by the receiver’s private key).

Second one is the Integrity, by integrity we make sure message will be received by intended recipient without any modification(the message signature (encrypted(by senders private key) hash value of the message) will be sent to the receiver, receiver generates message hash value with the same algorithm and compares it with the decrypted(by sender public key) signature).

And the last one is Authenticity. Authenticity’s purpose is to verify the originality of the message, it means authenticity makes sure that the message is from the source it claims to be from (sender sends a message to the receiver, message is encrypted by the sender’s private key and can be decrypted only by the sender’s public key, so the receivers conclude that the message is from the sender).

The security which is going to be implement is based on asymmetric-encryption, each party has its own key pair, a private and a public key, the public key as its name indicates is public and everyone has to have it while private key just belongs to the owner of the key pair so two key pairs are need, one for the server and another for the client.

## Signing, Encryption, Verification and Decryption

To implement security these four processes have to be applied to the message that is going to be transferred.
The sender of the message signs and encrypts the message and the receiver decrypts and verifies it.

In signing process sender puts the message signature alongside with it in order to enable the receiver to verify the message origin and receiver at the first place must decrypt the signature with its private key and gets the hash value of the message.

In the second step the hash value of the message is generated utilizing the same algorithm used by sender to generate the hash value of the message and finally receiver compares two hash values (the one that was decrypted and the one was generated), if they’re the same, signature is verified otherwise it can conclude that message is not from the source it claims to be.

### Sing-Verify

![Sign-verify]({{site.baseurl}}/assets/img/signverify.png)

As the picture depicts, first step is the signing process, sender signs the message (actually generates the message signature and puts it alongside the message), then encrypts them by receiver’s public key.

In the receiver’s side, first step is the decryption, after it is done, receiver verifies the message by using message signature and the last step is comparing two hash values.

### Key pair Generation

There are several tools out there for generating key pairs but in this tutorial Jdk is the preferred one. Jdk is bundled with a bunch of utilities and keytool is the one needed. With the use of keygen two pairs of keys are generated, first one is going to be used by web service server and web service client uses the second one, also it has to be mentioned that public keys of the key pairs have to be shared between server and client which is called  [key_exchange][key-exchange].

First we generate server keystore (java keystore) using this command:
{% highlight cmd %}
keytool -genkey -alias server -keyalg RSA -keysize 1024 -keypass password -keystore server.jks - storepass password
{% endhighlight %}
The output of this command is a java keystore (jks), to be brief jks is a repository of certificates. The jks we have generated contains two entries, first one is a public key (actually a self-signed one) and the second one is a private key.

In the second step, client keystore is going to be generated:
{% highlight cmd %}
keytool -genkey -alias client -keyalg RSA -keysize 1024 -keypass password -keystore client.jks -storepass password
{% endhighlight %}
And in the last step the generated jks files are going to be changed, client and server public keys have to be exported and imported into other parties jks files(in this way public keys are shared between client and server). To exchange the public keys again Jdk keytool utility is used:

First we export server public key:
{% highlight cmd %}
keytool -export -alias server -file serverpubk.cert -keystore server.jks
{% endhighlight %}
Then we import the exported public key (serverpubk.cert) to the client keystore:
{% highlight cmd %}
keytool -importcert -file serverpubk.cer -keystore client.jks -alias "server"
{% endhighlight %}
In the next step we do the same for the client keystore, exporting public key:
{% highlight cmd %}
keytool -export -alias client -file clientpubk.cer -keystore client.jks
{% endhighlight %}
and finally we import the client public key to the server keystore:
{% highlight cmd %}
keytool -importcert -file clientpubk.cer -keystore server.jks -alias "client"
{% endhighlight %}

To sum it up, there are 3 entries in each jks files. In server.jks there server private key, server public key and client public key and inside client.jks we have client public key, client private key and server public key.

### Project Structure
In these two pictures the structure of the projects (both server and client) are depicted, as you can see, intellij alongside maven is used to develop the server and client side of the webservice. It should be noticed that it is not intended to get into the details of the projects, just important parts of the projects are going to be pointed out.

#### Client Porject Structure
![Client_Porject_Structure]({{site.baseurl}}/assets/img/clientprojectstructure.jpg)

#### Server Porject Structure
![Client_Porject_Structure]({{site.baseurl}}/assets/img/serverprojectstructure.jpg)

#### ServerSide
This is the content of cxf.xml, actually it’s the application context configuration file and some beans are defined in it:

{% highlight XML %}
<!--(1)-->
<beans xmlns="http://www.springframework.org/schema/beans"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:jaxws="http://cxf.apache.org/jaxws"
 xsi:schemaLocation="http://www.springframework.org/schema/beans 	http://www.springframework.org/schema/beans/spring-beans.xsd http://cxf.apache.org/jaxws 	http://cxf.apache.org/schemas/jaxws.xsd">
 <!--(2)-->
 <bean id="helloServiceBean" class="net.ali4j.service.server.HelloServiceImpl"/>
  <!--(3)-->
  <jaxws:endpoint id="helloServiceEndpoint" implementor="#helloServiceBean" address="/services/helloService">
   <!--(3.1)-->
   <jaxws:features>
    <bean class="org.apache.cxf.feature.LoggingFeature" />
   </jaxws:features>
   <!--(3.3)-->
   <jaxws:inInterceptors>
    <ref bean="taxInboundInterceptor" />
   </jaxws:inInterceptors>
   <!--(3.2)-->
   <jaxws:outInterceptors>
    <ref bean="taxOutboundInterceptor" />
   </jaxws:outInterceptors>
  </jaxws:endpoint>
  <!--(4)-->
  <bean id="taxInboundInterceptor" class="org.apache.cxf.ws.security.wss4j.WSS4JInInterceptor">
    <constructor-arg>
     <map>
      <!--(4.1)--><entry key="action" value="Timestamp Signature Encrypt"/>
      <!--(4.2)--><entry key="signaturePropFile" value="server.properties"/>
      <!--(4.3)--><entry key="decryptionPropFile" value="server.properties"/>
      <!--(4.4)--><entry key="passwordCallbackClass" value="net.ali4j.util.ServerPasswordCallback"/>
       </map>
    </constructor-arg>
   </bean>
   <!--(5)-->
   <bean id="taxOutboundInterceptor" class="org.apache.cxf.ws.security.wss4j.WSS4JOutInterceptor">
    <constructor-arg>
     <map>
      <!--(5.1)--><entry key="action" value="Timestamp Signature Encrypt"/>
      <!--(5.2)--><entry key="user" value="server"/>
      <!--(5.3)--><entry key="encryptionUser" value="client"/>
      <!--(5.4)--><entry key="signaturePropFile" value="server.properties"/>
      <!--(5.5)--><entry key="encryptionPropFile" value="server.properties"/>
      <!--(5.6)--><entry key="passwordCallbackClass" value="net.ali4j.util.ServerPasswordCallback"/>
      <!--(5.7)--><entry key="signatureParts" value="{Element}{http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd}Timestamp;{Element}{http://schemas.xmlsoap.org/soap/envelope/}Body"/>
      <!--(5.8)--><entry key="encryptionParts" value="{Content}{http://schemas.xmlsoap.org/soap/envelope/}Body"/>
      <!--(5.9)--><entry key="encryptionSymAlgorithm" value="http://www.w3.org/2001/04/xmlenc#tripledes-cbc"/>
     </map>
    </constructor-arg>
   </bean>
</beans>
{% endhighlight %}

(1) is the root element of context configuration file, namespaces which are used through this configuration file are defined in this element (such as jaxws).<br/>

(2) is an instance of the SEI and is used by another beans.<br/>

(3) is the endpoint of the web service and it’s the main component which exposes the service. The value of id attribute is “helloServiceEndpoint” which is used to get the bean from application context in main class, implementer is a reference to an instance of SEI which we have defined it as bean (2) and finally address attribute is the address of the webservice which is going to be published. This bean can have multiple children and we have defined three of them.
   <br/>(3.1) is a “feature” and its duty is to log the incoming and out coming messages.
   <br/>(3.2) is an “InInterceptor”, one of the usages of interceptors is to apply some changes on messages, both incoming and out coming, this interceptor is used to apply encryption and signing on the out coming messages.
   <br/>(3.3) another interceptor used to decrypt the incoming messages, also message verification is done by this interceptor.

<br/>(4) instance of WSS4JInInterceptor class, message decryption and verification process is done using this interceptor. We have sent some constructor args for initializing this bean as a Map object:
   <br/>(4.1) list of actions which are going to be performed by this instance.
   <br/>(4.2&4.3) for both of these entries one properties file is used, client sends encrypted and signed messages to the server, server (or the recipient of the messages) has to decrypt and verify the incoming messages, data needed for these two actions are included in server.properties file.
   <br/>(4.4) this class is used to provide the password of the private key which is resided inside server key store file (server.jks), in fact this key store includes three entries, first one is the private key of the server, second one is the server public key and the last on is the client public key. To access the server private key we have to have two passwords, first one is the password of the jks file (which is provided by server.properties file) and the second one is the one for the private key itself, but client public key which also is included in server key store doesn’t have any password on its own.

<br/>(5) is another interceptor which is used to customize out coming messages, actually it’s an instance of WSS4JOutInrerceptor and signing and encryption is its duty. Again we have to send some arguments for this interceptor constructor as a map:
   <br/>(5.1) This is how WSS4J is advised to put a timestamp in the SOAP message, and there is signing and encryption.
   <br/>(5.2) alias of server private key entry in keystore
   <br/>(5.3) alias of client public key entry in server keystore
   <br/>(5.4&5.5) these two entries provide the configuration data needed for signing and encryption
   <br/>(5.6) this is a class which provides the password of the private key
   <br/>(5.7) this entry indicates to the parts of the message which have to be signed and as it shows Timestamp and Body of the message are going to be signed.
   <br/>(5.8) again this entry is used to show the parts of the message which have to be encrypted and actually it indicates to the Body of the message.
   <br/>(5.9) is the algorithm which is used to encrypt the data.

   To sum up the cxf.xml file an overview of the beans and their corresponding usages is given:

   1- <bean id=”helloServiceBean” class=”net.ali4j.service.HelloServiceImpl”/> <br/>
   This is an instance (bean) of the implementation of the hello world service.

   2- <jaxws:endpoint id=”helloServiceEndpoint” implementer=”#helloServiceBean” address=”/services/helloService”><br/>
   This element implies that cxf uses jax-ws frontend to publish the web service and it is obvious the value of address attribute is the address which service is published under it.

   3- <bean id=”taxInboundInterceptor” class=”org.apache.cxf.ws.security.wss4j.WSS4JInInterceptor”><br/>
   The purpose of the usage of this bean is to apply message verification and decryption on input soap messages.

   4- <bean id=”taxOutboundInterceptor” class=”org.apache.cxf.ws.security.wss4j.WSS4JOutInterceptor”><br/>
   This is bean is used to sign and encrypt output soap messages.


#### Client Side
To use a published web service, proxy classes have to be generated in some way and there are many ways, apache axis and apache cxf are a few of them, here maven wsimport plugin is choosed.

To use wsimport plugin first create a directory named wsdl in src folder of the project (when using wsimport plugin, it looks for files ends with wsdl in this directory) and put web service wsdl file inside it and then use:

{% highlight cmd %}
mvn jaxws:wsimport 
{% endhighlight %}

command to generated proxy classes. They reside in target folder under generated-resources/wsimport directory.

Spring IOC container is used in client side too for instantiating and initializing required beans, so again there is a context configuration file, client-beans.xml actually:

{% highlight XML %}
<!--(1)-->
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:jaxws="http://cxf.apache.org/jaxws" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
 <!--(2)-->
 <jaxws:client id="helloClient" serviceClass="net.ali4j.service.server.HelloService" address="http://localhost:8080/CXFSignEncServerHelloWorld/services/helloService" wsdlLocation="http://localhost:8080/CXFSignEncServerHelloWorld/services/helloService?wsdl">
  <!--(2.1)-->
  <jaxws:inInterceptors>
   <ref bean="inbound-security" />
  </jaxws:inInterceptors>
  <!--(2.2)-->
  <jaxws:outInterceptors>
   <ref bean="outbound-security" />
  </jaxws:outInterceptors>
  <!--(2.3)-->
  <jaxws:features>
   <bean class="org.apache.cxf.feature.LoggingFeature" />
  </jaxws:features>
 </jaxws:client>
 <!--(3)-->
 <bean class="org.apache.cxf.ws.security.wss4j.WSS4JInInterceptor" id="inbound-security">
  <constructor-arg>
   <map>
    <!--(3.1)--><entry key="action" value="Timestamp Signature Encrypt"/>
    <!--(3.2)--><entry key="signaturePropFile" value="client.properties"/>
    <!--(3.3)--><entry key="decryptionPropFile" value="client.properties"/>
    <!--(3.4)--><entry key="passwordCallbackClass" value="net.ali4j.service.util.ClientPasswordCallback" />
   </map>
  </constructor-arg>
 </bean>
 <!--(4)-->
 <bean class="org.apache.cxf.ws.security.wss4j.WSS4JOutInterceptor" id="outbound-security">
  <constructor-arg>
   <map>
    <!--(4.1)--><entry key="action" value="Timestamp Signature Encrypt"/>
    <!--(4.2)--><entry key="user" value="client"/>
    <!--(4.3)--><entry key="signaturePropFile" value="client.properties"/>
    <!--(4.4)--><entry key="encryptionPropFile" value="client.properties"/>
    <!--(4.5)--><entry key="encryptionUser" value="server"/>
    <!--(4.6)--><entry key="signatureKeyIdentifier" value="DirectReference"/>
    <!--(4.7)--><entry key="passwordCallbackClass" value="net.ali4j.service.util.ClientPasswordCallback"/>
    <!--(4.8)--><entry key="signatureParts" value="{Element}{http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd}Timestamp;{Element}{http://schemas.xmlsoap.org/soap/envelope/}Body"/>
    <!--(4.9)--><entry key="encryptionParts" value="{Element}{http://www.w3.org/2000/09/xmldsig#}Signature;{Content}{http://schemas.xmlsoap.org/soap/envelope/}Body"/>
    <!--(4.10)--><entry key="encryptionSymAlgorithm" value="http://www.w3.org/2001/04/xmlenc#tripledes-cbc"/>
   </map>
  </constructor-arg>
 </bean>
</beans>
{% endhighlight %}

(1) root element of context configuration file.
<br/>(2) this element is used to add a web service client to the spring context, id attribute is employed to get the bean from application context in main method, value of serviceClass attribute is the full qualified class name of the interface generated by wsimport plugin and address is the URL to connect to in order to invoke the service. It also supports many child elements and just two of them are used.
   <br/>(2.1) message decryption and verification is the purpose of the usage of this interceptor.
   <br/>(2.2) and this interceptor is used to sign and encrypt the message
   <br/>(2.3) this element holds a list of features, feature is component that provides extra capability to the server, client or bus over their existing functionality, Logging feature is used to log incoming and out coming messages
<br/>(3) is the same as the server context configuration file
<br/>(4) is the same as the server context configuration file

#### Test
In the last step incoming and out coming messages have to be observed and what’s important inside messages is their payload.
Thanks to the LoggingFeature which is used, incoming and out coming messages (actually inbound and outbound messages in cxf terminology) for both client and server is logged in console so we can have a look at what gets transmitted between client and server.

Use this command to run the server: 
{% highlight cmd%}   
	mvn jetty:run 
{% endhighlight %}   
and then access the wsdl of the service from this address:
	http://localhost:8080/CXFSignEncServerHelloWorld/services/helloService?wsdl

In order to run the client just after cloning by running main method, do not change the address and port of the published service, otherwise you have to generate the client code again.

client inbound and outbound messages:<br/>
![Client_Porject_Structure]({{site.baseurl}}/assets/img/clientlog.jpg)

server inbound messages:<br/>
![Client_Porject_Structure]({{site.baseurl}}/assets/img/serverlogin.jpg)

server outbound messages:<br/>
![Client_Porject_Structure]({{site.baseurl}}/assets/img/serverlogout.jpg)

Details of the messages payloads are far beyond the purpose of this tutorial, so it is not aimed to be discussed.

#### Conclusion
Client side and Server side of the implemented webservice didn’t get discusses in details but since they are quite simple, they are self descriptive and only important parts of them such as generating proxy classes and spring IOC usage described.
To sum up we have implemented message level signing and encryption in a jax-ws webservice using apache cxf.
Both server and client projects are available in my
[Github][Github] ([client][client] and [server][server]), so feel free to take a look at them.


[key-exchange]: https://en.wikipedia.org/wiki/Key_exchange
[Github]: https://github.com/ali4j
[client]: https://github.com/ali4j/CXFSignEncClientHelloWorld
[server]: https://github.com/ali4j/CXFSignEncServerHelloWorld
