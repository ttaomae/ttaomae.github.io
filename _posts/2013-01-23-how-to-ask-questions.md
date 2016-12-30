---
layout: post
title: How to Ask Questions
categories: miscellaneous
tags: ics-691
---
Do you know how to ask a question? It seems simple, right? Well, it is. Anyone can ask a question. On the other hand, asking a *good* question is a completely different matter. In particular, it can be very difficult to ask a good question about a technical problem.

## How to Ask *Good* Questions
There are countless guides, tutorials, or other documents online that will attempt to teach you how to ask good questions and receive good answers. One such guide is ["How To Ask Questions The Smart Way."](http://www.catb.org/esr/faqs/smart-questions.html) This somewhat lengthy guide describes the entire process of posing a question to the "hacker" community. A second, similar article is ["Getting Answers,"](http://www.mikeash.com/getting_answers.html) which is primarily targeted towards programmers, but attempts to provide general advice to anyone hoping to get useful answers to their questions. Although the first guide provides more depth, I found it to be too specific and verbose. If you are looking specifically to ask hackers for help, it is definitely a good idea to read the guide to introduce yourself to some of the etiquette of the hacker community. If you are looking for general advice on how to get your questions answered, I would recommend taking a look at "Getting Answers."

Of course, neither guide provides a truly comprehensive list of all things to consider when asking a question. Below I summarize what I found to be the best pieces of advice from both articles.

#### Write clearly and use proper grammar and spelling
This may seem obvious, but it is easy to forget how important this is, especially when writing in an informal setting on the internet. No one wants to struggle through poorly a written question with unclear meaning, unnecessary abbreviations, poor spelling, or improper punctuations. A question that uses proper grammar and spelling is more likely to be read and therefore answered. A question that is clearly written is more likely to be understood and therefore answered *correctly*.

#### Do your research
Before you even consider asking someone else for help, you should see what you can figure out on your own. When you first encounter a problem, it may be a good idea to turn to Google rather than to a friend or your forum of choice. You may find that you can solve the problem on your own. If you still can't solve the problem, doing research will help you ask better, more specific questions. The research doesn't stop once you ask your question. If you receive a response that you don't understand, instead of just asking for an explanation, see if you can figure it out on your own.

#### Be specific
When you ask your question, be sure to include as much information as possible. If you have a programming question, for example, you may want to provide the relevant code. This may prevent a long line of extra questioning just to figure out what you really need. Along those same lines, be sure that you are asking the right question. Often times someone will have a problem X and they think method Y is the best solution. When they run into problems, they ask about Y, when asking about X would have led to a better solution. This is known as the ["XY Problem."](http://www.perlmonks.org/?node_id=542341)

#### Explain what you know/tried
Let's say you've already done your research and tried to solve your problem on your own. When you finally turn to someone else for help, be sure to explain what you know from your research and to describe your attempts to solve your problem. This will make the process easier for you and the person answering your question since they will not suggest things that you've already tried. This will also demonstrate to the person answering your question that you have actually done research and tried to solve the problem on your own.

#### Consider the answer
Even if you think an answer is wrong, take a minute to consider it. After all, if you knew the right answer, you wouldn't be asking for help. If you assume that the answer is wrong, you may end up wasting time looking for another answer, when the first one was correct all along.

#### Follow up
After you've solved your problem, it is usually a good idea to follow up on what your final solution was. It will provide a sense of closure for those who helped you and if you are posting to an online forum, it may help others in the future who have a similar problem.

#### Be courteous and respectful
It doesn't hurt to include a "please" and "thank you" in your question. Although this will not ensure a response, it may make others more willing to help you if you are polite. It is also a good idea to thank those who offered their help, even if they didn't provide the right solution.

## Fixing a bad question
In order to further investigate each of the above criteria for a good question, I will look at a bad question and see how well it fulfills each of them.

To find a bad question, I went to [Stack Overflow](http://stackoverflow.com/), which is a website where users can ask and answer questions about programming. I then looked at the newest questions which were tagged as being about Java and selected one with a negative rating. [This](http://stackoverflow.com/questions/14432665/java-arrays-2dimension) is what I found.

<blockquote>
Hi I need to store the elements in single dimension array into two dimension and moreover, If Suppose single dimension consists of 9 elements i need to store 3 elements in first row and next 3 elements in second row and same as , can anyone help me and guide me.

thank You I hope anyone will guide me .

</blockquote>
The first thing to notice is the poor grammar and punctuation. If the user put a little more time into writing the question, it would be easier to read and understand. It is possible that English is not the user's first language. If this is the case, it might be good to let the readers know this. However, this does not mean that you do not have to put effort into writing clearly.

This user clearly did not do his research. A quick trip to Google probably would have turned up a solution in a few minutes. After he received an answer, he continued not to do research. His response to one answer was, "sorry i didn't get you can you little bit more clear idea....."

This question is fairly vague. The user could definitely include more details. For example, does he know the size of the one-dimensional array in advance? Will the width of the two-dimensional array always be 3? The user may also want to explain why he is trying to do this. It may reveal that he actually has an XY Problem.

This user did not explain what he tried. For all we know, he didn't try anything. Within a few minutes, the first response was simply, "whathaveyoutried.com" which redirects [here](http://mattgemmell.com/2008/12/08/what-have-you-tried/).

This user did not even get to the point where he could consider the answer and follow up because the question was "closed as not a real question" before he got a satisfactory answer.

The one thing that this question did right was that it included a "thank you." However, this is probably one of the least important criteria for a good question.

Below I provide a revision of the question which attempts to fulfill most of the criteria.

<blockquote>
Hi. I need to store the elements of a one-dimensional array into a two-dimensional array. For example, suppose the one-dimensional array consists of 9 elements. I need to store 3 elements in the first row, 3 elements in the second row, and so on. The one-dimensional array can be of arbitrary length. The length of the rows in the two-dimensional array will always be 3.

This is what I've tried so far, but it doesn't work when the one-dimensional array has less than 3 elements.
[example code]

Can anyone help me? Thank you.

</blockquote>
With just a little bit of extra effort the question becomes much more clear and easy to read. Adding some details about the problem will help you get the answer you are looking for. By adding example code, you demonstrate that you have put some effort into solving the problem on your own. There is still room for improvement, but overall, the question is now much more appealing to read and to answer.

