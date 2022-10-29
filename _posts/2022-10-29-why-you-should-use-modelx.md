---
layout: post
comments: true
title: "Why you should use modelx"
categories: modelx
---

**modelx** is a handy tool which enables you to use
Python like a spreadsheet and build object-oriented models. 

You may be wondering, "Why should I use modelx? Why not just use Python alone?"

To feel that modelx makes our life easier, let's build a simple model using modelx,
and see how the same model could be built using Python without modelx.

## A Monte-Carlo simulation

Let's say we want to build a model that performs a Monte Carlo simulation to generate 10,000 stochastic paths of a stock price that follow a geometric Brownian motion
and to price an European call option on the stock. 
For now, let's ignore the fact that a Black-Scholes formula would give the analytical solution.
Later, we will check the analytical method gives the same answer.

Here's the entire script for building the model using modelx.

```python
import modelx as mx
import numpy as np

model = mx.new_model()                  # Create a new Model named "Model1"
space = model.new_space("MonteCarlo")   # Create a UserSpace named "MonteCarlo"

# Define names in MonteCarlo
space.np = np
space.M = 10000     # Number of scenarios
space.T = 3         # Time to maturity in years
space.N = 36        # Number of time steps
space.S0 = 100      # S(0): Stock price at t=0
space.r = 0.05      # Risk Free Rate
space.sigma = 0.2   # Volatility
space.K = 110       # Option Strike


# Define Cells objects in MonteCarlo from function definitions
@mx.defcells
def std_norm_rand():
    gen = np.random.default_rng(1234)
    return gen.standard_normal(size=(N, M))


@mx.defcells
def S(i):
    """Stock price at time t_i"""
    dt = T/N
    if i == 0:
        return np.full(shape=M, fill_value=S0)
    else:
        epsilon = std_norm_rand()[i-1]
        return S(i-1) * np.exp((r - 0.5 * sigma**2) * dt + sigma * epsilon * dt**0.5)


@mx.defcells
def CallOption():
    """Call option price by Monte Carlo"""
    return np.average(np.maximum(S(N) - K, 0)) * np.exp(-r*T)
```

After executing the code above from IPython console,
running the model  is as simple as calling a function:
```python
>>> CallOption()
16.26919556999345
```

One of modelx's advantages is, all the intermediate values used to calculate the target `CallOption` are retained and available at no cost.

```python
>>> S(space.N)      # Stock price at i=N i.e. t=T
array([ 78.58406132,  59.01504804, 115.148291  , ..., 155.39335662,
        74.7907511 , 137.82730703])
```

If we want to see the option price for another strike, simply assign the new strike to ``K``. 

```python
>>> space.K = 100   # Cache is cleared by this assignment

>>> CallOption()    # New option price for the updated strike
20.96156962064
```

You can even dynamically create multiple copies of *MonteCarlo*
with different combinations of ``r`` and ``sigma``,
by parameterizing *MonteCarlo* with ``r`` and ``sigma``:

```python
>>> space.parameters = ("r", "sigma")   # Parameterize MonteCarlo with r and sigma

>>> space[0.03, 0.15].CallOption()  # Dynamically create a copy of MonteCarlo with r=3% and sigma=15%
14.812014828333284

>>> space[0.06, 0.4].CallOption()   # Dynamically create another copy with r=6% and sigma=40%
33.90481014639403
```

## Understanding the modelx approach

Now, let's look back and take a closer look at the initial script to understand more about what was going on when we built the model.

```python
import modelx as mx
import numpy as np
```

The first `import` statement starts modelx behind the scene, and defines `mx`, an alias for the modelx modules for convenience.

The second import statement should be familiar to most Python users.
It imports the numpy module as `np` into the global namespace of the `__main__` module, which is the module that we are just working in.
As we will see later, defining `np` in the global namespace of `__main__` doesn't make it available from the *Formulas*. We'll get to this point later.

By the next statement, we are creating a new *Model* object and assigning it 
to a name `model`. Since we don't give an explicit name to the `new_model` function,
the model is named *Model1* by modelx. 
A *Model* object is to modelx what a *Workbook* is to Excel. 
It is the outermost container of all objects contained in it.

Then the next statement creates a *UserSpace* object named *MonteCarlo* in the model.
A *UserSpace* is to modelx what a *Worksheet* is to Excel.
It is a container in which we are going to create *Cells* objects.

```python
model = mx.new_model()                  # Create a new Model named "Model1"
space = model.new_space("MonteCarlo")   # Create a UserSpace named "MonteCralo"
```
A *Cells* object acts like a cached function.
It can be called like a function, and the returned value is retained until
it needs to be updated.
A *Cells* object resembles a cell in Excel, but unlike Excel's cell,
its formula can have parameters, so it can retain multiple values, 
one value for one set of parameter values.

A *UserSpace* has another important role, aside from being the parent of containing *Cells*, 
which is to provide the namespace for the *Formulas* of the containing *Cells*.
In this sense, a *UserSpace* resembles a Python module. 

