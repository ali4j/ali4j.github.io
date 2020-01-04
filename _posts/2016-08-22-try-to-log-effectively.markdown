---
layout: post
title: "Try to log effectively!"
date: 2016-8-22 00:00:00 +0400
description: my first post about logging. # Add post description (optional)
img: log_effectively.jpg # Add image post (optional)
tags: [log4j, logging, logback]

---
Logging plays an important role in software development. Actually the main purpose of logging is to monitor what your application is doing, especially when it’s not possible for you to debug it, e.g. production environment. Whenever I encounter a bug or an error in the product I have developed, the only place which I can elicit some information from it, is the log file, so logging appropriately is an essential part of developing.

I’m just going to talk about approaches I take and configurations I use in my development environment.

## Configurations
In my current project, I’m using log4j as the logging framework. There are some reasons that I’ve choose it. Actually it’s a mature framework with lots of document, it’s popular and well-known, and also it’s the one that I’m most knowledgeable about.

1. Use a meaningful pattern for your logs. You are going to use your log file as a source of runtime information, so you have to get the most out of it, therefore consider these items in your logs:

* Date and Time
* Thread name
* Full qualified class name
* Log message

{% highlight Properties %}
log4j.appender.rollingAppender=org.apache.log4j.DailyRollingFileAppender
log4j.appender.rollingAppender.File=path/to/logfile/logfilename.log
log4j.appender.rollingAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.rollingAppender..layout.ConversionPattern=%d{dd MMM yyyy HH:mm:ss}-%t-%p-%C-%M-%m%n
{% endhighlight %}

The above lines of log4j configuration results in a log message like the below one:

10 Aug 2016 12:49:37-main-INFO-net.ali4j.log4jtest.TestClass-test-this is the log message

This line of log message is based on the conversion pattern that is used in configuration lines, we can check each part of coversion pattern separately regarding log message. First part of the log message is the date and time of the message which is configured using `%d{dd MMM yyyy HH:mm:ss}` and it results `10 Aug 2016 12:49:37`. Second part of the message (actually main -INFO)is the result of `%p` and it prints the name thread in which the messages in printed the appender and the level if the message. Thirdly, `%C` prints the full qulified class name and `%M`  prints the method name. Finally, `%m%n` prints the log messages and appends a new line character at the end of the message (using `%n`).

2. Try to break your log file into smaller ones. By using DailyRollingFileAppender class, log4j can create a separate log file for every single day, moreover, the current date is appended at the end of the file name which gives you more control over the logs. As you can see DailyRollingFileAppender class is used as the rollingAppender, similarly FileAppender can be used too but as a result we are going to have only one log file.

3. In addition to the daily file name, you can put a limit on the size of the log file and break your log file based on its size. By adding below line of configuration you can achieve this feature:

{% highlight Properties %}
log4j.rollingAppender.MaxFileSize=10KB/10MB/10GB
{% endhighlight %}

The only point you have to notice is that just `DailyRollingFileAppender` and `RollingFileAppender` classes have this property so you can’t use this configuration for `FileAppender` class.

## Ways of logging
The approach you take to log requires special attention, notice that the information you log is going to be used in production environment as a source of data to monitor your application, so keep these tips in mind.

1. Use different log levels for logging. There are different log levels in logging frameworks(log4j too) which are used for particular purposes. But these are the most commonly used ones:


    DEBUG<INFO<ERROR

    **INFO**: use this level to log changes in the state of the application, for instance, when new user is registered, some transaction has been commited, a record has been added to the database, or anything that has been done successsfully or unsuccessfully.

    **DEBUG**: as it’s name shows this level is used for debugging. You can use debug level to log vast range of data such as input and output of methods, thread states, user inputs, incoming requests, outgoing responses, validation states and so on. Generally anything in detail should be logged by using this level.

    **ERROR**: this level is most commonly used to log exceptions and some kind of situations that system can not tolerate such as NPE, database unavailability and … . Logs of this levels show that something hazardous has happened and has to be taken care as soon as possible.


2. Remember to use Logging Guards:
    {% highlight java %}
    if(_logger.isDebugEnabled()) _logger.debug(“debug message”);
    {% endhighlight %}

    Whenever you use debug level to log a message, check if debug level is enabled and then log it. The purpose of this guard is not to prevent log message from being written, 
    actually it’s intended to prevent useless calls to toString() which can be quite expensive.

That’s all about my experience with logging in java. Whenever you decide to use logging in your application(of course you should use) just keep in mind that later you need this information, so try to log effectively!