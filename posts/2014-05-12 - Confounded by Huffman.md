This week, I am finishing up my "coursework" in Stanford's CS106B course. It covers programming abstractions in C++.

I have two more assignments, plus I'll probably tackle some of the section homeworks for additional review.

I am currently working on an Huffman encoding program. Even though I've done this already for my Scala course, writing it again through the imperative paradigm has proven challenging.

The decoding process leaves me stuck. This was how I originally wrote the code.

![1](https://lh6.googleusercontent.com/-JLIumm6qENY/U3Egu8sVwsI/AAAAAAAAAu0/SO1hMZ4Dyxs/326481ffb54777d51d2a5cf9e670a0ec-2014-05-12-20-27.png)

This approach worked fine for text-only files, but crashed upon decoding a compressed image file. I found this alternative formulation through a repo. (Which was infuriatingly more elegant than my approach.)

![2](https://lh5.googleusercontent.com/-IGnwtEhuQ5M/U3Egvq6pLrI/AAAAAAAAAu8/dyffZ3dvZdI/15a61cdf018cfe5158c29ad8828c0c8e-2014-05-12-20-27.png)

This version works fine with image fines. Comparing the two, I'm at a loss for why theirs works and mine does not.

Any thoughts?