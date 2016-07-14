---
layout: page
title: Fun With Functors - Enumerating a Product Type
---

Imagine you have a product type made of up many sum types.

```haskell
data Make = Tesla | Honda | Toyota
          deriving (Show, Eq)

data Model = ModelS | Model3 | ModelX | Odyssey | Camry
           deriving (Show, Eq)

data Color = Red | Blue | Green | Yellow | Black | Grey | White
           deriving (Show, Eq)

data Car = Car Make Model Color
         deriving (Show, Eq)
```

Each `Car` has a make, model, and color. This is cool and all, but how many total cars could we make, given this information?

Well we know `Car` is a product type, and we know `Make`, `Model`, and `Color` are sum types, so we have `3 * 5 * 7 = 105` different cars!  So we could just write them all out by hand!  Just kidding. I am lazy, and you should be too.  We can figure out a better way.

How can we generate `allCars :: [Car]` which represents every possible car?  We could start out by making Lists representing all of the values of each sum type.

```haskell
allMakes :: [Make]
allMakes = [Tesla, Honda, Toyota]

allModels :: [Model]
allModels = [ModelS, Model3, ModelX, Odyssey, Camry]

allColors :: [Color]
allColors = [Red, Blue, Green, Yellow, Black, Grey, White]
```

Now that we have the setup completed, we need to remember a few basic facts:
1. Data constructors are _functions_
2. Lists are _Functors_, and more specifically, _Applicative Functors_

This means that we can use all of that function and Functor goodness to complete our problem.

Let's look at the type signature for the data constructor `Car`.

```haskell
Car :: Make -> Model -> Color -> Car
```

Okay, so all we have to do is give a make, model, and color to the `Car` data constructor, and we get a Car! That's great! But how can we get _all possible_ cars?

Here comes our Functor!
Recall that the Functor typeclass provides the fmap function.

```haskell
fmap :: Functor f => (a -> b) -> f a -> f b
```

Believe it or not, we already have this mysterious function `(a -> b)`, it is the Car data constructor!  If we remember that in Haskell, all functions are curried, so we can do `fmap Car allMakes` to get a List of new functions that are waiting to receive a `Model` and `Color` in order to create a `Car`.

Now recall that the Applicative typeclass provides the `<*>` function.

```haskell
<*> :: Applicative f => f (a -> b) -> f a -> f b
```

`<*>` will allow us to take our List of functions, and apply it to a list of values, to get another list of curried functions!  We can repeat this for `allModels` and `allCars`.  The result is `[Car]`!!

It is important to note that `<*>` is left associative, so we can chain its application without parenthesis.  Here is the full code:

```haskell
allCars :: [Car]
allCars = (fmap Car allMakes) <*> allModels <*> allColors
```

If we remember that `<$>` is a cute left associative infix version of `fmap`, we can clean up that code even further.

```haskell
allCars = Car <$> allMakes <*> allModels <*> allColors
```

Indeed, the length of `allCars` is `105`, just like we expected!

So there you have it!  We enumerated a product type by utilizing a the Functor and Applicative Functor properties of the `List` type!  
