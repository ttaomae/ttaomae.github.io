---
layout: post
title: Solving the Tetris Cube
mathjax: enable
---
About a year ago someone lent me a [Tetris Cube](http://www.amazon.com/dp/B000PWPAJ4) which has been sitting in my office since then. Being a fan of puzzles I thought it would be fun to tinker away at it occasionally during any down time or if I needed to clear my mind. For a few months I would spend a few minutes here and there trying to solve it. After a while I figured that my puzzle solving skills would not be enough and decided to put my programming skills to use. Since then I have hardly touched the puzzle and I haven't had much time to try to create a solver until recently.

## Polyomino Tiling
Rather than jumping straight into solving the Tetris Cube, I decided that a solving a similar, but much easier, problem might be a good place to start. I decided to try solving polyomino tiling puzzles first. Anyone who has ever played or knows anything about [Tetris](http://en.wikipedia.org/wiki/Tetris) should be familiar with polyominoes. In case you aren't familiar with the game it involves trying to place various falling polyominoes such that you fill up entire rows with no gaps. Polyominoes are simply shapes made from adjacent squares. Tetris uses tetrominoes which are polyominoes consisting of four squares.

The goal of a polyomino tiling puzzle is similar to Tetris except instead of dealing with random falling shapes, you are given a set of polyominoes which you must place in order to fill up a specified shape. Usually you can modify the polyominoes by rotating them by 90 degrees at a time or by flipping them over.

### Backtracking
My initial thought was that this should be easy to solve with a backtracking algorithm. Backtracking is probably better explained in relation to a different problem, such as the [eight queens puzzle](http://en.wikipedia.org/wiki/Eight_queens_puzzle). In the eight queens puzzle the goal is to place eight queens on a chessboard such that none of the queens can attack any other queen. You might approach this problem by placing a queen on the first column of the first row, then placing a queen on the third column of the second row (the third column is the first one which does not lead to the queens attacking each other), and so on for the following columns. At some point you won’t be able to place any more queens, which is where you begin to backtrack. You remove the last queen that you placed and move it to the next valid column and try to move down the rows again. Each time you’ve tried all possible moves for a single row, you backtrack one more row and then repeat the process again.

In the case of the tiling puzzle, the rows would be analogous to the different polyominoes and the columns would be analogous to the different positions and orientations of the polyominoes. For example, if we are given three pieces *A*, *B*, and *C*. We might place *A* in the top left corner, then *B* to the right of that. Then we may find that there is no way to place *C* so we rotate *B* and try again. After we’ve tried all rotations of *B* we can move it and try all rotations again. After we’ve tried all positions and rotations of *B* we go back to *A* and rotate it, and so on.

### Implementation
I've had some experience with backtracking before (to solve Sudoku puzzles) so implementing the algorithm wasn't difficult. I actually found it more difficult to decide on a good representation of polyominoes and to implement the methods to rotate and flip the polyominoes.

I ended up with a 2-dimensional boolean array for both the puzzle and the polyominoes. The polyominoes additionally had an x and y position to allow them to move.

After everything was implemented I tested my solver on some very simple problems such as fitting a single piece into a grid of the same shape and fitting for square tetrominoes into a 4x4 square.

Once I was confident that my solver was working I tried it on some of the examples shown [here](http://en.wikipedia.org/wiki/Tetromino) and [here](http://en.wikipedia.org/wiki/Pentomino). I was quite disappointed to find that it took much longer than expected to solve. The 5x8 tetromino puzzle took several minutes and none if the pentomino puzzles finished within the time I let them run (I think I let one run for about an hour before giving up).

My backtracking algorithm at this point was very naive. It would place pieces as I described earlier and only backtrack if there was no space to place the next piece. This would sometimes lead the algorithm down an obviously wrong path. For example, if the first piece is placed in such a way that there is a 1x1 hole in the corner, it is obvious that nothing can fit in there (except a monomino). However the algorithm would continue to fill in other pieces and might be stuck on that path for quite some time. In order to partially alleviate this problem I added a simple heuristic to help prune the search space. Before placing any additional pieces, I would find the size of the smallest connected gap in the puzzle. If the smallest polyomino remaining is larger than the gap, then the algorithm will immediately backtrack. Although this approach was not perfect it did help reduce the run time. However, the pentomino puzzles were still not going to be completed in any reasonable amount of time.

At this point it was pretty clear that this approach probably wouldn't scale well into three dimensions even if I implemented better heuristics for further pruning the search space. So I decided I would need to find another way.

## Algorithm X and Dancing Links
Luckily Wikipedia led me to an answer pretty quickly. I don’t remember exactly how I got there, but it was just a few clicks away from the polyomino page. In particular I was led to the pages on [Knuth’s Algorithm X](http://en.wikipedia.org/wiki/Knuth%27s_Algorithm_X) and [Dancing Links](http://en.wikipedia.org/wiki/Dancing_Links).

### Exact Cover
In order to understand Algorithm X, you must first understand what an [exact cover](http://en.wikipedia.org/wiki/Exact_cover) problem is. I will try to explain it without using too much mathematical or technical terms.

Suppose you have a set of *things* numbered *1* through *10*. You are given a set of options that each contain one or more of those things. For example you might have option A = {1, 2, 5, 6}, option B = {2, 3, 7, 10}, and so on. Notice that you can have the same thing in multiple options. The goal of an exact cover problem is to select a set of options such that each thing is chosen exactly once.

If that was too abstract suppose that the 10 things are different kinds of candy. We want to know all the ways that we can pick five pairs of candy such that each one is selected only once. In this case our options would look something like this: {1, 2}, {1, 3}, …{2, 3}, {2, 4}, … {7, 9}, {8, 9}. We want to pick 5 options such that each number appears only once.

We can create a variation of this problem where we have 10 pieces of candy that we want to distribute to 5 children, such that they each get two pieces.  Let us further suppose that each child only likes certain types of candy. Since we only want to give each child two pieces of candy, we must be sure that we select each child only once. In order to maintain this restriction we should add 5 more things to our initial set. Let us label them *a* through *e*, representing the 5 children.

Suppose child *a* only likes candies 1, 2, 3, and 4. To represent this we would include the options {a, 1, 2}, {a, 1, 3}, {a, 1, 4}, {a, 2, 3}, {a, 2, 4}, and {a, 3, 4}. We would include similar options for each child depending on what their preferences are.

Believe it or not, a polyomino tiling puzzle can be solved by direct analogy to this problem. Instead of children, we are dealing with the different polyominoes and instead of each child having different possible combinations of candy, each polyomino has different combinations of coordinates on the shape that they are trying to fill. So a square tetromino (often referred to as an 'O' tetromino due to its resemblance to the letter) might have options {O, 00, 01, 10, 11}, {O, 10, 11, 20, 21}, and so on, where the first digit of each number represents an x-coordinate and the second digit is a y-coordinate.

### Algorithm X
Now that we understand the exact cover problem and how we can use it to solve a polyomino tiling puzzle, we need a way to solve the exact cover problem. It turns out that this problem is actually pretty difficult. In fact, it is [NP-complete](http://en.wikipedia.org/wiki/NP-complete), which basically means that there is no known way for the problem to be solved in polynomial time. In very simple terms it basically means that as the problem size grows, the runtime of the algorithm will grow very quickly. At this point I simply hoped that my problem was simple enough to be solved in a reasonable amount of time.

Luckily, smarter people than I have worked on solving this problem in a relatively efficient manner. In particular, [Donald Knuth](http://en.wikipedia.org/wiki/Donald_Knuth) developed [Algorithm X](http://en.wikipedia.org/wiki/Knuth%27s_Algorithm_X) to solve this problem. Algorithm X relies on a particular representation of the exact cover problem. It turns out that you can represent an exact cover problem using a binary matrix. For example, going back to (a smaller version of) the candy problem, the following matrix might represent a few options.

<p>\(
\begin{array}{|c|c|c|c|c|c|c|c|c|c|}
  \hline
  & a & b & c & 1 & 2 & 3 & 4 & 5 & 6 \\
  \hline
  bf{A} & 1 & 0 & 0 & 1 & 1 & 0 & 0 & 0 & 0 \\
  \hline
  bf{B} & 1 & 0 & 0 & 1 & 0 & 1 & 0 & 0 & 0 \\
  \hline
  bf{C} & 0 & 1 & 0 & 0 & 1 & 0 & 0 & 1 & 0 \\
  \hline
\end{array}
\)</p>

In this example, **A** represents child *a* receiving candies *1* and *2*, **B** represents child *a* receiving candies *1* and *3*, and **C** represents child *b* receiving candies *2* and *5*.

Given this representation, we can start to understand Algorithm X. If you prefer pseudo-code, you can simply refer to the Wikipedia page. If you want a complete and in depth discussion of the algorithm, you may want to read Knuth’s [paper](http://arxiv.org/abs/cs/0011047). In this post I will try to describe it in plain English.

The first step is to choose a column. Any one will do, but Knuth recommends the one with the fewest 1’s. Then we go down that column and stop at every row where there is a 1. For example, if we chose column *a*, we would stop at rows **A** and **B**. Each time we stop at a row, we delete that row as well as any column with a 1 in that row. So, if we stop at row **A**, we would delete row **A** and columns *a*, *1*, and *2*. The row that we just deleted is part of our partial solution.

We would then repeat this process by selecting a new column (of the ones remaining after the deletions in the previous step). If we reach a point where the matrix is empty, then we’ve found a solution. If we reach a point where there are no columns with any 1’s in it, then we’ve hit a dead end and there is no way to find a solution by continuing to search. Either way, we must backtrack. In this case, that means we undelete everything from the previous step, then move to the next row.

### Dancing Links
Luckily for me, Knuth not only developed an algorithm for solving the exact cover problem, he also described an efficient implementation, since a naive implementation would spend a large portion of the time searching for 1’s. It seems he was more interested in the implementation than the algorithm, as evidenced by his complete lack of effort in naming the algorithm.

He refers to the implementation technique as [Dancing Links](http://en.wikipedia.org/wiki/Dancing_Links). To understand the name, we must talk a little about the data structure that he uses. In order to avoid having to search for 1’s in what will usually be a sparse matrix, only the 1s will be stored. They are stored as a node of a type of [doubly linked list](http://en.wikipedia.org/wiki/Doubly_linked_list), which is where the "links" part of the name come from. The “dancing” part comes from the fact that in the process of deleting and undeleting rows and columns the links will be undone and redone in a sort of "dance" using the technique outlined in the code below.

```java
// remove node from list
node.left.right = node.right;
node.right.left = node.left;

// node still points to its neighbors
// we can make the neighbors point back to node
    // which adds it back to the list
node.left.right = node;
node.right.left = node;
```

I won’t go into full detail about the implementation; again, you can refer to the Wikipedia page or the paper for more details. However, I need to clarify what I meant by a "type of doubly linked list." You could maybe call it a "double circular doubly linked list". Again, each 1 in the matrix is represented by a node. Each node, points to the nodes above and below and to the left and right and, since it is circular, of course the bottom node points to the top node and vice versa and similarly for the far left and right nodes. There are some other components, but again, I won’t go into any more detail.

## Polyomino Tiling. Again
After completing my implementation of Dancing Links, I once again tried solving some polyomino tiling problems. I would once again need a way to represent the polyominoes as well as a way to convert the tiling problem into an exact cover problem. I decided to use a different approach, both for the practice of creating and implementing different designs and because of the way the problem is now framed. Previously I would check if any polyominoes are overlapping, which involves checking whether or not a polyomino occupies a certain square. Now, constructing each row of the exact cover matrix only requires knowing which squares the polyomino does occupy. I created a simple class to represent the coordinates of a single square of a polyomino. I could have put these into a list, but since I knew each coordinate should only occur once per polyomino, I used a set instead.

The next step was to convert the polyominoes into an exact cover matrix. For rectangular puzzles, I simply used a height and width, but I also provided a way to create irregular shaped puzzles. Of course you also need the puzzle pieces as well as names for each of the pieces so that they can be given a column label on the matrix. For this I used a map of Strings (the labels) to polyominoes (the piece with that label). The remainder of the columns are all the coordinates that are part of the puzzle.

In order to convert a polyomino into a row on the matrix, I check that each coordinate of the polyomino is also a coordinate of the puzzle. If it is not, it is discarded. Otherwise, it is converted into a row by basically setting 1’s in the appropriate columns. Of course I then repeat the process for each transformation of each polyomino, which includes rotations (turning), reflections (flipping), and translations (moving).

I was glad to see that they were solved in seconds rather than hours. I didn’t perform any extensive testing but I did perform some basic sanity checks before scaling up to three dimensions. First I made sure that the solutions actually worked by going through a few manually and making sure that the pieces actually fit in the puzzle properly. I also compared some numbers from my implementation and solutions to those found in Knuth’s Dancing Links paper (where he discusses different exact cover problems, including, but not limited to, polyomino tiling). One particular problem that he focused on was the problem of using one of each pentomino in an 8x8 square with a 2x2 square removed from the middle. He claimed that the exact cover matrix should have 1568 rows, which is exactly how many I had. He reported that there were 65 unique solutions, while I found 520 solutions. However, my solutions are not unique when taking into account rotations and reflections. When taking into account that there are 8 rotations and reflections (including the original) for each solution, my 520 matches his 65.

Unfortunately I have not found a generalized way to eliminate these duplicate solutions. For the particular problem mentioned above, Knuth accomplishes this by restricting certain pieces such that the rotations and reflections are impossible. Basically he restricts a particular piece to a certain quadrant of the puzzle since the rotations and reflections would cause the piece to show up in a different quadrant.

## Solving the Soma Cube
It was now almost time to finally solve the Tetris Cube. However, I once again decided that it might be a good idea to take a smaller step towards the final problem instead. This time I tried to solve the [Soma Cube](http://en.wikipedia.org/wiki/Soma_cube). In order to do that I would need to somehow represent the [polycubes](http://en.wikipedia.org/wiki/Polycube) (which is the name for shapes made up of adjacent cubes, like the ones used in the Tetris Cube) and convert the puzzle into an exact cover matrix. For this I followed pretty much exactly the same process as for the polyomino puzzles.

According to Wikipedia, there are 240 unique solutions to the Soma Cube. However, I came up with 11,520 solutions. When taking into the 24 rotations of each solution (imagine a six-sided die, there are six sides that can face up and you can rotate it by 90 degrees, while keeping the same side up, four times; 6 * 4 = 24), that brings my number down to 480. I’m not completely sure why it is still off by a factor of 2[^rotations], but at least it’s an even multiple which probably means that I’ve done everything reasonably correctly.

## Parallel Dancing Links
After seeing the Soma Cube solved in a matter of minutes, I was quite disappointed to find that, like my first attempts at solving the polyomino puzzles, it was going to take many hours to complete the Tetris Cube. However, one major difference was that this time it would find all solutions instead of just one. Regardless, I wasn’t quite satisfied with the performance and decided to try to parallelize the algorithm.

My initial idea was fairly simple. During the first pass through the matrix, instead of going down the column one row at a time, you could do several rows in parallel, since the operations are all independent of each other. Sort of. What I mean is that completing one row does not rely on the previous one being completed. However, if you are deleting and undeleting rows and columns in the same Dancing Links structure in parallel, you will quickly run into problems.

The obvious solution would be to make a copy of the Dancing Links. However, before moving onto that solution I did a quick search to see if anyone else has tackled a parallel version of the algorithm and perhaps came up with a better solution. I came across two papers ([here](https://janmagnet.files.wordpress.com/2008/07/decs-draft.pdf) and [here](http://www.csc.kth.se/~smakadir/parallel-exact-cover-solver.pdf)) which describe similar algorithms which are a slightly more general form of the algorithm I proposed. Instead of simply parallelizing the first pass, you would search up to a certain depth and compute a number of partial solutions. You would then use the partial solutions as a starting point for the parallel search.

Unfortunately neither paper described any way to conveniently manipulate the data structure so I resorted to making a copy for each parallel search. I could not think of an easy way to copy the data structure so I resorted to having it store the exact cover matrix that it was constructed from and simply recreate the Dancing Links from scratch whenever necessary. I would then delete the rows and columns that were covered by the particular partial solution.

## Solving the Tetris Cube… Finally
I used my laptop which has a 2.2GHz i7 processor to run the program. It goes up to 2.8GHz with Intel Turbo Boost enabled but I decided to keep it at 2.2GHz so that my laptop didn’t overheat while solving the puzzle with 8 threads (4 cores plus hyper-threading) at full power. After about an hour I found 236136 solutions, which means that, if I've done everything correctly, there are probably 9839 unique solutions. This time it divides evenly by 24, but not by 48 (as with the Soma Cube[^rotations]).


## Reflection
I went into this thinking that I would spend a day or two writing a simple backtracking solution and be done with it. I was quite disappointed to find that it would not be so simple. However, it was an interesting journey that I actually enjoyed quite a lot.

While this does show the importance of a good algorithm, I think it is also important to keep in mind that it isn’t necessarily a good idea to jump straight into fastest algorithm. It took me much longer to understand and implement Algorithm X with the Dancing Links structure than it did for me to create the naive backtracking algorithm. If it turned out that the simpler algorithm were good enough for my purposes but I went with the faster but more complicated algorithm first, I might have wasted time researching, understanding, and implementing an unnecessary algorithm. On the other hand, by starting with the simple algorithm I spent a relatively short amount of time on an unnecessary algorithm, but I gained a better understanding of its limitations in the process.

If you are interested, you can find the code on [GitHub](https://github.com/ttaomae/Exact-Cover-Solver). It includes the ability to create and solve your own exact cover problems, to create and solve polyomino or polycube puzzles, or to solve the Tetris Cube or Soma Cube.

[^rotations]: Since the Tetris Cube matches my expectations about the rotations, it indicates that maybe it is a property of the Soma Cube that gives it extra symmetrical solutions. Looking at the pieces of the Soma Cube, I suspect that it may be the fact that it has two [chiral](http://en.wikipedia.org/wiki/Chirality_%28mathematics%29) pieces, which means that they are basically mirror images of each other.
