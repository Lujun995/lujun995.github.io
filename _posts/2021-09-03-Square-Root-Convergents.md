---
layout: post
title: Square Root Convergents
subtitle: by Lucas Zhang
tags: [projecteuler]
---

*[Original description](https://projecteuler.net/problem=57) from the Project Euler:*

It is possible to show that the square root of two can be expressed as an infinite continued fraction.

$$\sqrt 2 =1+ \frac 1 {2+ \frac 1 {2 +\frac 1 {2+ \dots}}}$$

By expanding this for the first four iterations, we get:

$$1 + \frac 1 2 = \frac  32 = 1.5$$

$$1 + \frac 1 {2 + \frac 1 2} = \frac 7 5 = 1.4$$

$$1 + \frac 1 {2 + \frac 1 {2+\frac 1 2}} = \frac {17}{12} = 1.41666 \dots$$

$$1 + \frac 1 {2 + \frac 1 {2+\frac 1 {2+\frac 1 2}}} = \frac {41}{29} = 1.41379 \dots$$

The next three expansions are $$\frac {99}{70}$$, $$\frac {239}{169}$$, and $$\frac {577}{408}$$, but the eighth expansion, $$\frac {1393}{985}$$, is the first example where the number of digits in the numerator exceeds the number of digits in the denominator.

In the first one-thousand expansions, how many fractions contain a numerator with more digits than the denominator?

**Solution**: 

By observation of the construction of this number sequence, we can identify that a new denominator come from the sum of its prior numerator and denominator, while a new numerator comes from the sum of the prior denominator and the new denominator. For example, for the first fraction, we have $$\frac {3}{2}$$. For the second fraction, we have $$\frac {2+(3+2)}{3+2}$$.

Also because of this construction, by [the Euclidean algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm), we can show that all the pairs of numerators and denominators in this sequence are coprime, i.e. no common divisor greater than 1 exists between the numerator and the denaminator. For example, regarding the third fraction $$\frac {17}{12}$$, 

$$17 = 12+5$$

$$12 = 5 \times 2 + 2 = 5 + 7$$

$$5 = 2 \times 2 + 1 = 2+3$$

Therefore, the greatest common divisor between 17 and 12 is 1. We can also identify the numerators and denominators of the first and the second fractions in this process of Euclidean algorithm. It follows that all the numerators and denominators in fractions in the sequence have a greatest common divisor of 1 by mathematical induction. Namely, all the pairs of numerators and denominators in this sequence are coprime so we do not need to consider the reduction of the fractions. Therefore, we can use data types of double instead of a very long integer as it provides sufficient precision. 

**Source code in Python**:

{% highlight python %}
def solve_square_root_convergents():
    """This function is a wrapper function designed to solve the Square root convergents 
    problem (problem 57) in the ProjectEuler.
    
    return:
        count: the number of fractions contain a numerator with more digits than the 
        denominator in the first one-thousand expansions.
    """
    import math
    a = 2 #the first denominator
    b = 3 #the first numerator
    c = 0
    d = 0
    count = 0 #the number of fractions contain a numerator with more digits than the denominator
    for i in range(2,1001):
        c = a + b #the next denominator
        d = c + a #the next numerator
        a = c
        b = d
        if math.floor(math.log10(b)) - math.floor(math.log10(a)) == 1 :
            #if the numerator have more digits than the denominator
            count += 1
        if i == 800 : #reduce the number to avoid reaching the limits of double
            b= b/1e100
            a= a/1e100
    return count

print(solve_square_root_convergents()) #153
{% endhighlight %}

**Reference**: 

Project Euler. *Square root convergents*. https://projecteuler.net/problem=57.
