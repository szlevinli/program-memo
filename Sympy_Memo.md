# Sympy Memo

## Basic Operations

### Substitution

Example:

```python
expr = cos(x) + 1
expr.subs(x, y)
# out put
# cos(y) + 1
```

Substitution is usually done for on of two reasons:

1. Evaluating an expression at a point.
2. Replacing a subexpression with another subexpression.

### Converting Strings to SymPy Expression

The `sympify` function can be used to convert strings into SymPy expressions.

Example:

```python
str_expr = "x**2 + 3*x - 1/2"
expr = sympify(str_expr)
expr
# out put
# x**2 + 3*2 - 1/2
```

### `evalf`

To evaluate a numerical expression into a floating point number, use `evalf`

```python
expr = sqrt(8)
expr.evalf()
# out put
# 2.82842712474619
```

By default, 15 digits of percision are use, but you can pass any number as the argument to `evalf`.

```python
pi.evalf(100)
# out put
# 3.141592653589793238462643383279502884197169399375105820974944592307816406286208998628034825342117068
```

### `lambdify`

使用其他的数字计算包计算, 提高效率

```python
import numpy 
a = numpy.arange(10) 
expr = sin(x)
f = lambdify(x, expr, "numpy") 
f(a) 
# [ 0.          0.84147098  0.90929743  0.14112001 -0.7568025  -0.95892427
#  -0.2794155   0.6569866   0.98935825  0.41211849]
```

To us lambdify with numerical libraries that it does not know about, pass a dictionary of `sympy_name:numerical_function` pairs.

```python
def mysin(x):
    """
    My sine. Note that this is only accurate for small x.
    """
    return x
f = lambdify(x, expr, {"sin": mysin})
f(0.1)
# 0.1
```

