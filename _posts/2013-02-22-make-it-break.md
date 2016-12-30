---
layout: post
title: Make it Break
categories: software-engineering
tags: ics-691 java
---
Quality assurance is an important part of developing any system, product, or service; developing a software system is no exception. Quality assurance can be divided into two broad categories: automated and manual. Manual quality assurance for software systems includes, for example, unit testing and code reviews. Automated quality assurance involves the use of tools such as [static code analysis](http://en.wikipedia.org/wiki/Static_program_analysis) tools.

## Introduction
Both forms of quality assurance have advantages and disadvantages. Both are able to detect certain flaws that the other cannot. For example, manual quality assurance can find deficiencies in requirements, while automated tools cannot. On the other hand, automated tools can find flaws that may be too complex for a person to find. Another important strength of manual quality assurance is that there is a fairly low chance for false positives, whereas an automated tool has a much higher chance of giving false positives. Perhaps the main strength of automated tools is that they can be reused for many projects with very low overhead.

Today, I will be looking at three automated quality assurance tools for Java: [Checkstyle](http://checkstyle.sourceforge.net/), [PMD](http://pmd.sourceforge.net/), and [FindBugs](http://findbugs.sourceforge.net/). Checkstyle and PMD use static analysis on Java [source code](http://en.wikipedia.org/wiki/Source_code) and FindBugs analyzes Java [bytecode](http://en.wikipedia.org/wiki/Bytecode). In order to get a better understanding of these tools, I will write a very short program which compiles but causes each of these tools to throw at least five unique errors.

## Checkstyle
Checkstyle is a tool which helps to enforce coding standards. Generally, [coding standards](http://en.wikipedia.org/wiki/Coding_standards) are put in place to help ensure that code is readable. Arguably, readable code is nearly as important as correct code. Enforcing standards ensures that everybody working on a software project will produce code that everyone else can read. Having easily readable code is important because a large portion of software development involves reading and maintaining code that is often written by someone else. Because coding standards can vary between projects and organizations, Checkstyle requires a configuration file that specifies what the specific coding standards are.

Checkstyle was the easiest of the three tools to make complain. Because I am familiar with the coding standards being used for this project, I mostly just wrote code that did not comply to these standards. However, before even writing any code, Checkstyle complains that my project was missing a package-info.java file.

Below, I have included the relevent parts of code that cause Checkstyle to throw errors. An elipsis (...) refers to code which I have excluded as they do not influence the errors given by Checkstyle.

```java
import java.io.*;

public class makeitbreak extends Object implements Serializable {
...
  public makeitbreak()
  {
    if (true) {
      {
        if(false);
      }
    }
  }
}
```

The second error (after the missing package-info.java error) is the use of the wildcard (*) in imports. The next error is that there are no Javadoc comments for the class or the constructor. Fixing these errors would generally help to make the code easier to understand. This is particularly true for the error regarding Javadoc comments. Including proper comment can make it very clear what is trying to be accomplished without having to read through all of the code.

There is another error which says to "avoid nested blocks." In this case, it is referring to the pair of curly braces inside of the first if statement because they do not add anything to the structure of the code and instead make it harder to understand.

There are two errors that are strictly related to formatting. The first is that the opening parenthesis of the constructor should not be on a line of its own and the second is that there should be a space between the if keyword and the opening parenthesis. These errors only help to ensure that all developers write in a consistent style, rather than helping to make the code easier to understand.

Checkstyle definitely has its uses, but there may be better ways of enforcing coding standards. For example, [Eclipse](http://www.eclipse.org/) has a built, in automatic formatter which can prevent a lot of the errors that Checkstyle would find.

## PMD
Although PMD also uses source code analysis, it performs a very different task than Checkstyle. Rather than simply looking at the formatting and style of the code, it actually performs some analysis of the semantics of the code. PMD will give errors that are in some ways related to style, but they are geared more towards preventing possible bugs and making it easier to understand the code. This is in contrast to Checkstyle which is more geared towards formatting and readability.

Below, I have provided the relevant portion of my code which will cause PMD errors.

```java
...
public class makeitbreak extends Object implements Serializable {
  ...
  public makeitbreak()
  {
    if (true) {
      {
        if(false);
      }
    }
  }
}
```

The first error is that the class extends Object. Since all classes, implicitly extend Object, it is not necessary to explicitly state it.

The next error comes from the two if statements. PMD refers to these as unconditional if statements because they will always evaluate to the same value, so there is no condition. Since there is no condition, the code will either always execute (if it is true) or never execute (if it is false), which makes the if statement unnecessary.

The second if statement produces three more errors. First, PMD complains that there is an empty statement (i.e. the semicolon immediately following the if statement) that is outside of a loop. It also complains, more specifically, that there is an empty if statement. PMD recognizes that this bit of code does not do anything, which makes it unnecessary. The last error for this if statement is that it does not use curly braces. Although curly braces are not syntactically required by Java for if statements, it is a common to include them as it reduces the chances of a particular bug, which I will briefly illustrate below.

```java
if (foo == 3)
  bar();
```

In the above (syntactically correct) snippet of Java code, the method `bar()` will only execute if the variable `foo` is equal to 3. However, suppose you also want to execute the method `baz()` when the condition is true. To accomplish this, you might write the following code.

```java
if (foo == 3)
  bar();
  baz();
```
At first, this might seem correct, but with this code, `baz()` will execute regardless of whether or not the condition is true. In Java, only one statement, rather than an entire block of code, is executed after an if statement without curly braces. If you want to execute multiple lines of code after an if statement, you need to put it inside a pair of matching curly braces, as shown below.

```java
if (foo == 3) {
  bar();
  baz();
}
```

Based on my experience so far, PMD seems to be a very valuable tool. Although it will not be able to prove that your code is correct, it seems to be able to detect potential logic errors (such as unconditional if statements) and maybe even prevent future bugs (such as requiring curly braces on if statements).

## FindBugs
FindBugs differs greatly from the previous two tools in that it uses bytecode analysis rather than source code analysis. Based on this alone, we know that it will not be able to detect a lot of the bugs that Checkstyle and PMD can find since those tools rely on the formatting and syntax of your code. This does not mean that it is worse than the other tools. It just means that it will find different types of errors.

Again, I have provided the relevant parts of my code.

```java
...
public class makeitbreak extends Object implements Serializable {
  int serialVersionUID = 1;

  public makeitbreak()
  {
    String S = new String("S");
    ...
  }
}
```

The first error that FindBugs gives me is that the class name does not begin with an upper case letter. I found it a little odd that it would complain about this since it should not matter in the bytecode what the names of classes or variables are.

Another error was `serialVersionUID` is not static; `serialVersionUID` is used by the Serializable interface and it is expected to be static. This is probably something that the other tools would not be able to detect because it would probably need to look into the semantics of the Serializable interface.

FindBugs also complains that `serialVersionUID` is an unread field, which means that it is never used. Similar to the unread field error, FindBugs also complains of a [Dead store](http://en.wikipedia.org/wiki/Dead_store) to `S`. Again, it is complaining that a value is never used. Since these variables are never used after being initializeds, they are making the CPU waste cycles on something that has no effect on the program. The last error from FindBugs was that an inefficient constructor was used to instantiate `S`.

FindBugs was able to find errors that the other tools would not be able to. It is more focused on the efficiency and execution of the code as opposed to the other tools which are mainly concerned with readability and understandability.

## Reflection
Below I have provided the full text of the Java class. Although it compiles, it is far from being well-written.

```java
import java.io.*;

public class makeitbreak extends Object implements Serializable {
  int serialVersionUID = 1;

  public makeitbreak()
  {
    String S = new String("S");
    if (true) {
      {
        if(false);
      }
    }
  }
}
```

Good software engineering is about more than producing code that compiles and runs. Even code that is syntactically and semantically correct, can be bad code. Good code does not only provide correct results. It should also be easy to read and understand, and, whenever possible, it should be efficient.

