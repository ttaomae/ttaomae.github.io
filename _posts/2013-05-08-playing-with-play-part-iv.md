---
layout: post
title: Playing With Play Part IV - Creating Views
categories: software-engineering
tags: ics-691 java play-framework
---
This post will likely conclude my [journey](/posts/tag/play-framework) with the Play Framework. Although I have not completely finished the application that I originally set out to create, I am satisfied with what I have learned about the [MVC architecture](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), the Play Framework, and web application development in general.

## View Development
This week, I focused on view development, which is the final component of the MVC architecture. Views are pretty self-explanatory; they are basically what is going to be displayed in the browser. As before, I followed a [screencast](https://www.youtube.com/watch?v=MTtbnzxhYAM) as the basis for my own application.

### Integrating Twitter Bootstrap
Before I started actually creating my own views, I incorporated Twitter Bootstrap into Play. To do this, I made the following changes, taken from the screencast. I added the following highlighted lines to the `project/Build.scala` file.

```scala
import sbt._
import Keys._
import play.Project._

object ApplicationBuild extends Build {

  val appName = "book_exchange"
  val appVersion = "1.0-SNAPSHOT"

  val appDependencies = Seq(
    // Add your project dependencies here,
    javaCore,
    javaJdbc,
    javaEbean,
    "org.webjars" % "webjars-play" % "2.1.0",
    "org.webjars" % "bootstrap" % "2.3.1"
  )

  val main = play.Project(appName, appVersion, appDependencies).settings(
    // Add your own project settings here
    //resolvers += "webjars" at "http://webjars.github.com/m2"
  )

}
```

I also added the following file to the `app/views/helper` directory.

```html
@(elements: helper.FieldElements)

@*
Generate input according twitter bootstrap 2.0 rules.
To use, include the following in your view file:
  @implicitFieldConstructor = @{ FieldConstructor(twitterBootstrapInput.f) }
*@

<div class="control-group @if(elements.hasErrors) {error}">
  <label class="control-label" for="@elements.id">@elements.label</label>
  <div class="controls">@elements.input
    <p class="help-inline">@elements.infos.mkString(", ")</p>
    @if(elements.hasErrors) { <p class="help-block">@elements.errors.mkString(", ")</p> }
  </div>
</div>
```

### Creating Views
In general, creating views with the Play Framework is pretty simple if you have a decent understanding of HTML. Views in the Play Framework are sort of a mix between Scala and HTML files. You can create a view with just pure HTML, but Play allows you to pass in parameters and call other views, sort of like you would call a method.

For example, I have a two views called `main.scala.html` and `index.scala.html` which are shown below.

```html
@(title: String)(content: Html)

<!DOCTYPE html>

<html>
  <head>
    <title>@title</title>
    <link rel="stylesheet" media="screen" href="@routes.WebJarAssets.at(WebJarAssets.locate("css/bootstrap.min.css"))">
    <link rel="stylesheet" media="screen" href="@routes.Assets.at("stylesheets/main.css")">
    <link rel="shortcut icon" type="image/png" href="@routes.Assets.at("images/favicon.png")">
  </head>
  <body>
    <div class="navbar navbar-fixed-top">
      <div class="navbar-inner">
        <div class="container">
          <a class="brand" href="@routes.Application.index()">Student Book Exchange</a>
          <div class="navbar-content pull-right">
            <a class="btn btn-primary">Log in</a>
          </div>
        </div>
      </div>
    </div>
    @content
  </body>
</html>
```

```html
@main("Student Book Exchange | Homepage") {
  <div class="container">
    <div class="page-header">
      <h1>Welcome to the Student Book Exchange</h1>
    </div>
    <div class="row">
      <div class="span8 offset2">
        <div class="well">
          <h3>Save money by exchanging books with your fellow students instead of paying outrageous book store prices!</h3>
        </div>
      </div>
    </div>
    ...
  </div>
}
```

Basically, anything that begins with an `@` will be replaced with the appropriate content. For example, the main view has a parameter named title, which is a String (line 1). The index view calls the main view with the argument `"Student Book Exchange | Homepage"` so the index page, will contain everything in the main view, except with `@title` (line 7) replaced with the appropriate text. Similarly, `@content` (line 23) will be replaced with everything in between the curly braces in the index view. Line 16 from the main view provides another example. In this case, the `href` will be replaced with whatever the appropriate URL is based on that particular route.

Another more complex example can be seen in my file `bookList.scala.html`

```html
@(books : List[models.Book])

@main("Student Book Exchange | Find Books") {
<div class="container">
  <div class="page-header">
    <h1>Available Books</h1>
  </div>
  <div class="row">
    <div class="span12">
      <table class="table table-hover">
        <thead>
          <tr>
            <th>Book ID</th>
            <th>ISBN</th>
            <th>Title</th>
            <th>Edition</th>
            <th>Bookstore Price</th>
            <th>Offers</th>
            <th>Requests</th>
          </tr>
        </thead>
        <tbody>
          @for(book <- books) {
            <tr>
              <td><a href="@routes.Book.edit(book.getPrimaryKey())">@book.getBookId()</a></td>
              <td>@book.getIsbn()</td>
              <td>@book.getTitle</td>
              <td>@book.getEdition</td>
              <td>@book.getPrice</td>
              <td>@if(book.getOffers().isEmpty()) {X} else {O} </td>
              <td>@if(book.getRequests().isEmpty()) {X} else {O} </td>
            </tr>
          }
        </tbody>
      </table>
    </div>
  </div>
  <div class="row">
    <a href="@routes.Book.create()" class="btn btn-block btn-success">Add Book</a>
  </div>
</div>
}
```

Notice that this view has a parameter which is a list of books. This allows us to do some interesting things in the view. For example, on line 23, there is a loop so that we can list all of the books. The information in the `td` tags will be replaced with whatever each method call returns. You can even use conditionals as shown in lines 30 and 31.

Unlike the main view, which is called by another view, the bookList view is called by the following method in the Book controller.

```java
public static Result index() {
  List<models.Book> books = models.Book.find().findList();
  return ok(bookList.render(books));
}
```

Another powerful feature of Play is its [form helpers](http://www.playframework.com/documentation/2.1.1/JavaFormHelpers). The following is an example usage of the form helpers in my file `bookCreate.scala.html`.

```html
@(bookForm: Form[models.Book])

@import helper._
@implicitFieldConstructor = @{ FieldConstructor(twitterBootstrapInput.render) }

@main("Student Book Exchange | Create Book") {
<div class="container">
  <h1>Create Book</h1>
  <div class="span6">
    <div class="well">
      @form(routes.Book.save(), 'class -> "form-horizontal") {
        @inputText(bookForm("bookId"), '_label -> "ID")
        @inputText(bookForm("title"), '_label -> "Title")
        @inputText(bookForm("edition"), '_label -> "Edition")
        @inputText(bookForm("price"), '_label -> "Bookstore Price")
        @inputText(bookForm("isbn"), '_label -> "ISBN")

        <div class="control-group">
          <div class="controls">
            <input id="create" type="submit" value="Create" class="btn btn-primary">
            <a href="@routes.Book.index()" class="btn">Cancel</a>
          </div>
        </div>
      }
    </div>
  </div>
</div>
}
```

It shouldn't be too hard to decipher how the `form` and `inputText` helpers work to create an actual HTML form. (See the link at the end of this post for screenshots of the application.)

## View Testing
I based my tests on the same screencast that I mentioned earlier. The tests use [FluentLenium](http://fluentlenium.github.io/FluentLenium/).

Writing tests for the views requires two steps. First, I created a `FluentPage` for each view that I was testing. Below is an example of a `FluentPage` for the index view.

```java
package pages;

import static org.fest.assertions.Assertions.assertThat;
import org.fluentlenium.core.FluentPage;
import org.openqa.selenium.WebDriver;

public class IndexPage extends FluentPage {
  private String url;

  public IndexPage(WebDriver webDriver, int port) {
    super(webDriver);
    this.url = "http://localhost:" + port;
  }

  @Override
  public String getUrl() {
    return this.url;
  }

  @Override
  public void isAt() {
    assertThat(title()).isEqualTo("Student Book Exchange | Homepage");
  }

  public void gotoViewBooks() {
    click("# viewBooks");
  }

  public void gotoViewStudents() {
    click("# viewStudents");
  }

  public void gotoViewRequests() {
    click("# viewRequests");
  }

  public void gotoViewOffers() {
    click("# viewOffers");
  }
}
```

The `getUrl` method is self-explanatory and the `isAt` method simply tests that you are on the correct page. The other methods provide a way to interact with the page. In this example, you can only click on different links. On other pages, you can fill out and submit forms.

The next step is to write the actual tests. Here is an example of the test for the index view.

```java
@Test
public void testIndexPage() {
  running(testServer(PORT, fakeApplication(inMemoryDatabase())), HTMLUNIT, new Callback<TestBrowser>() {
    @Override
    public void invoke(TestBrowser browser) {
      IndexPage homepage = new IndexPage(browser.getDriver(), PORT);
      browser.goTo(homepage);
      homepage.isAt();

      // return to homepage before clicking each link
      browser.goTo(homepage);
      homepage.gotoViewBooks();
      assertThat(homepage.title()).isEqualTo("Student Book Exchange | Find Books");

      browser.goTo(homepage);
      homepage.gotoViewStudents();
      assertThat(homepage.title()).isEqualTo("Student Book Exchange | Find Students");

      browser.goTo(homepage);
      homepage.gotoViewRequests();
      assertThat(homepage.title()).isEqualTo("Student Book Exchange | Find Requests");

      browser.goTo(homepage);
      homepage.gotoViewOffers();
      assertThat(homepage.title()).isEqualTo("Student Book Exchange | Find Offers");
    }
  });
}
```

The first few lines are boilerplate code before you can actually start the test. The first three lines of the `invoke` method are a simple test to see if you can navigate to the correct page. The remainder of the method checks to see that you can click each link and that it will take you to the correct page.

The rest of the tests are a little more complicated, but the basic idea is the same. Navigate to a page, perform some action, and check that the action was performed correctly.

## Reflection
Despite the various problems that the Play Framework has given me over the past month, I have to admit that I am quite pleased with what I was able to produce with it. It isn't quite what I had planned and described in my [last post]({{ site.baseurl }}{% post_url 2013-04-26-a-break-from-play %}) but it is far better than anything that I could have created before using Play.

Having finally learned about all three components of the MVC architecture, I have mixed feelings about Play. I think the MVC architecture is a great paradigm for web application development, but I think Play needs some time to mature. Play is a little newer than alternatives such as Ruby on Rails and Django, and Play has had some major changes fairly recently, such as rewriting the framework core to use Scala instead of Java. If I ever decide to develop another web application, I will think twice about using Play. It's not out of the question, but I will definitely consider some alternatives.

It may seem strange that I've spent all this time talking about views and yet I haven't actually shown any. For that, I direct you to the [GitHub page](http://ttaomae.github.io/book-exchange/) where you can also download the entire source code if you're interested.

