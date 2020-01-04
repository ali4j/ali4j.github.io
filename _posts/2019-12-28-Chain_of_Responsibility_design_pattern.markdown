---
layout: post
title: "Chain of Responsibility design pattern"
date: 2019-12-28 14:00:00 +0400
description: a simple demonstration of Chain of responsibility pattern.
img: cor_design_pattern.jpg
tags: [java,java core, design patterns, chain of responsibility, cor]

---
This is one of the behavioral patterns which comes quite handy. This pattern specifies the way in which components interact with each other and that it is categorized in behavioral patterns.


As the name suggests this patterns is useful when there is a chain of business logic units which should be applied on a piece of data. There are several demonstrations of this pattern which is utilized by many popular frameworks and libraries. For instance, security filter chain in spring security is a clear and awesome implementation of this pattern which applies several security constraints on a request sent from the client.

The general idea of this design pattern is that if the request (or the piece of data which should be processed) is successfully processed by current component of the chain then it should be passed to the next component, otherwise some exception should be thrown or some error-handling policy should be provided.

Here I go through a simple example of this pattern. This sample imitates the security filter chain of spring security in a simple form and tries to process the request throughout a chain of filters. 

![chian_of_responsibility_class_diagram]({{site.baseurl}}/assets/img/cor_class_diag.png)


{% gist 9d39c2927941437562d7d475afd0a3d8 %}

Based on the class diagram, there is an interface defining a contract which every request handler should implement. It has two methods, setNext(Handler handler) which sets the next handler of the chain and execute(Request request) which performs the business logic.


{% gist 77c1480224338d186311984da5bdce19 %}

AbstractHandler class implements Handler interface and provides an implementation of setNext method. The other method of Handler class is not implemented since it is expected to be implemented by a concrete class.

ValidationHandler, AuthenticationHandler and AuthorizationHandler are the concrete classes which provides specific implementations for execute method of Handler interface. As the name shows, Request instance gets validated in ValidationHandler, the user/client which has sent the request gets authenticated in authentication handler and finally the request gets authorized to check if they have access to the requested resource in AuthorizationHandler.

{% gist 848504dc516d0cd8caf7f2156087ee30 %}
{% gist 88444e65baaa26c170c0394b2a357969 %}
{% gist b464880eaadbf78caa44d96379306a22 %}

An important point about the handler concrete classes is that their order in the chain can be set in runtime and in our example the first handler is the ValidationHandler, second one is the AuthenticationHandler and last one is AuthorizationHandler.

Finally, there are two value classes, first one is Request class which simulates a http request sent by a client and User class which holds the data of authenticated users, actually it plays the role of a datasource which provides registered users.

{% gist cb2dcac78e1935f73cb5f7086bd8f08a %}

{% gist 4a2b184d31fcf2ecc51dbdafcbef8653 %}

The Demo class contains several tests to show how the request is processed by the handlers.

{% gist 55ffa6064a0ca2bf47b1d68364794c37 %}
