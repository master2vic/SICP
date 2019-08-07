# Exercise 1.1
### Q : Below is a sequence of expressions. What is the result printed by the interpreter in response to each expression? Assume that the sequence is to be evaluated in the order in which it is presented.

~~~scheme
10
(+ 5 3 4)
(- 9 1)
(/ 6 2)
(+ (* 2 4) (- 4 6))
(define a 3)
(define b (+ a 1))
(+ a b (* a b))
(= a b)
(if (and (> b a) (< b (* a b)))
    b
    a)
(cond ((= a 4) 6)
      ((= b 4) (+ 6 7 a))
      (else 25))
(+ 2 (if (> b a) b a))
(* (cond ((> a b) a)
         ((< a b) b)
         (else -1))
   (+ a 1))
~~~

结果:
~~~scheme
10
12
8
3
6


19
#f
4
16
6
16
~~~
其中空行是因为define式没有返回值, #f的值就是false

# Exercise 1.2
### Q : Translate the following expression into prefix form:
$$
\frac{5+4+(2-(3-(6+\frac45)))}{3*(6-2)*(2-7)}
$$
~~~scheme
(/ (+ 5 4 
    (- 2 
        (- 3 
            (+ 6 
                (/ 4 5)))))
    (* 3
        (- 6 2)
        (- 2 7)))
~~~
这道题看上去简单,但是如果书写不规范的话很容易不小心搞错.所以还是建议写的时候遵守规范. 同时如果写的时候脑子中构建一个树, 自顶向下的构建的话, 会比较简单.

# Exercise 1.3

### Q : Define a procedure that takes three numbers as arguments and returns the sum of the squares of the two larger numbers.

暂时没有想到好的答案, 先这么做
~~~ scheme
(define (sum-of-larger x y z)
    (cond ((and (< x y) (< x z)) (+ y z)) 
        ((and (< y x) (< y z)) (+ x z))
        (else (+ x y))))
~~~

# Exercise 1.4
### Q : Observe that our model of evaluation allows for combinations whose operators are compound expressions. Use this observation to describe the behavior of the following procedure:
~~~ scheme
(define (a-plus-abs-b a b)
    ((if (> b 0) + -) a b))
~~~

这道题就就是 如果 b>0 则 a+b,否则 a-b.也就是a加b的绝对值.

其实这个函数的命名就已经出卖了它😀

# Exercise 1.5
### Q : Ben Bitdiddle has invented a test to determine whether the interpreter he is faced with is using applicative-order evaluation or normal-order evaluation. He defines the following two procedures:
~~~scheme
(define (p) (p))

(define (test x y) 
  (if (= x 0) 
      0 
      y))
~~~ 
### Then he evaluates the expression:
~~~scheme
(test 0 (p))
~~~
### What behavior will Ben observe with an interpreter that uses applicative-order evaluation? What behavior will he observe with an interpreter that uses normal-order evaluation? Explain your answer. (Assume that the evaluation rule for the special form if is the same whether the interpreter is using normal or applicative order: The predicate expression is evaluated first, and the result determines whether to evaluate the consequent or the alternative expression.)

### answer:
正则序：
~~~scheme 
(test 0 (p))

=>

(if (= 0 0 )
    0
    (p))

=>

p

~~~

应用序；

~~~scheme
(test 0 (p))

=>

(test 0 (p))

=>

...

=>

(test 0 (p))
~~~

看到区别了吗？因为在定义 (p)的时候是一个死递归，所以根本不会跳出循环。
在应用序的解析过程中，需要先求解参数，所以这就是为什么会陷入循环。但是在正则序的的解析中会“先展开后规约”所以没有出发到那个死循环的时候就得到结果了。

关于正则序和应用序在后面会有更进一步的了解。但是要注意到的就是现在的Lisp解释器大都采用应用序的方式，这是为了减少计算次数。

# Exercise 1.6
### Alyssa P. Hacker doesn’t see why if needs to be provided as a special form. “Why can’t I just define it as an ordinary procedure in terms of cond?” she asks. Alyssa’s friend Eva Lu Ator claims this can indeed be done, and she defines a new version of if:
~~~scheme
(define (new-if predicate 
                then-clause 
                else-clause)
    (cond (predicate then-clause)
        (else else-clause)))
