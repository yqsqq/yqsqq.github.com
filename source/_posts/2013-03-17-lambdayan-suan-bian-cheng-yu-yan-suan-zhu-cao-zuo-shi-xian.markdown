---
layout: post
title: "Lambda演算-编程语言算术操作实现"
date: 2013-03-15 18:54
comments: true
categories: Lambda ruby 函数式编程
---
###<a target="_blank" href="http://zh.wikipedia.org/wiki/%CE%9B%E6%BC%94%E7%AE%97 ">Lambda演算</a>  
λ演算，λ演算是一套用于研究函数定义、函数应用和递归的形式系统。它由 Alonzo Church 和 Stephen Cole Kleene 在 20 世纪三十年代引入，Church 运用 lambda 演算在 1936 年给出 判定性问题 (Entscheidungsproblem) 的一个否定的答案。这种演算可以用来清晰地定义什么是一个可计算函数。关于两个 lambda 演算表达式是否等价的命题无法通过一个通用的算法来解决，这是不可判定性能够证明的头一个问题，甚至还在停机问题之先。Lambda 演算对函数式编程有巨大的影响，特别是Lisp 语言。


###定义常量
我们这里使用ruby的lambda函数，为了编写方便，我们给lambda定义一个别名 L；同时，为了能方便地看到输出结果，我们定义了printL函数
``` ruby
alias L lambda

def  printL n
	puts n[L{|x| x + 1}][0]
end

```

我们先来看 Church数的定义。  
0 ≡ λf.λx.x,  
1 ≡ λf.λx.f x  
2 ≡ λf.λx.f (f x)  
3 ≡ λf.λx.f (f (f x))  
 ..  
n ≡ λf.λx.fn x  
..  
由此可见Church整数是一个高阶函数，以单一参数函数f为参数，返回另一个单一参数的函数。 
据此，我们定义几个需要用到的整数，例如0-5：
``` ruby
$_0 = L{|f| L{|x| x}}
$_1 = L{|f| L{|x| f[x]}}
$_2 = L{|f| L{|x| f[f[x]]}}
$_3 = L{|f| L{|x| f[f[f[x]]]}}
$_4 = L{|f| L{|x| f[f[f[f[x]]]]}}
$_5 = L{|f| L{|x| f[f[f[f[f[x]]]]]}}

```

Church布尔值:  
TRUE = λx.λy.x  
FALSE = λx.λy.y  

条件分支:  
IFTHENELSE =  λp.λa.λb. p a b
``` ruby
$_TRUE = L{|x| L{|y| x}}
$_FALSE = L{|x| L{|y| y}}
$_IF = L{|p| L{|a| L{|b| p[a][b]}}}
```

有序对（2元组）数据类型可以用TRUE、FALSE来定义。  
CONS := λm.λn.λs (s m n)  
CAR := λp.p TRUE  
CDR := λp.p FALSE  
``` ruby

$_CONS = L{|m| L{|n| L{|s| s[m][n]}}}
$_CAR = L{|p| p[$_TRUE]}
$_CDR = L{|p| p[$_FALSE]}
```

###算术运算
我们先来看看自增的定义：  
INC = λn.λf.λx.f(n f x)  
我们可以定义自增的函数，接收一个参数n, 返回n+1。有了自增，我们就可以实现加法了  
ADD = λn.λm.m INC n  
自增和加法的代码如下:
``` ruby
$_INC = L{|n| L{|f| L{|x| f[n[f][x]]}}}
$_ADD = L{|n| L{|m| m[$_INC][n]}}

#验证结果 4 + 5
printL $_ADD[$_4][$_5]
#输出 9
```
乘法的定义：  
MULT = λm.λn.m (ADD n) 0  
乘法的代码如下:
``` ruby
$_MULT = L{|m| L{|n| m[$_ADD[n]][$_0]}}

#验证结果 4*5
printL $_MULT[$_4][$_5]
#输出 20
```
阶乘的定义:  
POW = λb.λe.e b  
阶乘的代码如下:
``` ruby
$_POW = L{|b| L{|e| e[b]}}
# 验证结果 3的四次幂
printL $_POW[$_3][$_4]
# 输出 81 
``` 

减法代码：  
``` ruby
$_WRAP = L{|f| L{|p| $_CONS [$_FALSE] [$_IF[$_CAR[p]][$_CDR[p]][f[$_SND[p]]]]}}
$_SUB1 = L{|n| L{|f| L{|x| $_CDR[n[$_WRAP[f]][$_CONS[$_TRUE][x]]]}}}
$_SUB = L{|n| L{|m| m[$_SUB1][n]}}


#验证 3自减1
printL $_SUB1[$_3]
#输出 2

#验证 5 减去 2
printL $_SUB[$_5][$_2]
#输出 3
```



