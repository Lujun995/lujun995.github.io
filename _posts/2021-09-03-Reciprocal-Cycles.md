---
layout: post
title: Reciprocal Cycles
subtitle: by Lucas Zhang
tags: [projecteuler]
---

<p>A unit fraction contains 1 in the numerator. The decimal representation of the unit fractions with denominators 2 to 10 are given:</p>
<blockquote>
<table><tr><td><sup>1</sup>/<sub>2</sub></td><td>= </td><td>0.5</td>
</tr><tr><td><sup>1</sup>/<sub>3</sub></td><td>= </td><td>0.(3)</td>
</tr><tr><td><sup>1</sup>/<sub>4</sub></td><td>= </td><td>0.25</td>
</tr><tr><td><sup>1</sup>/<sub>5</sub></td><td>= </td><td>0.2</td>
</tr><tr><td><sup>1</sup>/<sub>6</sub></td><td>= </td><td>0.1(6)</td>
</tr><tr><td><sup>1</sup>/<sub>7</sub></td><td>= </td><td>0.(142857)</td>
</tr><tr><td><sup>1</sup>/<sub>8</sub></td><td>= </td><td>0.125</td>
</tr><tr><td><sup>1</sup>/<sub>9</sub></td><td>= </td><td>0.(1)</td>
</tr><tr><td><sup>1</sup>/<sub>10</sub></td><td>= </td><td>0.1</td>
</tr></table></blockquote>
<p>Where 0.1(6) means 0.166666..., and has a 1-digit recurring cycle. It can be seen that $$\frac {1}{7}$$ has a 6-digit recurring cycle.</p>
<p>Find the value of $$d < 1000$$ for which $$\frac {1}{d}$$ contains the longest recurring cycle in its decimal fraction part.</p>

**Solution**:
<p>The length of the reciporal cycles equals to the length of remainder cycles. Namely, the number of steps that remainders take to get back to a remainder that previously occurs. Taking $$\frac {1}{7}$$ as an example, its remainders are </p>
<center>$$10 \div 7 = 1 \cdots 3$$, <\center>
<center>$$30 \div 7 = 4 \cdots 2$$..., <\center>
<p>and the remainders go as $$ 3 \Rightarrow 2 \Rightarrow 6 \Rightarrow 4 \Rightarrow 5 \Rightarrow 1 \Rightarrow 3 ...$$. Therefore, we can count the length of remainder cycles to calculate the length of the reciporal cycles.</p>
<p>In addition, By the definition of a remainder $$r$$, $$r$$ is always less than the dividor $$d$$. Therefore, $$d$$ can only have $$d$$ different remainders, <i>i.e.</i> $$0, 1, ..., d - 1$$. Accoringly, by <a href="https://en.wikipedia.org/wiki/Pigeonhole_principle">the pigeonhole principle</a>, when the length $$n$$ of the reciprocal cycles of $$d$$ would be shorter than $$d$$. Ohterwise, for any $$n^\prime$$ > $$d$$, we can always identify a digit equal to a previous remainder so the remainders enter a cycle.</p>

<p><b>Source code in Python</b>:<\p>
~~~
def solve_reciprocal_cycles():
    """"This is a wrapper function designed to solve Problem 26 (Reciprocal cycles) 
    in the ProjectEuler.
    Return:
        d_n_max: return the number of the divisor (less than 1000) that have the longest
        cycle in its reciprocal.
    """
    n_max = 0 #the largest reciporal cycles
    d_n_max = 0 #the number that has the largest reciporal cycles
    for d in range(999, 0, -1) : #d is a divior ranging 999 - 1
        if d<=n_max : break 
            #the cycle length of d is always smaller than d;
            #therefore, if i<=n_max, cycle length < n_max
        resids = [0]*(d+1)
            #initialize a list for the remaiders of d.
            #there can not be more than (d+1) different remainders.
        r = 1 #set the first remainder to 1
        t = 0 #index for current position for the remainder array
        while (r not in resids[0:max(t-2,0)]) or (t <= 1):
            if r == 0: break
            while r < d: r = r*10 #enlarge the remainder until becoming greater than d
            r = r % d # calculate the residual
            resids[t] = r 
            t += 1
        t0 = resids.index(r)
        n = (t - 1) - t0 #length of Reciprocal cycles for i
        if n > n_max: 
            n_max = n
            d_n_max = d
    return d_n_max
solve_reciprocal_cycles() #983
~~~
