# python联动Mathematica

在python中可以使用`wolframclient`包调用MMA的功能。

```python
from wolframclient.evaluation import WolframLanguageSession
from wolframclient.language import wl, wlexpr
'''
A session to a Wolfram kernel enabling evaluation of Wolfram Language expressions.
The parameter is the Wolfram kernel's path.
'''
session = WolframLanguageSession("D:\Mathematica\WolframKernel.exe")
```

使用Mathematica的表达式需要用到`wlexpr`，使用Mathematica的函数功能需要用到`wl`。

```python
a=session.evaluate(wl.N(1/5))
print(a,type(a))
# >>> 0.2 <class 'float'>

b=session.evaluate(wl.Table(i^2,{i,1,5}))
print(b,type(b))
# >>> (1, 4, 9, 16, 25) <class 'tuple'>

def J(n:int|float,x:int|float)->float:
    result = session.evaluate(wl.BesselJ(n, x))
    return result
print(J(0,0)) #>>> 1

result=session.evaluate(wlexpr("NIntegrate[x^2,{x,0,1}]"))
print(result,type(result))
# >>> 0.3333333333333338 <class 'float'>

roots=session.evaluate(wlexpr("NSolveValues[x^2+4*x+3==0,x]"))
print(roots,type(roots))
# >>>(-3.0, -1.0) <class 'tuple'>
```

一般情况下`wlexpr`输出的是`wolframclient.language.expression.WLFunction`,也就是MMA自带的函数形式，尤其当返回值是解析式，有理数而不是数值。

```python
inte=session.evaluate(wlexpr('Integrate[x^2,{x,0,1}]'))
print(inte,type(inte))
# >>> Rational[1, 3] <class 'wolframclient.language.expression.WLFunction'>

func=session.evaluate(wlexpr('Integrate[x^2,x]'))
print(func)
# >>> Times[Rational[1, 3], Power[Global`x, 3]]

root = session.evaluate(wlexpr('FindRoot[x^2+2*x+1==0,{x,0}]'))
print(root,type(root))
# >>> (Rule[Global`x, -0.9999999850988388],) <class 'tuple'>
```

我们可以使用`wl`里面的函数把他转化成python能接受的形式，比如说上面几个例子：

```python
a=session.evaluate(wl.N(inte)) #Numerical Calculation to float
print(a)

def f(x):
    return session.evaluate(wl.N(wl.ReplaceAll(func, wl.Rule(wl.Global.x,x))))
print(f(1))

result = session.evaluate(wl.ReplaceAll(wl.Global.x, root))
print(result)
```

```python
#DSolveValue
sol=session.evaluate(wlexpr("DSolveValue[{y''[x]-y[x]==0,y[0]==1,y'[0]==0},y[x],x]"))
print(sol)
#>>> Times[Rational[1, 2], Power[E, Times[-1, Global`x]], Plus[1, Power[E, Times[2, Global`x]]]]

def f(x):
    return session.evaluate(wl.N(wl.ReplaceAll(sol, wl.Rule(wl.Global.x,x))))
print(f(1))
#>>> 1.543080634815244
```

```python
#NDSolveValue
sol=session.evaluate(wlexpr("NDSolveValue[{y''[x]-y[x]==0,y[0]==1,y'[0]==0},y[x],{x,0,2}]"))
print(sol)
#long string starting with InterpolatingFunction 

def f(x):
    return session.evaluate(wl.N(wl.ReplaceAll(sol, wl.Rule(wl.Global.x,x))))
print(f(1))
#>>> 1.5430806949569018
```

似乎无法正确地输出图片，所以`matplotlib`的功能无法取代。

