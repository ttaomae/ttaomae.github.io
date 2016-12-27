---
layout: post
title: Playing with Play Part II - Making Models
---
[Last week]({{ site.baseurl }}{% post_url 2013-04-03-playing-with-play %}) I discussed my brief experience with the Play Framework. This week I've continued to explore the framework with a focus on the model part of the [MVC architecture](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller).

## Model Development
This week, I've only worked on model development. In terms of the MVC architecture, the model is just a model of the data that will be used in the application and how it is related to each other. The models exist on their own and they do not rely on the other components of the MVC architecture (the views and controllers), so it was a good place to start. In the coming weeks I will start to explore controllers and views in more depth.

For my models, I have been using [Ebean](http://www.avaje.org/), which is an [object-relation mapper](http://en.wikipedia.org/wiki/Object-relational_mapping) (ORM) provided with Play. Ebean provides persistence for your data. Although there are other methods to allow you to persist your data, Ebean seems like a good approach for the Play Framework because it allows you to continue to think in an object-oriented manner.

## Warehouse Data Model
The first data model that I implemented was a simple warehouse. My data model is largely based off of the model described in [this screencast](https://www.youtube.com/watch?v=FSPNjUtenlA). In the model, a single warehouse can have many stock items. Each stock item is associated with a particular product, but a single product can have many stock items. There are can also be any number of tags which are associated with any number of products. After completing the model described in the screencast, I made a small enhancement to add an address which is associated with the warehouse. All of the relationships can be seen in the simple [ER model](http://en.wikipedia.org/wiki/Entity%E2%80%93relationship_model) below.

[<img src="warehouse_data_model.png" alt="warehouse data model" />](warehouse_data_model.png)

## Book Exchange Data Model
After completing the warehouse data model, I moved onto a data model that would support a textbook exchange for students, which is outlined [here](http://ics613s13.wordpress.com/modules/web-application-development/a24-simple-model-development/). First I created a simple ER model (shown below), which describes the relationships between books, students, requests, and offers. A student can make any number of requests or offers and there can be any number of requests or offers for a book. Creating the basic model was fairly simple, but Ebean began to give me problems when I tried to test the model.

[<img src="book_exchange_data_model.png" alt="book exchange data model" />](book_exchange_data_model.png)

## Model Testing
A basic test of the model is fairly simple. First create some data, then persist it to the database, then check that the database has all of your data. The next thing to check is that all of your relationships still exist. Again, this is pretty simple and it didn't give me any problems. However, I started to have problems when I tried removing data from the database. When I tried deleted a book it would also deleted the requests and offers associated with it. Arguably, this is not a problem because if a book no longer exists, you do not want to have offers or requests for the non-existent book. Ultimately, this is how I decided to model my data. The problem, however, was that it would also delete the student (if all of the student's requests and offers were deleted). In this case, it makes sense to have a student that has no requests or offers, so you would not want to delete it.

After collaborating with another student who was working on the same project and having similar issues, we seem to have figured out the issue. In each entity, I defined each relationship to cascade all operations. That means when I deleted the book, it would automatically delete the requests and offers associated with it, then delete the students associated with the requests and offers. To solve my problems, I provided an overridden delete method, which would manually cut any links between entities so that the cascade would not affect it. Now when I delete a book, the student remains even if the student has no requests or offers. Perhaps further testing will uncover more problems, but for now, this seems to work.

## Reflection
Although I don't quite understand the entire MVC architecture yet, I can see why the data model can be very important to many web applications. Although Ebean ended up giving me some problems, I actually enjoyed using an ORM. It seems like an interesting way to mix object-oriented programming and relational data.

So far, I don't think I've had enough experience with Play to fully appreciate it. Hopefully in a few weeks, I will begin to understand what is so great about the MVC architecture. In the meantime, I've released both the [warehouse](https://github.com/ttaomae/warehouse) and [book exchange](https://github.com/ttaomae/book-exchange) on GitHub. I will continue to develop both of these over the coming weeks.

