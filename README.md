# foldl

`foldl` is a Unison module that allows you to compose strict left folds such that
their results can be combined in a single pass of input data.

This is a partial Unison port of the Haskell [foldl](https://hackage.haskell.org/package/foldl-1.1.2)
module, and you'll find much better documentation there.

## getting the dependency

In a `ucm` session, run the following. Feel free to change `external.foldl` to whatever
namespace you prefer.

    .> pull https://github.com/ceedubs/unison-dev.git:.foldl.trunk external.foldl

## usage

### simple folds

One of the simplest folds that you can perform is to simply count the number of
input elements.

    folds.Nat.count.examples.nonempty = List.fold count [1, 2, 3]
    ↳ 3

Similarly you can calculate the sum of the input.

    folds.Nat.sum.examples.nonempty = List.fold Nat.sum [1, 2, 3]
    ↳ 6

### composite folds

You can compose folds using [applicative](http://hackage.haskell.org/package/base-4.14.0.0/docs/Control-Applicative.html)-style
composition. The combined results of these folds will be calculated in a single
pass of the data.

    Fold.ap.examples.paired = List.fold (pair <$> Nat.sum <*> count) [1, 2, 3]
    ↳ (6, 3)


```
Fold.ap.examples.badAverage =
  use Nat /
  (/) <$> Nat.sum <*> count

```


There are a couple of issues with this average function. Since it is using `Nat`
division, the result may be truncated. Also it will throw a `divide by zero` error
if you pass in an empty list. You are better off using the `average` function provided
by this library. The `average` function expects the input to have type `Float`,
which we can accomplish with a `premap` on the input to the `average` fold.

```
Fold.premap.examples.natToFloat =
  List.fold (premap Nat.toFloat average) [1, 2, 3, 4]

↳ Some 2.5

```

The result type is `Optional Float`, because we might not be able to calculate the
average if there are zero items in the input. If you want to map the output of the
fold in addition to pre-mapping the input, you can use `dimap` and supply both.
Let's map the output to return `0.0` instead of `None` if there are no items in
the input.

```
Fold.dimap.examples.natAverageOrZero =
  dimap Nat.toFloat (orDefault 0.0) average

Fold.dimap.examples.natAverageOrZeroNonemptyFold =
  List.fold natAverageOrZero [1, 2, 3, 4]

↳ 2.5

Fold.dimap.examples.natAverageOrZeroEmptyFold = List.fold natAverageOrZero []

↳ 0.0

```

## module API

Here is a list of the terms included in this library.

```haskell
type Fold a b
type Fold' a b x
Fold'.Fold' : (x -> a -> x) -> x -> (x -> b) -> Fold' a b x
Fold.<$> : (b -> c) -> Fold a b -> Fold a c
Fold.<*> : Fold a (b -> c) -> Fold a b -> Fold a c
Fold.Fold : (∀ r. (∀ x. Fold' a b x -> r) -> r) -> Fold a b
Fold.List.fold : Fold a b -> [a] -> b
Fold.Set.fold : Fold a b -> Set a -> b
Fold.dimap : (a -> b) -> (c -> d) -> Fold b c -> Fold a d
Fold.fromFold' : Fold' a b x -> Fold a b
Fold.map : (b -> c) -> Fold a b -> Fold a c
Fold.mkFold : (x -> a -> x) -> x -> (x -> b) -> Fold a b
Fold.premap : (a -> b) -> Fold b c -> Fold a c
Fold.runFoldl : (∀ a b. (b -> a -> b) -> b -> f a ->{𝕖} b) -> Fold a b -> f a ->{𝕖} b
find : (a -> Boolean) -> Fold a (Optional a)
folds.Float.average : Fold Float (Optional Float)
folds.Float.sum : Fold Float Float
folds.Nat.count : Fold a Nat
folds.Nat.sum : Fold Nat Nat
folds.all : (a -> Boolean) -> Fold a Boolean
folds.any : (a -> Boolean) -> Fold a Boolean
```
