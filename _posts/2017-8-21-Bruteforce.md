---
layout: post
title: Why not brute force everything?
---

The crazy increase of computing power we've seen during the last decades kind of makes
anything seem possible. In fact, why even bother studying algorithms anymore? Couldn't we
just build faster computers instead of optimizing the code?

[Moore's law](https://en.wikipedia.org/wiki/Moore%27s_law),
that has held for a long time, is a hand-wavy promise that our computing power doubles every other year.
While that's an amazing rate, the complexity of some problems is way greater than exponential.
For instance, when Deep Blue defeated Kasparov in chess, it was conjectured that Go champions would stay
ahead of computers for a long time, simply due to the sheer number of possible moves.
That's why AlphaGo beating the Go champion was such a big deal: We got beyond the limits of
the previously known algorithms with machine learning.

Ok then, if our computing power is limited, why not come up with smart algorithms?
Well, sometimes that's not possible at all and sometimes it's simply really hard.
Let me give you an example:
Consider a $8 \times 8$-grid.
<center><img src="{{site.url}}/images/board.png" alt="Drawing" style="width: 350px;"/></center>

Put a $2 \times 1$-block on the grid:

<center><img src="{{site.url}}/images/board2.png" alt="Drawing" style="width: 350px;"/></center>

It's trivial to cover the entire grid with these blocks, right?

<center><img src="{{site.url}}/images/board3.png" alt="Drawing" style="width: 350px;"/></center>

What if we modify the grid so that the squares $(1,1)$ and $(8,8)$ are excluded? 

<center><img src="{{site.url}}/images/board4.png" alt="Drawing" style="width: 350px;"/></center>

Is it possible to cover the modified grid with $2 \times 1$-blocks? The blocks
must stay inside the grid, but you can change their orientation if you like:

<center><img src="{{site.url}}/images/board_ex.png" alt="Drawing" style="width: 350px;"/></center>

<script>
$(document).ready(function(){
    $("#show_solution").click(function(){
    if(confirm("Are you sure you want to see the solution?")){
        $("#board_solution").slideDown("slow");}
    });
});
</script>

As you increase the size of the grid, this problem quickly gets infeasible to brute force (simply trying
every possible combination). However, there is a really simple constant time algorithm for this
(display the solution by clicking
**<a id="show_solution">here</a>**),
but it's **hard** to come up with.

{::options parse_block_html="true" /}
<div id="board_solution" style="display:none;background-color:rgb(246, 242, 255);padding-left:1cm;padding-top:0.5cm;padding-bottom:1cm;padding-right:1cm">

## The solution

It's **not** possible to cover the modified grid using $2 \times 1$-blocks.
To prove this, color the grid like a chessboard:

<center><img src="{{site.url}}/images/board5.png" alt="Drawing" style="width: 350px;"/></center>

Notice that a $2 \times 1$ block always covers two adjacent squares: One white and one black square.

<center><img src="{{site.url}}/images/board6.png" alt="Drawing" style="width: 350px;"/></center>

Consequentially, for it to be possible to cover the grid using $2 \times 1$-blocks, there would have to be equally
many white and black squares, but the modified grid has $32$ black squares and only $30$ white squares.
</div>

## Combinatorial explosion

A typical problem in elementary combinatorics deals with computing the number of possible
ways of doing something. Often some number of independent choices are made and the
numbers are combined via multiplication, for example, if there are 3 shirts and 3
pants, you have $3 \cdot 3 = 9$ outfit combinations. Because the total number of combinations
often behaves in this multiplicative way,
the number of operations easily blows up in brute force algorithms: In a nested loop
{% highlight python %}
for i in range(n):
    for j in range(m):     
        # Some expensive operations.
{% endhighlight %}
the expensive operations are ran $n \cdot m$ times.

Combinatorial explosion refers to a situation where the combinatorics of the problem
behave in a way that quickly makes it intractable, for example, when the input size increases.
Let's say, for example, that we take a string of length $n$ and iterate over its all
permutations. That's $n!$ rounds, which can easily be seen to be *way* more than $2^n$ for large $n$:

<center>$$\frac{n!}{2^n} = \frac{n}{2} \underbrace{\left( \frac{n-1}{2}\frac{n-2}{2} \cdots \frac{3}{2}\frac{2}{2}\right)}_{\geq 1}  \frac{1}{2} \geq \frac{n}{4} \to \infty,$$</center>

as $n \to \infty$. We wouldn't practically be getting anywhere by waiting, even if Moore's law held to the infinity.

That's why smart algorithms are important. On the other hand, they can be really hard to come up with.
Maybe, as in the case of Go, there are ways to use machine learning to deal with this?
