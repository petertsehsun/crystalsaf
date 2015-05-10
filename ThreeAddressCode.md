# Three Address Code #
Three address code is a representation of a program where there are no sub-expressions. Each expression is broken into smaller parts by using temporary variables. Thus, the following statement:

```
x = a.foo().bar(b, c.d, baz(e + f));
```

becomes:

```
t1 = a.foo();
t2 = c.d;
t3 = e + f;
t4 = baz(t3);
x = t1.bar(b, t2, t4);
```

This makes Three address code (TAC) much easier to analyze since you don't have to analyze sub-expressions; each transfer function is already working on the bottom-most level.

Additionally, there are no nodes in Crystal's TAC that represent control flow of any kind: no while statements, no ternary conditionals, and no && and || (short-circuiting is a form of control flow). All control flow is handled by the control flow graph, and these control statements are broken down into their sub parts.

**Put in the example of && here!**

This means that overall, there are also less types of nodes to transfer over, thus making Crystal's TAC easier to understand than most other abstract syntax trees (like Eclipse's `ASTNode` hierarchy).

You will almost never need to transfer on a flow node itself, but if you do, you will need to use the transfer function interfaces `edu.cmu.cs.crystal.flow.ITransferFunction` or `edu.cmu.cs.crystal.flow.IBranchSensitiveTransferFunction`, which work on `ASTNode`s rather than `TACInstruction`s.

[Javadoc of Crystal's TAC package](http://crystalsaf.googlecode.com/svn/trunk/CrystalPlugin/doc/edu/cmu/cs/crystal/tac/package-summary.html)