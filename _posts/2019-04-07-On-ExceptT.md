---
layout: post
title:  "Success with ExceptT"
permalink: exceptional-success
---
As the [documentation](https://pursuit.purescript.org/packages/purescript-transformers/4.2.0/docs/Control.Monad.Except.Trans#t:ExceptT) for `ExceptT` states, the `ExceptT` monad transformer is used to add exceptions to other monads.  More specifically, `ExceptT` allows the use of `throwError :: e -> m a` within a monad transformer stack, where `throwError` halts computation, yielding the error `e`.

However, the naming convention here is unfortunate, because exceptions are usually used to denote _bad_ or _unexpected_ events in a system, especially those which the system may not be able to handle.  If someone is coming from a javascript background where the `throw` statement is almost always used to indicate bad input or some other error case, then it is easy to glance over the power of `ExceptT`.

Looking at the definition of `ExceptT` shows that it allows us to embed an `Either` computation into a computation in some other monad. (Some people prefer to call this monad transformer `EitherT`)
```haskell
data Either e a = Left e | Right a

newtype ExceptT e m a = ExceptT (m (Either e a))
```

`Either` gives us the capability to short-circuit our computations where the `Left` constructor short-circuits with some value, and the `Right` constructor allows the continuation of a computation with some intermediate result.

A pattern that I've been enjoying recently is using `ExceptT` to represent success cases, not error cases.

I've used this recently for calculating the preferred alignment of tooltip components.  You can use `ExceptT` to run a computation which yields the first alignment with an on-screen tooltip.  You can then handle the case where none of the alignments yields an on-screen tooltip when you interpret the `ExceptT`.

```haskell
data Alignment = Top | Bottom | Left | Right
-- Computes the position of the base element for which we will render a tooltip
computeBasePosition :: Tooltip -> Effect { x :: Int, y :: Int }

-- Computes whether or not a tooltip will be offscreen given some base coordinates
isOffscreen :: { x :: Int, y :: Int } -> Tooltip -> Effect Boolean

-- Generates a list of alignments to attempt in preferred order
alignmentsToAttempt :: Tooltip -> [Alignment]

-- Computes the preferred alignment for a tooltip
getPrefferedAlignment :: Tooltip -> Effect Alignment
getPrefferedAlignment tooltip = do
  coords <- computeBasePosition tooltip
  result <- runExceptT $
    for_ (alignmentsToAttempt tooltip) \alignment -> do
      offscreen <- liftEffect $ isOffscreen coords tooltip
      when (not offscreen) do
        -- short-circuit further computations, as we have already found the alignment we want
        throwError alignment
  pure $ case result of
    -- If a computation succeeded, then we short-circuited with the preferred on-screen alignment
    Left alignment -> alignment
    -- If none of the computations succeeded, then we default to the preffered alignment for the tooltip
    Right _ -> tooltip.alignment
```

The underlying `ExceptT` type used in the example above is `ExceptT Alignment Effect Unit`.  Using `Unit` as the result type (where the computation did _not_ succeed) allows for clean use of [`for_`](https://pursuit.purescript.org/packages/purescript-foldable-traversable/4.1.1/docs/Data.Foldable#v:for_) and [`when`](https://pursuit.purescript.org/packages/purescript-prelude/4.1.0/docs/Control.Applicative#v:when), which looks a lot like a for loop with a break or return statement!

Other applications of the pattern are for turn based games, where each turn may result in a winner of the game.  Consider a type `StateT GameState Effect Unit` which respresents the state we need to thread in between distinct turns.  You could have each turn check the state and do nothing if the win condition has been met, _or_ you can add `ExceptT` to make `ExceptT Winner (StateT GameState Effect) Unit`. You can then use `throwError` to short-circuit when a win condition is satisfied.  A turn can then look like this:
```haskell
turn :: ExceptT Winner (StateT GameState Effect) Unit
turn = do
  execTurn
  state <- gets
  winner <- checkWinStates state
  when (isJust winner) do
    throwError winner
```

A more concrete example is tic-tac-toe where there are at most 9 turns. Either a player wins during their turn, or all 9 turns happen and no one wins.  This is easily represented using the pattern above:
```haskell
ticTacToe :: Effect Unit
ticTacToe = do
  result <- traverse_ identity (replicate 9 turn)
              # runExceptT
              # flip evalStateT initalState
  case result of
    Left winner -> do
      log $ winner <> " wins!"
    Right _ -> do
      log "It's a tie!"
```
