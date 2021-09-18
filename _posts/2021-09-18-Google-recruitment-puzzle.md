---
layout: post
title: Google Recruitment Puzzle
subtitle: by Lucas Zhang
cover-img: /assets/img/Google-puzzle.png
thumbnail-img: /assets/img/Google-puzzle.png
share-img: /assets/img/Google-puzzle.png
tags: [Google recruitment puzzle]
---

Find the first 10-digit prime in the decimal expansion of 17π. 

The first 5 digits in the decimal expansion of π are 14159. The first 4-digit prime in the decimal expansion of π are 4159. You are asked to find the first 10-digit prime in the decimal expansion of 17π. First solve sub-problems (divide and conquer):

- Write a function to generate an arbitrary large expansion of a mathematical expression like π. Hint: You can use the standard library `decimal` or the 3rd party library `sympy` to do this
- Write a function to check if a number is prime. Hint: See Sieve of Eratosthenes
- Write a function to generate sliding windows of a specified width from a long iterable (e.g. a string representation of a number)

Write unit tests for each of these three functions. You are encouraged, but not required, to try [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development).

Now use these helper functions to write the function that you need.
Write a unit test for this final function, given that the first 10-digit prime in the expansion e is 7427466391. Finally, solve the given problem.

**Solution**:

According to the [prime number theorem](https://en.wikipedia.org/wiki/Prime_number_theorem), the proportion of prime numbers in all 10-digit numbers are approximately 4.5%. This is because:

$$\frac {1} {\ln(10^9)} \approx \frac {1} {\ln(10^(10))} \approx 4.5\%$$

As the decimal expansion of π comprises random numbers, we would expect a probability of observing at least one prime number in the first 100 digits of the decimal part of π of $$Pr = 1- (1 - 4.5\%)^100 \approx 99\% $$. Therefore, the first prime number would likely occur in the first 100 digits of the decimal part of π.

As inidcated above, we firstly need to write a function to generate π with a high precision. This goal can be achieved with the library `sympy`:

{% highlight python %}
#! python3 -m pip install --quiet sympy
def value_generator(exp, digits):
    '''This function is used to generate n-digits pi or Euler's number e by evaluating
    argument "exp".
        PARAMETER:
            exp: string, a math expression to be evaluated
            digits: integer, how many digits of pi that you wanted.
        RETURN:
            A pi number of class decimal.
    '''
    import sympy as sp
    value = sp.N(exp, digits)
    return(value)
{% endhighlight %}

With `sympy`, we can evaluate the value of a mathmatical expression to any digit we want:

{% highlight python %}
def unit_tests_1():
    '''This function is to test if other functions go well
    
        RETURN:
            If all functions go in correct direction, return 0.
    '''
    test1 = str(value_generator("pi", 1+2)) #"3.14", 1+2 means 1 digit for integer part and 2 digits for the decimal part.
    test2 = str(value_generator("E", 1+2)) #"2.72"
    test3 = str(value_generator("17 * pi", 2+2)) #"53.41"
    assert test1 == "3.14"
    assert test2 == "2.72"
    assert test3 == "53.41"
    return 0
unit_tests_1() #0, no problem
{% endhighlight %}

Thereafter, we will need some functions to examine if a number is prime or not. According to the definition of prime numbers, a prime number greater than 1 needs to be co-prime with all the prime numbers less than its square root. This is because if $$x= a \times b$$, either $$a$$ or $$b$$ would be smaller than $$\sqrt(x)$$. If $$x$$ can not be divided by a number $$a <= \sqrt(x)$$, it would neither be divided by $$b > \sqrt(x)$$. Thus, I write one function to generate the list of these prime numbers less than a given limit and one to check if the current number is prime.

{% highlight python %}
def prime_list(limit):
    '''This function returns a list of primes smaller than a given limit
        PARAMETER:
            number: an integer required to be checked if prime
        RETURN:
            prime_table: a list of prime numbers
    '''
    prime_table = [2,3,5,7] #a list to store the prime numbers smaller than the limit
    for i in range(3,-1,-1):
        if prime_table[i] > limit: 
            prime_table.pop()
        else:
            break
    if limit <=10: 
        return prime_table
    #start from here, the limit should >10
    for i in range(11, limit+1):
        is_composite = False
        for primes in prime_table: # note that the prime_table may be appended after each i loop
            if i % primes == 0: #i.e. i is composite
                is_composite = True
                break #break the "primes" loop
        if not is_composite: prime_table.append(i) #if i is prime, add it to the prime_table
        pass #the next "i" loop
    return prime_table

def prime_checker(number, prime_table = None):
    '''This function is to check if a given number is coprime with number in the prime_table. If 
    prime_table is not given, it returns whether the number is prime or nor.
        PARAMETER:
            number: an integer required to be checked if prime
            prime_table: a list of primes given to check if the number is co-prime with them.
            Using this argument with caution: all the element should be prime numbers less
            than "number"
        RETURN:
            flag: boolean, if the number is prime, return TRUE, otherwise FALSE
    '''
    import math
    
    #exception handling
    if not isinstance(number, int):
        coprime_flag = False
        return coprime_flag
    if number <= 1:
        coprime_flag = False
        return coprime_flag
    if number == 2: 
        #exclude 2, because ceil(sqrt(number)) will equal to itself 
        coprime_flag = True
        return coprime_flag
    
    if prime_table is None:
        limit = math.ceil(math.sqrt(number)) # we just need to check if number can be divided by 
                                            #the prime numbers less than its square root
        prime_table = prime_list(limit)
    
    coprime_flag = True
    
    for primes in prime_table:
        if number % primes == 0: # that is "i is composite"
            coprime_flag = False
            break #break the "primes" loop
    
    return coprime_flag
    #the end of coprime_checker
{% endhighlight %}

The results of the two functions can be like:

{% highlight python %}
def unit_tests_2():
    '''This function is to test if other functions go well
    
        RETURN:
            If all functions go in correct direction, return 0.
    '''
    test4 = prime_list(5)
    test5 = prime_list(10)
    test6 = prime_list(11)
    test7 = len(prime_list(100))
    assert test4 == [2,3,5]
    assert test5 == [2,3,5,7]
    assert test6 == [2,3,5,7,11]
    assert test7 == 25 #should be 25 (means 25 primes less than 100)
    pass
    
    test8 = prime_checker(2) #to check if work well for small numbers
    test8_1 = prime_checker(1)
    test9 = prime_checker(3)
    test10 = prime_checker(10)
    test11 = prime_checker(14159) #to check if work well for large numbers
    test12 = prime_checker(14159, prime_table=prime_list(1000)) #when provided a prime list
    test13 = prime_checker(14160, prime_table=prime_list(1000))
    assert test8 == True
    assert test8_1 == False
    assert test9 == True
    assert test10 == False
    assert test11 == True
    assert test12 == True
    assert test13 == False
    pass
    
    return 0
unit_tests_2() #0, no problem
{% endhighlight %}

With the help of these functions, we finally need to write a wrapper function that integrates these parts and create a sliding windows along the decimal expansion of π.

{% highlight python %}
def find_first_prime(exp, width, digit_limit):
    '''This function is a wrapper function which can find the first prime in the decimal 
    expansion of a math expression "exp"
        PARAMETERs:
            exp: string, a math expression to be evaluated, e.g. "17 * pi"
            width: intger, the digits of prime that we are look for, e.g. 10
            digit_limit: intger, the function will be stopped if reaching a digit limit of 
                digit_limit, e.g., digit_limit = 100 means the funtion will stopped 
                at the 100th decimal digit if a prime number have not been found.
        RETURN:
            number: the first prime number in the decimal expansion of "exp". If None, meaning
                no prime number has been found before reaching "digit_limit"
            
    '''
    import sympy
    import math
    value = value_generator(exp, digit_limit + width + 10)
        #plus 10 to take the integer part into account and avoid round errors
    dec_part = str(value % 1)
    limit = math.ceil(math.sqrt(10 ** width)) 
    prime_table = prime_list(limit) #to get all the prime numbers less than sqrt(10^width)
    for i in range(2, digit_limit+2): 
        #start from the 3rd position of the value_dec string, right after "0."
        #while Cliburn asks to create a function to generate this sliding windows,
        #I feel it makes more sense to integrate it in the main function 
        #because it will make the program simpler and more logical.
        number = int(dec_part[i:(i+width)])
        flag = prime_checker(number = number, prime_table = prime_table)
        if flag: break
    if(flag): 
        return(number)
    else: 
        return(None)
{% endhighlight %}

Tested this function with some known examples:

{% highlight python %}
#Here comes the unit tests
def unit_tests_3():
    '''This function is to test if other functions go well
    
        RETURN:
            If all functions go in correct direction, return 0.
    '''
    
    test14 = find_first_prime(exp = "E", width = 1, digit_limit = 5) 
        #check if the first digit is included
    test15 = find_first_prime("pi", 1, 3) 
        #check if stops at a desired digit_limit
    test15_1 = find_first_prime("pi", 1, 4)
    test16 = find_first_prime("pi", 5, 10) #check the given examples
    test17 = find_first_prime("pi", 4, 10)
    test18 = find_first_prime("E", 10, 100)
    
    assert test14 == 7
    assert test15 == None
    assert test15_1 == 5
    assert test16 == 14159
    assert test17 == 4159
    assert test18 == 7427466391
    pass
    
    return 0

unit_tests_3() #0, no problem
{% endhighlight %}

And finally solve the first prime in the deciaml expansion of 17π:

{% highlight python %}
#Here comes the final result
print("The first prime number is", find_first_prime(exp = "17 * pi", width = 10, digit_limit = 100) )
    #with digit_limit = 100, we have >98% probability to get a prime number
    #The first prime number is 8649375157
{% endhighlight %}

**Reference**: 

Prime number theorem. https://en.wikipedia.org/wiki/Prime_number_theorem.