~~~
### Eva demonstrates the program for Alyssa:
~~~scheme
(new-if (= 2 3) 0 5)
5

(new-if (= 1 1) 0 5)
0
~~~
### Delighted, Alyssa uses new-if to rewrite the square-root program:
~~~scheme
(define (sqrt-iter guess x)
    (new-if (good-enough? guess x)
        guess
        (sqrt-iter (improve guess x) x)))
~~~
What happens when Alyssa attempts to use this to compute square roots? Explain.

### answer：

这道题一开始感觉没有任何问题，觉得不都是一样的吗？但是越是感觉莫名其妙越是觉得应该有没有考虑到的。

所以稳妥起见，先把代码以4的情况展开。(注意这里要用应用序，因为1.5就讨论过，lisp用的是应用序)

~~~scheme
(sqrt-iter 1 4)

(new-if (good-enough? 1 4)
    guess
    (sqrt-iter (improve 1 4) 4))

~~~

其实展开到这里就会发现问题，这里的'new-if'也是一个过程。又因为Lisp语言解析器要对所有的参数都要先进行算值然后再进行下一步展开。也就是肯定要进入下一个'sqrt-iter'函数。而正常情况下'sqrt-iter'的执行有一个先决条件:'good-enough?'过程返回的是'false'。而这个条件再计算参数的时候是不直接经历的，所以就会程序就会死在这里。

![sqrt-iter tree.png](pic\sqrt-iter6r.png)

# Exercise 1.7
### The good-enough? test used in computing square roots will not be very effective for finding the square roots of very small numbers. Also, in real computers, arithmetic operations are almost always performed with limited precision. This makes our test inadequate for very large numbers. Explain these statements, with examples showing how the test fails for small and large numbers. An alternative strategy for implementing good-enough? is to watch how guess changes from one iteration to the next and to stop when the change is a very small fraction of the guess. Design a square-root procedure that uses this kind of end test. Does this work better for small and large numbers?

这个问题其实就是在问：good-enough?去判断很大的数和很小的数合不合适。

两点原因会导致判断不准确：
1. 计算机有精度问题。
2. good-enough? 本身就是判断两次迭代中的变化量，但是对于很小的数就会不准却。

如： 'good-enough?'的精度是0.00001的时候。

1. 很小的数：当测试的数据是‘0.0000001’那么第一次迭代之后‘guess’的值就为0.000005。 其平方与数据的差值 < 0.00001(设定标准) 则会在第一次的时候就会出去。然而真实值大致是0.000316左右。那么此时的误差Δ=0.000311。这个误差相对于这个数本身显得十分大。
2. 很大的数：有可能会使得递归栈空间过大而溢出。

所以用另一种方式来判断：两次修正后的结果之差Δ < 一个较小值则认为这是一个较为近似的解。

~~~scheme
(define (good-enough? guess x)
    (< (abs (- guess (improve guess x)))
        0.00001))
~~~

但是做减法也有问题。那就是对于较小数的优化还是有问题。所以用比例的方式解决。

~~~scheme
(define (good-enough? guess x)
    (< (abs (-1+ (/ guess (improve guess x))
        0.0001))))
~~~

这个函数就是在两次改变率小于 0.0001的时候就采纳这个值作为猜测值。

# Exercise 1.8
### Newton’s method for cube roots is based on the fact that if y is an approximation to the cube root of x, then a better approximation is given by the value

$$
\frac{x/y^2+2y}{3}
$$

### Use this formula to implement a cube-root procedure analogous to the square-root procedure. (In 1.3.4 we will see how to implement Newton’s method in general as an abstraction of these square-root and cube-root procedures.)

在给定一个y的平方根的近似值之后可以采用上面的函数来寻找一个跟逼近的值。就是相当于改写improve函数。

~~~scheme
(define (improve guess x)
    (/ (+ (/ guess (square x))
            (* 2 x))
        3))
~~~
