# Pre-reqs #
This page assumes you know [how to make a dataflow analysis](DataflowAnalysis.md).

# What is branch sensitivity? #
A branch-sensitive dataflow analysis allows you to track different information along different branches. That is, in the code below, the then-branch can know that `x` is `null`, while the else-branch can know that `x` is not null.

```
if (x == null) {
   //handle accordingly
}
else {
  x.foo(); //we know that x isn't null
}
```

Branching is really a label on the control flow graph that tells you **why** the control flow is proceeding in this way.

Notice that a branch sensitive tracks only branches, not entire paths. Crystal does **not** provide path-sensitive capabilities. A path-sensitive analysis can associate a single path, and know that in the code below, we never cause an null pointer exception.

```
x = null;
if (b) {
   x = "foo";
}
if (b) {
   x = x.concat("bar");  //Crystal can't remember that these only occur on one path.
}
```

# What kinds of branches does Crystal have? #
Branching is more than just true and false branches. Here's all the branches you can have in Crystal:
  * Normal: for normal control flow
  * Boolean: Either true or false, for a conditional branch.
  * Exception: Retains the name of the exception, for exceptional flow.
  * Switch: Retains the expression of the case, for switch-based control flow.
  * Iterator: Either "hasNext" or "empty", for iterating.

Examples of branches on a node:
  * There are two branches leaving a relational operator like <: a true branch and a false branch.
  * There are two branches leaving a method call which returns a boolean: a true branch and a false branch.
  * There are three branches leaving a method call which throws two exceptions: a normal branch and a branch for each exception.
  * There are two branches leaving the expression of an enhanced for loop: a "hasNext" branch and an "empty" branch.
  * There are four branches leaving a switch statement with 3 cases and a default: one for each case and one for the default.

# Steps to change an existing dataflow analysis #
We'll start by just converting to the branch-sensitive interface, without actually doing any branch-sensitive work. The basic steps for this are:

  * Change the base class of your transfer functions to a branch-sensitive base class (like `edu.cmu.cs.crystal.flow.AbstractTACBranchSensitiveTransferFunction`)
  * Change the signatures of the `transfer` methods from `LE transfer(TACInstruction, LE)` to `IResult<LE> transfer(TACInstruction, IList<ILabel>, LE)`
  * Wrap the returned value with `LabeledSingleResult.createResult(labels, value)`

Why are we doing that? Here's more information:

The transfer function for a branch-insensitive analysis looked something like this:

```
public LE transfer(TACInstruction instr, LE value) {
   //do stuff to value
   return value;
}
```


Now we must produce a lattice element for each outgoing branch. The transfer function will tell us which branches we need by giving us a list of `ILabel`s. To return a value for each label, we will use the `IResult` class to map each `ILabel` to a lattice element. That means we need to change the signature to look like this:

```
public IResult<LE> transfer(TACInstruction instr, IList<ILabel> labels, LE value)
```

We also need to return an `IResult` now. The class `LabeledSingleResult` is a result where every label will get the same value, so we can just use that for now.

```
public IResult<LE> transfer(TACInstruction instr, IList<ILabel> labels, LE value) {
   //do stuff to value
   return LabeledSingleResult.createResult(labels, value);
}
```

After doing the above steps, you will have an analysis that runs exactly the same as before, but now uses the branch-sensitive interfaces.

# Returning different values on different branches #
Once you've converted to the branch-sensitive interface, you can start returning different values for each branch. You can do this with the class `LabeledResult`.

To create a `LabeledResult`, call `LabeledResult.createResult(List<ILabel>, LE)` with the labels you were given and the lattice element you want for the default value. This result will use the default if someone requests a label which it doesn't have (or which you didn't set).

For each branch that you want to provide a new value for, use `LabeledResult.put(ILabel, LE)`. Remember to **copy** your lattice elements using `ILatticeOperations.copy()`, otherwise they will be mutating each other!