A *Cells* object has an associated *Formula* object. 
The *Formula* object is essentially a Python function, 
except that it is not evaluated in the Python's global namespace, which is, in our case, 
`__main__`'s namespace, but instead,
it is evaluated in the namespace provided by the parent *UserSpace*.
You can define names in the *UserSpace*'s namespace by attribute assignment operations.  

The next block of code assigns values and objects we use in our model
to names in the namespace of *MonteCarlo*. 

```python
# Define names in MonteCarlo
space.np = np
space.M = 10000     # Number of scenarios
space.T = 3         # Time to maturity in years
space.N = 36        # Number of time steps
space.S0 = 100      # S(0): Stock price at t=0
space.r = 0.05      # Risk Free Rate
space.sigma = 0.2   # Volatility
space.K = 110       # Option Strike
```

The next part constructs the main body of our model's calculation logic.
It creates 3 *Cells* objects, `std_norm_rand`, `S` and `CallOption` in *MonteCarlo*.
A *Cells* object acts like a cached function. 
It can be called like a function, and the returned value is retained until
it needs to be updated.

`defcells` is a convenience decorator for creating *Cells* objects quickly 
from function definitions.
The first `def` statement with `defcells` decorator creates a *Cells*
object named `std_norm_rand`, and assigns the object to the name `std_norm_rand`
in the global namespace of `__main__`. 
In addition, 
the statement defines the *formula* property of the Cells object from the `std_norm_rand` function definition. The *formula* property holds the *Formula* object, 
which is essentially a copy of the decorated Python function, 
but the global names in the Formula refer to the values we just assigned above.

The same goes with `S` and `CallOption`. 
Note that within the definitions of the formulas, 
we can refer to the other Cells defined in *MonteCarlo* as well as the names defined above.
Also note that we can refer to the names directly, 
without preceding object names and the dot. 

```python
# Create Cells objects in MonteCarlo and define their formulas from function definitions

@mx.defcells
def std_norm_rand():
    gen = np.random.default_rng(1234)
    return gen.standard_normal(size=(N, M))


@mx.defcells
def S(i):
    """Stock price at time t_i"""
    dt = T/N
    if i == 0:
        return np.full(shape=M, fill_value=S0)
    else:
        epsilon = std_norm_rand()[i-1]
        return S(i-1) * np.exp((r - 0.5 * sigma**2) * dt + sigma * epsilon * dt**0.5)


@mx.defcells
def CallOption():
    """Call option price by Monte Carlo"""
    return np.average(np.maximum(S(N) - K, 0)) * np.exp(-r*T)
```

## Building the model without modelx

Now let's see how a similar model can be build by Python without modelx.

```python
import numpy as np

# Define names in this module

M = 10000     # Number of scenarios
T = 3         # Time to maturity in years
N = 36        # Number of time steps
S0 = 100      # S(0): Stock price at t=0
r = 0.05      # Risk Free Rate
sigma = 0.2   # Volatility
K = 110       # Option Strike


# Define Cells objects from function definitions

def std_norm_rand():
    gen = np.random.default_rng(1234)
    return gen.standard_normal(size=(N, M))


def S(i):
    """Stock price at time t_i"""
    dt = T/N; t = dt * i
    if i == 0:
        return np.full(shape=M, fill_value=S0)
    else:
        epsilon = std_norm_rand()[i-1]
        return S(i-1) * np.exp((r - 0.5 * sigma**2) * dt + sigma * epsilon * dt**0.5)


def CallOption():
    """Call option price by Monte-Carlo"""
    return np.average(np.maximum(S(N) - K, 0)) * np.exp(-r*T)
```

The code above defines the same names and functions as the modelx example above.
If you run the code then the names and functions are defined in the global namespace 
of the `__main__` module.

We can check that calling `CallOption` give the same value as the modelx example.
However, this code is very inefficient, as `std_norm_rand` is called 36 times
and every time it's called it generates the same 10,000 random numbers.
To feel the inefficiency more clearly, we can change `N` to, say, 360 and call `CallOption`.
We will see that the call takes a lot of time.

To avoid the multiple calls, a cache mechanism can be added.
Luckily, `functools`, one of the Python standard libraries offers decorators
to implement such functionality pretty quickly.
The code below modifies the previous implementation with the cache mechanism.

```python
import numpy as np
from functools import cache

# Define names in MonteCarlo

M = 10000     # Number of scenarios
T = 3         # Time to maturity in years
N = 36        # Number of time steps
S0 = 100      # S(0): Stock price at t=0
r = 0.05      # Risk Free Rate
sigma = 0.2   # Volatility
K = 110       # Option Strike


# Define Cells objects from function definitions

@cache
def std_norm_land():
    gen = np.random.default_rng(1234)
    return gen.standard_normal(size=(N, M))


@cache
def S(i):
    """Stock price at time t_i"""
    dt = T/N; t = dt * i
    if i == 0:
        return np.full(shape=M, fill_value=S0)
    else:
        epsilon = std_norm_rand()[i-1]
        return S(i-1) * np.exp((r - 0.5 * sigma**2) * dt + sigma * epsilon * dt**0.5)


@cache
def CallOption():
    """Call option price by Monte-Carlo"""
    return np.average(np.maximum(S(N) - K, 0)) * np.exp(-r*T)
```

