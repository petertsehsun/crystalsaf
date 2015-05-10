# Introduction #

Getting a Crystal analysis going can seem daunting, but it's actually quite easy. The wiki covers lots of topics; this page provides some high-level guidance and checkpoints to make sure things worked out ok.

Instructors can use each step as a checkpoint to make sure that students aren't lost, or students can follow this themselves to prevent silly mistakes.

# 1. Install Crystal #
**Reading and instruction:** [Installation](Installation.md)

**Verification:** You should see a "Crystal" menu in Eclipse.

# 2. Register your analysis #
**Reading and instruction:** [GettingStarted](GettingStarted.md) (don't bother with the ASTVisitor yet, just get an ICrystalAnalysis registered).

**Verification:** Start a child Eclipse. Your analysis name will appear under the Crystal menu.

# 3. Create a visitor #
**Reading and instruction:** [GettingStarted](GettingStarted.md) (now make a visitor too!)

**Verification:** Start the child Eclipse and run your analysis.

# 4. Create a flow analysis #
**Reading and instruction:** [DataflowAnalysis](DataflowAnalysis.md) (don't override any transfer functions, but get everything put together and have the visitor call to the flow analysis.)

**Verification:** Start the child Eclipse and run your analysis. It will give the same results as for the above step, since your flow analysis isn't transferring yet.

# 5. Start making those transfer functions smart! #
**Reading and instruction:** [DataflowAnalysis](DataflowAnalysis.md) (add in those transfer functions now)

**Verification:** Start the child Eclipse, you should now have a smarter analysis!

Throughout this process, we suggest working on one transfer function at a time to incrementally make your analysis better.

# Advanced topics #
At this point, instructors might suggest students can use other features of Crystal to make their analysis smarter. (There are plenty of possibilities for extra credit here!) Some possibilities are:

[Use specifications (in form of Java annotations)](UsingAnnotations.md)

[Use a branch sensitive analysis](BranchSensitivity.md)