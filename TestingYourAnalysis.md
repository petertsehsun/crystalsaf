# Introduction #

Writing a static analysis can be tricky. Writing one without tests is a downright bad idea! On this page, we will explain the AnnotatedTest class, a JUnit test, and how it can be used to write tests for an analysis that you have developed.

# Testing Strategy #

The AnnotatedTest class was written with a certain testing strategy in mind. We believe it to be a good one, but if you don't agree you may find that it does not suit your needs. What is this strategy?

  * Run your analysis on many small Java files.
  * Each one should test a different feature of your analysis.
  * The result of running your analysis on a input file should be either:
    * The analysis reported no errors.
    * The analysis reported errors, possibly a specific number of errors.
  * No other _interpretation_ of the testing results should be required, especially not manual interpretation.

If this seems like a good strategy, then AnnotatedTest will likely be a good fit for your needs. Certain sorts of analyses, particularly those that merely collect data for some other purpose such as visualization, may not report errors. Those analyses will probably need to be tested in a different manner.

# Using Test Annotations #

In order to use the AnnotationTest class, you will use specific annotations in your test files so that AnnotationTest can determine whether or not a particular Java file is a test, and if so which kind.

Annotations:
  * @PassingTest (edu.cmu.cs.crystal.annotations.PassingTest) Putting this annotation anywhere in a test file indicates that the analysis should run on this file and report no errors.
  * @FailingTest (edu.cmu.cs.crystal.annotations.FailingTest) Putting this annotation anywhere in a test file indicates that the analysis should run on this file and report some errors. If this annotation is given an integer greater than zero (e.g., @FailingTest(3)) then that particular number of errors must be reported.
  * @UseAnalyses (edu.cmu.cs.crystal.annotations.UseAnalyses) Putting this annotation anywhere in a test file indicates that the given set of analyses should be run. Takes an array of Strings. The names given to this annotation must match the analysis name returned by the getName method of !ICrystalAnalysis. While this annotation is allowed to be present without either of the above annotations, neither of the two above annotations can be present in a file without this one.

# Creating a Test Workspace #

In order to test your analysis, you have to run what is known as a "JUnit Plugin Test" in Eclipse. This means that your input test files must exist in a _different_ project in a _different_ workspace! I normally use my standard runtime workspace.

In your testing workspace, whichever one you choose:
  * You need to have a Java project which contains the Java classes under test. You can have more than one! AnalysisTest will look at every compilation unit in the project to see if it is a Crystal analysis test file.
  * Inside this Java project, you need to have the test annotations on your build path! The easiest way to do this? Download it from [the PLAID Annotations site](http://code.google.com/p/plaidannotations/). Alternately....
    * Go to the PlaidAnnotations project.
    * Choose to **Export** a **JAR File**.
    * Select just the classes in the edu.cs.cmu.crystal.annotations package (although having more doesn't really hurt).
  * Put this jar file in your testing workspace, inside the project where your test files are located.
  * Go back to your test project, **refresh** the project and add the new jar file to your build path.


After you have done this, annotate your test files as necessary. Here's a simple test file for the DivideByZeroAnalysis analysis:

```
import edu.cmu.cs.crystal.annotations.FailingTest;
import edu.cmu.cs.crystal.annotations.UseAnalyses;

@FailingTest(1)
@UseAnalyses("DivideByZeroAnalysis")
public class XXX_DivideByZ_test1 {
	void foo() {
		int i = 0 / 0;
	}	
}

```

# Running the Tests #

Now it's time to run those tests! This is pretty straightforward, but there are a few things that you need to watch out for! Here's how you do it:
  1. Close any open Eclipse instances that are using your test workspace.
  1. Go to the instance of Eclipse where you are developing in your Crystal/Analysis workspace.
  1. Select **Run** then **Run Configurations...**
  1. Create a new **JUnit Plugin Test** with the name of your choice.
  1. You want to **Run a single test** in the CrystalPlugin project with edu.cmu.cs.crystal.test.AnnotatedTest as your **Test class**. Make sure your are using JUnit 4 as your **Test runner**.
  1. Now go to the **Main** tab of the same dialog. Make sure the **Location** corresponds to your testing workspace! By default it probably will not!
  1. This step is important! Uncheck the **Clear** checkbox! Failure to do so will delete any files in your testing workspace!
  1. Select "[No Application ](.md) - Headless Mode" under the **Run an application** menu of **Program to Run**.
  1. You're done! Now just click the **Run** button.

# Troubleshooting #

If things don't work:

  * Make sure your analysis is 'registered.' Analyses are registered in the build.xml file, or alternatively by modifying the Crystak source code.