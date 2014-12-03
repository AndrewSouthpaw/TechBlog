I remember the first time I brushed up against the concept of recursion in a summer camp programming class in Visual Basic 6.0. (Yep, those were the good old days.)

Most programmers speak of recursion with awe in their voice. The words "elegant," "beautiful," and "artistic in its simplicity" often come up. My initial exposure was seeing a recursive program to draw a Sierpinski triangle.

![Sierpinski Triangle](http://eldar.mathstat.uoguelph.ca/dashlock/Outreach/Gallery/Siepinski1D960.gif)
The Sierpinski triangle.

After some puzzling over the code, I eventually understood how it worked, but never fully grokked the elegance. It wasn't until now, properly learning recursion through CS106B, that it hit me while learning about the Tower of Brahma (Tower of Hanoi) puzzle.

Fun side note: it's cool to read about the legend behind the [Tower of Brahma puzzle](http://en.wikipedia.org/wiki/Tower_of_Hanoi). From Wikipedia:

> The puzzle was first publicized in the West by the French mathematician Édouard Lucas in 1883. There is a history about an Indian temple in Kashi Vishwanath which contains a large room with three time-worn posts in it surrounded by 64 golden disks. Brahmin priests, acting out the command of an ancient prophecy, have been moving these disks, in accordance with the immutable rules of the Brahma, since that time. The puzzle is therefore also known as the Tower of Brahma puzzle. According to the legend, when the last move of the puzzle will be completed, the world will end. It is not clear whether Lucas invented this legend or was inspired by it.

> If the legend were true, and if the priests were able to move disks at a rate of one per second, using the smallest number of moves, it would take them 264−1 seconds or roughly 585 billion years or 18,446,744,073,709,551,615 turns to finish, or about 127 times the current age of the sun.

After the lecturer set up the problem, I thought for a moment how to solve it recursively. I didn't have any insight (not giving myself proper time to really work it through on my own), but I ventured to guess that the solution would be rather tricky and possibly complicated.

Then she revealed [the code](http://youtu.be/uFJhEPrbycQ?t=29m5s). 

All that work, accomplished through three simple lines. 

My reaction:

![What is this magic?!](http://24.media.tumblr.com/tumblr_lyw3zkonHG1qcxtm5o1_500.gif)

The lecturer, Julie, clearly takes pedagogy seriously, for she took the time to create illuminating animations to see the process of recursion. [Here it is](http://youtu.be/NdF1QDTRkck?t=38m4s), solving the Queens puzzle (how to fit 4 queens on a 4x4 chess board, or NxN board) using recursive backtracking. (She walks through it piecewise for a 4x4 board, then shows it automated for an 8x8 board at [42:30](http://youtu.be/NdF1QDTRkck?t=42m30s).

Fascinating. Now I am beginning to see the stunning power of recursion, where it can solve problems that are simply unsolvable any other way. 

Admittedly, I am a little nervous about actually being able to solve recursive problems. We'll see. I did have the intuition to come up with the recursive solution to the midpoint-finding Karel problem from CS106A. The other examples shown thus far I am able to figure out once the code is presented, but I'm not sure how quickly I would have reached that point on my own. 