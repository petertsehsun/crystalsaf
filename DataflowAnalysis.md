# Pre-reqs #
This page assumes you have [installed Crystal](Installation.md) and [created and registered a simple analysis](GettingStarted.md).

# Introduction #

Crystal comes with a [dataflow analysis](http://en.wikipedia.org/wiki/Dataflow_analysis) implementation based on an efficient "worklist" algorithm.  It requires customization with analysis-specific lattice values and transfer functions.  This page explains how to customize and use the dataflow analysis implementation.  We talk about intra-procedural analyses here; however, people have successfully implemented inter-procedural analyses with Crystal as well.  This page will usually use the shorter term "flow analysis" instead of "dataflow analysis".  This page also assumes familiarity with flow analysis concepts, including forward vs. backwards analyses, lattices, and transfer functions.

Using Crystal's dataflow analysis involves the following steps:

  * Create lattice value class
  * Create a lattice operations class which implements ILatticeOperations
  * Create transfer function implementation class (multiple options; all extending `IFlowAnalysisDefinition`)
  * Instantiate and query flow analysis in your Crystal Analysis (see GettingStarted).

An important thing to realize when writing a flow analysis is that the flow analysis is just a means to an end: a flow analysis **computes** some information about **every node** in a control flow graph.  We typically query this information _subsequently_ to find error conditions.

The package `edu.cmu.cs.crystal.simple` contains a facade over more complex interfaces; we will assume you are the simple interfaces for now. The complex interfaces are in `edu.cmu.cs.crystal.flow`.

# Lattice Value Implementation #

Lattice values represent the information tracked by your flow analysis.  For instance, in a divide-by-zero analysis, a lattice _value_ would be a particular mapping from local variables to one of "zero", "non-zero" or "maybe-zero".

For your analysis, you create a class, say, `MyLE`. You can use an **enum** class if your lattice has a fixed number of constant elements. Instances of this class will represent individual lattice values, which your transfer function will typically instantiate.

If you will eventually use this lattice value in a tuple lattice (see below), the lattice element must implement `edu.cmu.cs.crystal.util.Copyable`.

```
public class MyLE implements Copyable<MyLE> {
   ...
}
```

Done. That was easy, wasn't it?

# Lattice Operations #

A lattice isn't very useful unless you can do some operations on it. You'll need to create a class which implements `ILatticeOperations`. We suggest extending off of `edu.cmu.cs.crystal.simple.SimpleLatticeOperations`. You must parameterize your operations with the lattice element type you created earlier. In our example, your new class might start like this:

```
public class MyLatticeOps extends SimpleLatticeOperations<MyLE> {
   ...
}

```

Implementing `SimpleLatticeOperations` will require implementing a number of methods, most of which correspond directly to lattice theory:

  * `atLeastAsPrecise` compares two of your lattice values and determines if the information in the receiver implies the information in the argument (i.e., if the receiver is at least as precise as the argument). This is the comparison operator in lattice theory.

  * `join` returns a new value that approximates the receiver and argument values. This is the join operator in lattice theory.

  * `bottom` returns a new value that is the bottommost element in the lattice.

  * `copy` performs a "deep copy" of the receiver. **If the copy and original reference the same mutable state, it will lead to subtle bugs when running the flow analysis.** Notice that if you are using an enum for your lattice elements, or some other immutable type, you can just return `this`.

While the first three methods are from lattice theory, `copy` is purely an implementation device.  Since Crystal needs to keep analysis information separately for every program point it will call `copy` to give you a separate copy of the analysis information to work with in transfer functions (see below).  Notice that Crystal will `copy` all the time, so if your lattice values are really big, you may want to think about some [LatticeOptimizations](optimizations.md) here.

# Tuple Lattice Elements #
Under `edu.cmu.cs.crystal.simple`, there are two classes called `TupleLatticeElement` and `TupleLatticeOperations`. A tuple lattice is a mapping lattice, it will track a separate lattice element for every key. So, for example, if you want to track each variable and determine whether or not it is null, you would really want a tuple lattice. This will map each variable to your own null lattice.

Since Crystal provides this class for you, all you need to worry about is your own lattice, not the tuple part. If you want to use this though, your lattice will need to implement `Copyable'.

You should never need to extend these classes, but you may use them in the next step.

# Transfer Function Implementation #

A transfer function "transfers" your analysis information over a given program instruction.  In other words, if N is an instruction and IN is a given lattice value (called the "incoming" value) then a transfer function is a function f that computes new analysis information `OUT = f(N, IN)` from the incoming information and what is going on in the instruction.  For instance, if you're tracking whether variables are null, and N is `x = null`, then you want OUT to be like IN, except with whatever I says about x changed to x being definitely null. That is, `f(x=null, IN) = IN[x->NULL]`.

Now there are a ton of different kinds of instructions (method calls, assignments, arithmetic, etc.), and the behavior of a transfer function tends to depend greatly on the **kind of instruction** it transfers over.  So transfer functions are typically implemented with separate methods for each kind of instruction.

All transfer functions in Crystal implement `IFlowAnalysisDefinition`. There are many derived classes off this, so you'll need to choose which to extend. We suggest starting with `edu.cmu.cs.crystal.simple.AbstractingTransferFunction`, which uses ThreeAddressCode for the transfer functions. So your class will look something like this:

```
public class MyTransferFunctions extends AbstractingTransferFunction<MyLE> {
   ...
}
```

And if you are using a tuple lattice over variables, it will look like this:
```
public class MyTransferFunctions extends
 AbstractingTransferFunction<TupleLatticeElement<Variable, MyLE>> {
   ...
}
```

You'll need to define **two** methods to get the analysis started. After doing this, the analysis is actually usable by your Visitor. It will only give you back the default of course, but you can override transfer functions to do more interesting behavior.

## Lattice Operations ##
The 'createLatticeOperations' needs to return the lattice operations that should be used. We suggest actually storing an instance of a lattice operations in a field, and then returning the field reference here.

Remember, if you are using a tuple lattice, you'll need to return a TupleLatticeOperations with the underlying lattice operations and a default lattice element. This might look something like:

```
TupleLatticeOperations<Variable, MyLE> ops =
  new TupleLatticeOperations<Variable, MyLE>(new MyLatticeOps(), MyLE.TOP_ELEMENT);
```

## Starting Lattice ##

The `createEntryValue` method must return a lattice element which Crystal will use as the initial lattice value for analyzing a given method.  This gives you the opportunity to pre-populate the lattice with information known at the beginning of a method, such as assumptions about the method receiver and parameters.

If you are using a tuple lattice, you can use `TupleLatticeOperations.getDefault()` to get a `TupleLatticeElement` where every value is set to the default. You could then change values as desired, possibly like:

```
   TupleLatticeElement<Variable, MyLE> def = ops.getDefault();
   def.put(getAnalysisContext().getThisVariable(), MyLE.SOME_ELEMENT);
   return def;
```

## The Transfer Functions ##

Finally, we come to the transfer functions. We suggest using ThreeAddressCode for transfer functions, as it is simpler to use than the Eclipse AST.

For each instruction type that you are interested in, simply override the appropriate transfer function for that instruction type. So, if you want the `CopyInstruction` (`x = y;`) to copy over the lattice element too, you might write a transfer function like this:

```
@Override
public TupleLatticeElement<Variable, MyLE> transfer(CopyInstruction instr, TupleLatticeElement<Variable, MyLE> value) {
   value.put(instr.getTarget(), value.get(instr.getOperand()));
   return value;
}
```

By default, the `AbstractingTransferFunction` will just use the transfer function for the base class of the instruction (which is `OneOperandInstruction` for the `CopyInstruction`), and the top level, `TACInstruction`, will just return the input lattice value.

# Getting the flow results from the visitor #
At this point, you should have:
  * a lattice element class (`MyLE`)
  * a lattice operations class (`MyLatticeOps`)
  * transfer functions (`MyTransferFunctions`)
  * registered a Crystal analysis and have a visitor

Now, all you have to do is create the flow analysis and query for results.

## Create the flow analysis ##
In your Crystal analysis (usually in `analyzeMethod`), you need to create an instance of your transfer functions and pass them to a flow analysis. You must then store the flow analysis so that the visitor can query it.

The flow analysis you use must derive from IFlowAnalysis, and must match your transfer functions. That is, if your transfer functions are using ThreeAddressCode, you need to use `edu.cmu.cs.flow.TACFlowAnalysis`, which also uses ThreeAddressCode. An example is below:

```
@Override
public void analyzeMethod(MethodDeclaration d) {
   MyTransferFunctions tf = new MyTransferFunctions();
   flowAnalysis = new TACFlowAnalysis<TupleLatticeElement<Variable, MyLE>>(tf, getInput());
		
   d.accept(new MyVisitor());
}
```

## Query the flow analysis ##
In your visitor, you can now query the flow analysis and report errors:

```
@Override
public void endVisit(MethodInvocation node) {
   TupleLatticeElement<Variable, MyLE> beforeTuple = flowAnalysis.getResultsBefore(node);
			
   Variable varToCheck = flowAnalysis.getVariable(node.getExpression());
   MyLE element = beforeTuple.get(varToCheck);
			
   if (element == MyLE.BAD_VALUE)
      getReporter().reportUserProblem("The expression " + node.getExpression() + " was bad.", node.getExpression(), getName());				
}
```

Notice that the visitor will need a lot of methods/fields from your Crystal analysis, which is why we suggest making it an inner class.