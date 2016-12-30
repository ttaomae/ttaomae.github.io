---
layout: post
title: Playing With Play Part III - Creating Controllers
categories: software-engineering
tags: ics-691 java play-framework
---
A [few weeks ago]({{ site.baseurl }}{% post_url 2013-04-03-playing-with-play %}) I started working with the Play Framework, which claims to make it "easy to build web applications with Java & Scala." However, I am seriously starting to doubt that. The more I use it, the less it feels like play, and the more if feels like very frustrating work.

## Controller Development
This week, I intended to focus on controller development, but it turned out to cause unexpected problems which forced me to spend a lot of time rethinking and reworking my models. The controller of the [MVC architecture](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), in very general terms, serves as a link between the models, which I discussed [last week]({{ site.baseurl }}{% post_url 2013-04-10-playing-with-play-ii %}), and the views, which I will work on and discuss in the coming weeks. The controller is responsible for handling HTTP requests and sending commands to the views to display the content. It is also responsible for manipulating the models. For example, the controller is responsible for creating new entities and saving it to the database.

### Routing Requests
Before writing the actual controllers, the first thing to do is to modify the "routes" file, found in the "conf" directory. This file dictates how to handle different HTTP requests based on the page that the user is trying to navigate to. For example, if a user navigates to `<web-app url>/students` you may want to show a list of all students. To do this, you might add the following line to your routes file:

```
GET    /students     controllers.Student.index()
```

where `controllers.Student` is a `Controller`, and `index` is a method that will display a list of all students. You can also define actions for different types of HTTP requests such as POST and DELETE.

### Creating Controllers
Once you modify the routes file to handle the different types of requests, you can write the actual controllers. This process was fairly straightforward since my controllers are still quite simple. It involved writing several simple methods for each type of request. I based my controllers off of those described in [this screencast](https://www.youtube.com/watch?v=yMCj4U0aXq4). Most of it is fairly simple to understand once you have a basic understanding of the models and how to persist them. The one exception is understanding how to create a new entity that can be saved to the database. The way this was accomplished was through the use of a `Form`. I think the main difficulty was that the Form is generally tied to the view, which I haven't worked with yet. From my understanding, generally speaking, a user will enter information into a form online, then that information is put into a Form object which is then used to create a new entity. However, since I do not have a view yet, the information must be put into the Form object in some other way, which I will discuss more in the next section.

## Controller Testing
At first, testing the controllers seemed pretty straightforward. Basically, you call the methods that you wrote for your controllers, and you see if they output the expected results. Everything seemed to go smoothly until I tried to use the methods that used Forms to create new entities. In order to create a Form, you must supply it with some data. The way to do this is to provide a [map](http://en.wikipedia.org/wiki/Associative_array) of the data. Both the keys and the values are Strings where the key corresponds to the field (e.g. "name") and the value is the actual value that you want to fill that field with (e.g. "Todd"). At first, this didn't seem like a problem until I tried to use forms for more complex models. For example, in the book exchange that I described in my [last post]({{ site.baseurl }}{% post_url 2013-04-10-playing-with-play-ii %}), a Request requires both a Book and a Student. However, the form can only accept Strings. It seems to be able to handle numbers and automatically convert them, but it wasn't clear how to handle custom models such as a Book.

### Modifying Models
After some discussion with fellow students who are working on this same problem, the first approach was to modify the models. Instead of, for example, a Request requiring a Book, the Request keeps track of a `bookId` and that is used to find the Book from the database when the form is validated. This seemed like a decent solution, but it would require keeping track of extra information and doing extra database queries to find everything. Instead I tried a slightly different approach. After some testing, I found that when a Form is used to create a new entity, it calls the setters. My theory was that instead of keeping track of the `bookId`, I could just modify the `setBook` method to take a String and then query the database in that method. This would at least eliminate the need to keep track of extra information. To test my theory I created a new Play project and created a simple data model. The model had only books and students and the books required a student. The `setStudent` method of the Book model would take a String which was a `studentId` and find a student from the database. I was very pleased to see that this worked, so I went back to my book exchange project and updated the models. Unfortunately, for reasons that are still unclear to me, this did not seem to work anymore.

### Fixing Forms
After several days of frustration, another student found an alternative solution that seems to work with some limitations. The solution is based on samples provided with the Play Framework, which can be found [here](https://github.com/playframework/Play20/tree/master/samples/java/forms). Basically the solution is to use the Form to set information in the required fields in a model. It's probably best to explain with an example. Let's say a Request requires a String called `requestId` and a Student called `student`. A Student requires a String called `studentId` and another String called `name`. If you want to create a new Request using a Form, you would add the following data to the map.

```java
data.put("requestId", "Request-01");
data.put("student.studentId", "Student-01");
data.put("student.name", "Name");
```

This will create a Student with the specified `studentId` and `name`, then create a Request with the specified `requestId` and `student`. So far, this approach seems to work if you want to create a new Student. However, I have not yet figured out how to use an existing student with the Request.

## Reflection
Learning about controllers has really helped fill out my knowledge and understanding of the MVC architecture. I can definitely see how it can be a powerful way to develop web apps. However, so far I have not been impressed with the Play Framework. It seems to be causing me more frustration than it is worth. However, I will continue to work on it for a few more weeks. Perhaps I will have a new found appreciation for the framework after working with views.

Due to the unexpected problems with testing my controllers, I have not had time to update my warehouse project. For now, I will likely just focus on the book exchange project. As before, the project can be found on [GitHub](https://github.com/ttaomae/book-exchange).

