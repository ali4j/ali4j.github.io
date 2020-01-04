---
layout: post
title: "Basic Authentication"
date: 2016-8-26 00:00:00 +0400
description: basic authentication in soap web services . # Add post description (optional)
img: basic_authentication.jpg # Add image post (optional)
tags: [java,soap,webservices,basic_authentication, apache cxf, inteceptors]
---

Basic Authentication is one of the most-used methods of authentication due to its simplicity and is defined in RFC 7617 specification. 
In this approach the client sends its USERNAME and PASSWORD as cleartext along side its requests. To be more precise, the Authorization header is added to the list of headers and Base64 encoded value of the provided USERNAME and PASSWORD is added as the value of the header. The exact format of the value of the header is compromised of 3 parts:  
1. first part is the Word 'Basic'
1. then it is followed by a colon ':' 
1. and the final part is base64 encoded value of USERNAME:PASSWORD 


and the final form of the value is something like:
	Basic bmD9F8SDJFSDKJFSD98FNSDJFMNS0D9FMUSDJKFKSDHNFM8S0DUFNSDJKFH=.
	

The best practice is Using BA in combination with an external secure system such as TLS. Since user credentials is sent as plain text over the wire it should be encrypted in order to avoid sniffing. 

This was a brief introduction about BA, now we can move on to the next step in which this method of authentication is implemented.

BA can be applied on any resource which is exposed to be requested by clients, it can be a static content such as an image or a restful web service which can be used to fetch a record of a database. In this tutorial Im just going to implement BA on a soap web service implemented with apache cxf and a rest web service implemented sing spring boot.


### BA using apache cxf 
Apache cxf is an implementation of both JAX-WS and JAX-RS specifications, this means that you can use it for exposing both soap and rest services, but here it is used just for implementing a soap web service.

Cxf has a utility called `Interceptor` which works mostly like servlet filters, with this difference that it can be applied on different phases of processing incoming soap messages, while filters are applied on urls and http requests. Using this utility, it is possible to preprocess received soap messages and check whether the sender is authenticated.

Now we are going to take a look at the interceptor which implemenets a basic security authentiction: 

{% highlight java %}
package net.ali4j.tutorials.soapbasicauthentication; 

import org.apache.cxf.interceptor.Fault; 
import org.apache.cxf.message.Message; 
import org.apache.cxf.phase.AbstractPhaseInterceptor; 
import org.apache.cxf.phase.Phase; 
import org.apache.cxf.transport.http.AbstractHTTPDestination; 
import org.springframework.stereotype.Component; 
import javax.servlet.http.HttpServletRequest; 
import java.util.Optional; 

@Component 
public class BasicAuthenticationInterceptor extends AbstractPhaseInterceptor<Message> { 

   public BasicAuthenticationInterceptor() { 
       super(Phase.PRE_INVOKE); 
   } 

   private static final String DEFAULT_USERNAME = "username"; 
   private static final String DEFAULT_PASSWORD = "password"; 

   @Override 
   public void handleMessage(Message message) throws Fault { 
       HttpServletRequest request = (HttpServletRequest) message.get(AbstractHTTPDestination.HTTP_REQUEST); 
       Optional<String> authorizationOptional = 
               Optional.of(request.getHeader("Authorization").trim()).filter(s -> !s.isEmpty()); 
       if (authorizationOptional.isPresent())
           if(!checkAuthentication(authorizationOptional.get())) unauthenticated(); 
       else unauthenticated(); 
   } 

   private boolean checkAuthentication(String authenticationHeaderValue) { 
       BasicAuthenticationUserCredentials credentials = new BasicAuthenticationUserCredentials(authenticationHeaderValue); 
       return credentials.getUsername().equals(DEFAULT_USERNAME) && credentials.getPassword().equals(DEFAULT_PASSWORD); 
   } 

   private void unauthenticated() { 
       throw new RuntimeException("invalid credentials"); 
   } 

} 
{% endhighlight %}
In order to create an interceptor your class has to extend AbstractPhaseInterceptor which is an abstract interceptor provided by cxf and it should override handle method. Inside handle method, Authorization header (which is a http header) should be acquired from the request (which is represented as Message in handle method) so to achieve this, a HttpServletRequest is gotten from the message and using it we can have access to http headers sent by the client. 

Next step is to get the value of Authorization header and performing the authentication. Ive created a POJO class named BasicAuthenticationUserCredentials which when instantiating, you should provide the value of authorization header for the constructor and then it exposes the username and password set in heaer.

After BasicAuthenticationUserCredentials instance is created (with proper values), it can be used to authenticate user. Based on your business, authentication process may differ, for exmaple you can check user credentials against a database record or maybe somewhere an authentication api is provide, but here the simplest scenario is choosed and username and password is hardcoded.

Last point which worth mentioning is that, in this example, an unauthorized state is handled with a RuntimeException, again based on your business and scenario it can be changed and a more proper exception handling approach can be employed. 
A complete example is implemented using spring boot and apache cxf and you can take a look at it in [my github][Github]. 

[Github]: https://github.com/ali4j/ApacheCXFBasicAuthentication
