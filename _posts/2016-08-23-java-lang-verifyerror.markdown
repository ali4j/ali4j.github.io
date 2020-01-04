---
layout: post
title: "java.lang.verifyerror"
date: 2016-8-23 00:00:00 +0400
description: verifyerror which happens due to conflict in maven dependencies . # Add post description (optional)
img:  java_lang_verifyerror.jpg # Add image post (optional)
tags: [maven, maven_dependency, java]

---
Recently I had this error in my project and I wasn't able to fix it, but after searching awhile I finally resolved it.

{% highlight java%}
Caused by: java.lang.VerifyError: (class: net/ali4j/testservices/sendcustomsdeclarationchange/jaxws_asm/SendCustomsDeclarationChange, method: getChangeTime signature: ()Ljava/util/Date;) Falling off the end of the code
	at java.lang.Class.getDeclaredConstructors0(Native Method)
	at java.lang.Class.privateGetDeclaredConstructors(Unknown Source)
	at java.lang.Class.getConstructor0(Unknown Source)
	at java.lang.Class.getDeclaredConstructor(Unknown Source)
	at org.apache.cxf.common.util.ReflectionUtil$4.run(ReflectionUtil.java:105)
	at org.apache.cxf.common.util.ReflectionUtil$4.run(ReflectionUtil.java:102)
	at java.security.AccessController.doPrivileged(Native Method)
	at org.apache.cxf.common.util.ReflectionUtil.getDeclaredConstructor(ReflectionUtil.java:102)
	at org.apache.cxf.common.jaxb.JAXBUtils.getValidClass(JAXBUtils.java:582)
	at org.apache.cxf.jaxb.JAXBContextInitializer.addClass(JAXBContextInitializer.java:309)
	at org.apache.cxf.jaxb.JAXBContextInitializer.begin(JAXBContextInitializer.java:187)
	at org.apache.cxf.service.ServiceModelVisitor.visitOperation(ServiceModelVisitor.java:97)
	at org.apache.cxf.service.ServiceModelVisitor.walk(ServiceModelVisitor.java:74)
	at org.apache.cxf.jaxb.JAXBDataBinding.initialize(JAXBDataBinding.java:315)
	at org.apache.cxf.service.factory.AbstractServiceFactoryBean.initializeDataBindings(AbstractServiceFactoryBean.java:86)
	at org.apache.cxf.wsdl.service.factory.ReflectionServiceFactoryBean.buildServiceFromClass(ReflectionServiceFactoryBean.java:467)
	at org.apache.cxf.jaxws.support.JaxWsServiceFactoryBean.buildServiceFromClass(JaxWsServiceFactoryBean.java:696)
	at org.apache.cxf.wsdl.service.factory.ReflectionServiceFactoryBean.initializeServiceModel(ReflectionServiceFactoryBean.java:527)
	at org.apache.cxf.wsdl.service.factory.ReflectionServiceFactoryBean.create(ReflectionServiceFactoryBean.java:261)
	at org.apache.cxf.jaxws.support.JaxWsServiceFactoryBean.create(JaxWsServiceFactoryBean.java:199)
	at org.apache.cxf.frontend.AbstractWSDLBasedEndpointFactory.createEndpoint(AbstractWSDLBasedEndpointFactory.java:102)
	at org.apache.cxf.frontend.ServerFactoryBean.create(ServerFactoryBean.java:168)
	at org.apache.cxf.jaxws.JaxWsServerFactoryBean.create(JaxWsServerFactoryBean.java:211)
	at org.apache.cxf.jaxws.EndpointImpl.getServer(EndpointImpl.java:460)
	at org.apache.cxf.jaxws.EndpointImpl.doPublish(EndpointImpl.java:338)
	at org.apache.cxf.jaxws.EndpointImpl.publish(EndpointImpl.java:255)
	at org.apache.cxf.jaxws.EndpointImpl.publish(EndpointImpl.java:543)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(Unknown Source)
	at java.lang.reflect.Method.invoke(Unknown Source)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeCustomInitMethod(AbstractAutowireCapableBeanFactory.java:1581)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1522)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1452)
	... 68 more
{% endhighlight%}

You may encounter this error because of several reasons during runtime.
First and the most common one is that you have comipled your code against a different library you are using at runtime.

Second reason can be the bytecode size of the method in your class. If the size exceeds 64KB jre will throw this error.

Thirdly, you may use same library with different versions.If you have same library with different versions in your classpath probabley you will face this error, so take good care of your 3rd party libraries.

In my case the problem was occurred due to the conflict between two dependencies in maven. There were two direct dependency in pom file which behinde the scene were using same transparent dependency with different versions, so I decied to exclude both versions of the it and create a direct dependency to the newest one.
