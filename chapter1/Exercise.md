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

![exercise 1.2](pic\exer1.2.png)

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

