---
layout: post
title: Testing Out Test-Driven Development
categories: software-engineering
tags: java testing
---
In the past I've talked a little about some of my experiences with software testing. However, my experience with any formal approaches to testing have been fairly limited. In this post I will describe my first experience with test-driven development.

## Test-Driven Development
[Test-driven development](http://en.wikipedia.org/wiki/Test-driven_development) is a method of software development that is based on writing tests before writing the code that is being tested. The basic development cycle, as I understand it, is as follows: write a test case, which will initially fail (since you have not implemented the feature being tested); write the minimal amount of code needed to pass the test; add another test that will cause the newly added code to fail; add code to pass the tests; repeat. At this point, the code is only meant to pass the tests. After all functionality is implemented, you can refactor and clean up the code. If your tests are written well, you can make changes and be confident that your code remains correct, as long as it continues to pass all tests.

### Test-Driven FizzBuzz
One of my [first posts]({{ site.baseurl }}{% post_url 2013-01-16-fizzbuzz %}) was about writing and testing [FizzBuzz](http://en.wikipedia.org/wiki/Fizz_buzz). Below, I will try to briefly demonstrate using test-driven development (with [JUnit](http://en.wikipedia.org/wiki/JUnit) for testing) to implement FizzBuzz.
The goal of FizzBuzz is typically to print the numbers from 1 to 100 with some rules for certain numbers. However, for the sake of demonstration and testing, I will just write a method which takes an integer and returns a String based on the rules of FizzBuzz. In FizzBuzz, for most integers, we just want the String representation. 

The first step of test-driven development is to write a minimal test case, which is shown below.

```java
@Test
public void testGetValue() {
    assertEquals("11", FizzBuzz.getValue(11));
}
```

Since we have not written the `getValue` method, the test will fail. To pass the test we can write the following code.

```java
public static String getValue(int num) {
    return "11";
}
```

Now our test will pass but our code is obviously wrong, so we add another test case which will fail.

```java
@Test
public void testGetValue() {
    assertEquals("11", FizzBuzz.getValue(11));
    assertEquals("77", FizzBuzz.getValue(77));
}
```
Then we write code that will pass the tests.

```java
public static String getValue(int num) {
    return String.valueOf(num);
}
```


The next requirement of FizzBuzz is that if a number is divisible by 3, then it should return "Fizz" instead. We can add the following test, which will of course fail.

```java
@Test
public void testFizz() {
    assertEquals("Fizz", FizzBuzz.getValue(3));
}
```
Then we write code to pass the test.

```java
public static String getValue(int num) {
    if (num == 3) {
        return "Fizz";
    }
    return String.valueOf(num);
}
```

You've probably guessed that the next steps are to write a new test, then fix the code to pass the test.

```java
@Test
public void testFizz() {
    assertEquals("Fizz", FizzBuzz.getValue(3));
    assertEquals("Fizz", FizzBuzz.getValue(33));
}
```
```java
public static String getValue(int num) {
    if (num % 3 == 0) {
        return "Fizz";
    }
    return String.valueOf(num);
}
```

The next requirement of FizzBuzz is that for numbers that are divisible by 5, it should return "Buzz." The next few iterations of development might look like this:

```java
@Test
public void testBuzz() {
    assertEquals("Buzz", FizzBuzz.getValue(5));
}
```
```java
public static String getValue(int num) {
    if (num % 3 == 0) {
        return "Fizz";
    }
    if (num == 5) {
        return "Buzz";
    }
    return String.valueOf(num);
}
```
```java
@Test
public void testBuzz() {
    assertEquals("Buzz", FizzBuzz.getValue(5));
    assertEquals("Buzz", FizzBuzz.getValue(55));
}
```
```java
public static String getValue(int num) {
    if (num % 3 == 0) {
        return "Fizz";
    }
    if (num % 5 == 0) {
        return "Buzz";
    }
    return String.valueOf(num);
}
```

FizzBuzz's last requirement is that it should return "FizzBuzz" for numbers that are divisible by both 3 and 5. The end result might look something like this:

```java
@Test
public void testBuzz() {
    assertEquals("FizzBuzz", FizzBuzz.getValue(15));
    assertEquals("FizzBuzz", FizzBuzz.getValue(90));
}
```
```java
public static String getValue(int num) {
    if (num % 3 == 0 && num % 5 == 0) {
        return "FizzBuzz";
    }
    if (num % 3 == 0) {
        return "Fizz";
    }
    if (num % 5 == 0) {
        return "Buzz";
    }
    return String.valueOf(num);
}
```

At this point, the code passes all the tests and you can be fairly confident that it is correct. However, you may decide that you want to fix or refactor the code for some reason. For example, you may notice that you are checking if the number is divisible by 3 or 5 twice, so you make the following change:

```java
public static String getValue(int num) {
    if (num % 15 == 0) {
        return "FizzBuzz";
    }
    if (num % 3 == 0) {
        return "Fizz";
    }
    if (num % 5 == 0) {
        return "Buzz";
    }
    return String.valueOf(num);
}
```
If you're not a fan of multiple return statements, you might completely redo the method.

```java
public static String getValue(int num) {
    String result = "";

    if (num % 3 == ) {
        result += "Fizz";
    }
    if (num % 5 == 0) {
        result += "Buzz";
    }
    if (result.equals("")) {
        result += num;
    }

    return result;
}
```

As long as you continue to pass all tests, you can change your code in (almost) any way that you want and remain confident that you haven't broken anything.

## Redoing Connect N
For my first real project using test-driven development, I decided to redo a [project](http://toddtaomae.wordpress.com/connect-n/) from a few years ago, which I wanted to improve on. Redoing an old project allowed me to focus more on the testing rather than figuring out what classes I would need to create or how to implement certain features. Another reason I chose this project was so that I could get some experience with [JavaFX](http://en.wikipedia.org/wiki/JavaFX), which I will hopefully talk about in a future post. JavaFX is a platform for creating [rich Internet applications](http://en.wikipedia.org/wiki/Rich_Internet_Application) and, from my understanding, is more or less intended to replace [Swing](http://en.wikipedia.org/wiki/Swing_%28Java%29).

Although I made sure not to look at my old code, I did still remember the basic structure of the code, so a lot of it is very similar. The following discussion will try to compare my previous experience with this project and my current experience using test-driven development.

Throughout the development process, I definitely saw the advantages of test-driven development. The obvious one is the one that I already mentioned. Any time I wanted to make a change to some existing code, I could do so quickly and without worry of breaking anything. Of course, it also helps that I was using Git as version control, so I knew I could always revert back to a working version if anything went wrong. However, version control doesn't give you confidence that your code is still producing the correct output.

Another advantage that I didn't expect was that creating tests helped to guide the development process. It prevented me from getting too far ahead of myself since I would need to write tests before I writing any features. I sometimes think about methods or features that I *might* need and they start to clutter my thoughts rather than focusing on the features I need now. Writing tests first also prevents me from getting caught up in the details of *how* to implement certain features. Instead I can focus on *what* I want and then think about how to implement it. For example, the first thing I wanted to test was that a new game board is empty. I created a `Board` class, but I didn't have to worry yet about how to actually represent the board. In order to test that the board is empty, I would need some representation of what is on the board. For that I created a `Piece` enumeration. After testing an empty board, I wanted to test what happens when I play some pieces. If I only play one piece, I could sort of "fake it" (in the same vein of the first version of the FizzBuzz example). However, after that it became clear that I needed to have some representation of the Board. I decided to go with a 2-dimensional array, which is the same as in my original version.

Although I don't remember my exact process to arrive at a similar point during my original attempt at this project, I probably decided before I even wrote any code that I would use a 2-dimensional array to represent the board. In this case it's not much of a problem, since the mapping from the game board to a 2-dimensional array is a pretty obvious choice. However, if the choice is not as obvious, choosing too early could lead to problems down the road. The idea of writing a minimal amount of code means that you don't have to worry about the details until they actually become relevant. Testing your code also has the advantage of making it easier to refactor your code. For example, if I decided right now that I wanted to use a 1-dimensional array instead, I would much rather do it on my new, *tested* version. I could rip out chunks of code, which would obviously cause my tests to fail, but when I start rewriting everything I would have small goals that I can strive for. I can write small bits of code that will each pass one more test, until I pass all my tests and I know that my code is correct again.

### Testing Random Behavior?
One issue that I came across was testing random behavior. For example, I created a simple player that would select a random, but valid move. My tests only verify that the random player selects a valid move; they do not test anything related to the random behavior. What would a test of random behavior look like, anyway? In my case, it's basically just a random number generator with some restrictions. I could maybe test that the random numbers are evenly distributed, but that is basically just testing the underlying random number generator rather than the code I wrote.

Another related issue that I've considered is testing multi-threaded software, which is inherently non-deterministic. Concurrency is a huge topic that I won't get into right now, but if you're not familiar with it I will try to briefly describe some of the difficulties. The main issue is that you do not know or have control of how the operating system will schedule your threads. There are problems such as [race conditions](http://en.wikipedia.org/wiki/Race_condition) and [deadlocks](http://en.wikipedia.org/wiki/Deadlock) that can occur if you are not careful. Of course there are ways to prevent these situations, but when you have multiple threads sharing different data, it can be very difficult to manage it all. Even if you think you have everything correct, it can be hard to tell. Some bugs will only occur once in a while when the threads are scheduled in a particular way, so testing might not even help in these situations.

### Testing GUIs?
Another thing that I haven't quite figured out yet is how to test GUIs, or if they should be tested at all. I've tried to keep the game logic out of the GUI, so that's not an issue, but the GUI components have their own logic for how to display certain things or how to deal with actions (e.g. mouse clicks). So far I've just relied on visual inspection. It works well enough for this project, but for a more complex interface I would imagine you would want a more formal method of verifying that everything is correct.

## Reflection
I've had a very positive experience with test-driven development so far. I have seen its benefits in a relatively small project and I'm sure the benefits are even greater in a more complex project. I'm not sure if I will fully adopt test-driven development, but I will definitely try to incorporate more testing into my development cycle.

Of course I still have a lot to learn about test-driven development and testing in general, as evidenced by the fact that I haven't quite figured out how to test all aspects of my project. Perhaps the issue is not with the testing, but rather the design of the software being tested. I'm sure there are a lot of improvements that could be made to my code that might make it more testable.

As usual, the project can be found on [GitHub](https://github.com/ttaomae/ConnectN).

