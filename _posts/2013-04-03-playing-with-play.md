---
layout: post
title: Playing with Play
categories: software-engineering
tags: ics-691 java play-framework
---
This week I've started working with The Play Framework, which is a web application framework. The extent of my web development includes playing around with basic html and javascript, so this should be an intersting experience. I hope to have several updates over the coming weeks about my first real experience with web application development.

## The Play Framework
[<img src="http://upload.wikimedia.org/wikipedia/commons/f/fd/MVC-Process.png" alt="MVC Process" />](http://upload.wikimedia.org/wikipedia/commons/f/fd/MVC-Process.png)

The [Play Framework](http://www.playframework.org/) follows the [model-view-controller](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) (MVC) architecture. To be honest, I still don't quite understand what that means, but the diagram to the right seems to give a simple visual description of the architecture. The MVC architecture is used the by [Ruby on Rails](http://rubyonrails.org/) and [Django](https://www.djangoproject.com/), which are popular web application frameworks for Ruby and Python, respectively.

Play uses Java, and as of version 1.1, Scala. Since I am most familiar with Java, and know next to nothing about Scala, I will be using Java. This was a major factor in deciding to use Play.

## Doing the Todo List
For my first Play "project" I just followed the [Todo List](http://www.playframework.com/documentation/2.1.0/JavaTodoList) tutorial provided by the Play website. The tutorial was fairly easy to follow, but I thought it was quite lacking. It does a decent job of telling you how to use the framework, but it doesn't give you much details. Instead they just redirect you to more readings so you can figure it out later.

I went through most of the tutorial without problems, but I didn't feel like I learned much because they just told me what to do rather than really explaining why I'm doing each thing. Perhaps if I read all the extra documentation that they provide I might get a better understading. However, I feel like a walkthrough of your first application should have a little more detail and explanation.

I did run into one small problem while going through the tutorial. After setting up the database, I received a strange error (which can be seen in the screenshot below) when I refreshed the page. The error says "Database 'default' needs evolution!" I'm still not sure what this means, but clicking the "Apply this script now!" button seemed to solve the problem. After that, everything seemed to run smoothly.

[<img src="database_needs_evolution.png" alt="database needs evolution">](database_needs_evolution.png)

Overall, I had a pleasant experience with Play. It is definitely a huge step up from hacking away with simple javascript. Below is a screenshot of the completed application. It's not much to look at since it is basically the same as the screenshot in the tutorial, but it is my first stepping stone into the world of web application development.

[<img src="todo_list.png" alt="completed todo list">](todo_list.png)

## Reflection
Although I still don't quite understand the framework or the MVC architecture in general, I am looking forward to using Play more. Hopefully I will learn a lot more about web development and the Play Framework in the coming weeks. It seems like I have a difficult road ahead of me, but hopefully I will be able to take away some valuable lessons and be able to create my first web app. I will be sure to keep you updated on my progress.