Now, `CallOption` doesn't take much time even if `N` is large, thanks to `cache`.
However, `cache` doesn't detect changes made to names and functions that the decorating
function is referring to.
So, when you call `CallOption` in an interactive Python session and change 
`K` or `S` won't clear the cache of `CallOption`:

To get the updated value, you need to clear the cache either by rerunning the entire script or 
explicitly calling `cache_clear` on `CallOption` and `S`. This is obviously inconvenient and error prone.

```python
>>> CallOption()    # This caches the returned value 
16.26919556999345

>>> r = 0.03    # Changing the risk free rate 

>>> CallOption()    # Returns the cached value
16.26919556999345

>>> CallOption.cache_clear()    # Clearing the cached value

>>> CallOption()    # Still wrong because S(i) returns cached values
17.275226439112906

>>> S.cache_clear()

>>> CallOption.cache_clear()    # Need to clear again

>>> CallOption()    # Finally this returns the correct value
13.576812884830224
```

Another disadvantage of this approach is that we cannot instanciate the module, so to price multiple options we need to reuse the same module by clearing the caches, assigning new value to the names and calling the same `CallOption`. This limitation can be resolved by defining 
names and methods in a class instead. 

## Using Python class instead of modelx

Another approach would be to define the names and the logic in a class, so that we can make instances of the class.

```python
import numpy as np
from functools import cache

class MonteCarlo:

    def __init__(self):
        self.M = 10000     # Number of scenarios
        self.T = 3         # Time to maturity in years
        self.N = 36        # Number of time steps
        self.S0 = 100      # S(0): Stock price at t=0
        self.r = 0.05      # Risk Free Rate
        self.sigma = 0.2   # Volatility
        self.K = 110       # Option Strike

        # Workaround to use cache on methods. 
        self.std_norm_rand = cache(self._std_norm_rand)
        self.S = cache(self._S)
        self.CallOption = cache(self._CallOption)

    def _std_norm_rand(self):
        gen = np.random.default_rng(1234)
        return gen.standard_normal(size=(self.N, self.M))

    def _S(self, i):
        """Stock price at time t_i"""
        dt = self.T / self.N
        if i == 0:
            return np.full(shape=self.M, fill_value=self.S0)
        else:
            epsilon = self.std_norm_rand()[i-1]
            return self.S(i-1) * np.exp(
                (self.r - 0.5 * self.sigma**2) * dt + self.sigma * epsilon * dt**0.5)

    def _CallOption(self):
        """Call option price by Monte-Carlo"""
        return np.average(np.maximum(self.S(self.N) - self.K, 0)) * np.exp(-self.r * self.T)
```

There are two major differences from the module approach.

First, since `cache` cannot be used on methods as [this video](https://www.youtube.com/watch?v=sVjtp6tGo0g) explains in detail, 3 methods are assigned to instance variables in `__init__` to work around the limitation. 

Second, the names defined in `__init__` are preceded by `self.` when they are referenced
in the methods. This makes the methods less concise and harder to read.

With this approach, we can create multiple instances of *MonteCarlo*:

```python
>>> mc1 = MonteCarlo()

>>> mc1.CallOption()
16.26919556999345

>>> mc2 = MonteCarlo()

>>> mc2.r = 0.03

>>> mc2.CallOption()
13.576812884830224
```

However, even with this approach, the problem of caches being not automatically updated still persists.

```python
>>> mc1.r = 0.03

>>> mc1.CallOption()    # Returns the cached value
16.26919556999345

>>> mc1.CallOption.cache_clear()

>>> mc1.CallOption()    # Still wrong because S(i) returns cached values
17.275226439112906

>>> mc1.S.cache_clear()

>>> mc1.CallOption.cache_clear()    # Need to clear again

>>> CallOption()    # Finally this returns the correct value
13.576812884830224
```

## Conclusion

We have seen how modelx helps us quickly build a model that you cannot build in Python alone without modelx. modelx allows us to write recursive formulas thanks to its sophisticated caching mechanism. modelx's UserSpace provides the namespace for contained Cells, and multiple copies of it can be created dynamically by parameterizing it with names defined in it. Other than the cache and parameterization features, modelx offers many more features to help us interactively develop, run, debug, save and document models.

One last thing we should do is to check the option prices by the Black-Scholes formula. 
Continuing from the first script, we can define and run a Black-Scholes formula as follows.
The result is pretty close to the Monte-Carlo result.

```python
>>> @mx.defcells
... def BlackScholesCall():
...     """Call option price by Black-Scholes-Merton"""
...     from scipy.stats import norm
...     e = np.exp
...     N = norm.cdf
... 
...     d1 = (np.log(S0/K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
...     d2 = d1 - sigma * np.sqrt(T)
...     return S0 * N(d1) - K * e(-r*T) * N(d2)  

>>> space.K = 110   # Changing the strike price back to the initial value

>>> BlackScholesCall()
16.210871364283975

>>> CallOption() / BlackScholesCall()   # 0.3% error
1.0035978451989926
```