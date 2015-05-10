# Introduction #

This page gives a brief tutorial on how to create a simple Java source code analysis with Crystal.  There are basically three steps involved:

  * Install Crystal into your Eclipse
  * Create a Eclipse plugin project (must depend on the Crystal plugin)
  * Create analysis class (must implement `ICrystalAnalysis`)
  * Register analysis with Crystal (using extension point `edu.cmu.cs.crystal.CrystalAnalysis`)
  * Run your analysis

Crystal will instantiate an instance of your analysis class and call it for every compilation unit to be analyzed.

# Install Crystal #

Follow the [Installation](Installation.md) instructions.  We recommend you use the Crystal update site to install Crystal into your Eclipse.

To create analyses, you will also need the plugins Java Development Tools (org.eclipse.jdt) and Plugin Development Environment (org.eclipse.pde). (You do not need PDE to run Crystal; only to develop new plugins). The PDE plugins come with the following distributions of Eclipse:
  * Eclipse for RCP/Plugin Development
  * Eclipse Modeling Tools
  * Eclipse Classic

If you downloaded Eclipse IDE for Java Developers, you will need to take the following steps to install PDE:
  * In the menu, go to Help -> Software Updates...
  * Select the "Available Software" tab
  * Find the main Eclipse update site (In Ganymede, this is called "Ganymede Update Site")
  * Drill into "Ganymede Update Site" -> "Java Development"
  * Check "Eclipse Plugin Development Environment"
  * Click "Install..." and follow the instructions.

# Create Eclipse plug-in #

  * Create a new "Plug-in Project".
    * In the menu, go to "File" -> "New" -> "Project..."
    * Under "Plug-in Development", select "Plugin Project". (Note: If "Plug-in Project" is not available in the list then you do not have PDE installed. See above instructions.)
    * Click "Next"
    * Enter a project name of your choice and select project options to your choosing.
    * Click "Next" (You may finish at this point, but Eclipse might generate files you won't need.)
    * De-select "Generate an activator..."
    * De-select "This plugin will make contributions to the UI"
    * Click "Finish
  * Now you have a simple plugin project, and we'll add two dependencies.
    * The plugin configuration editor should be open; if it is not, open the file "MANIFEST.MF" in the META-INF directory.
    * Go to "Dependencies" (the second tab).
    * Under "Required Plugins", click "Add...".
    * Enter "edu.cmu.cs.crystal" and click "Ok". This will give you the ability to extend Crystal.
    * Click "Add..."
    * Enter "org.eclipse.jdt.core" and click "Ok". This will give you the ability to use Eclipse's Java AST.
    * Save the plugin configuration

Once you save, you will be able to see Javadoc and source code of all Crystal classes.  This may be useful for understanding in more detail what is going on.

# Create Analysis Class #

Crystal's entry point into your analysis is a class implementing the `ICrystalAnalysis` interface.  In your project, create such a class (menu File | New | Class), for example with the name `MyAnalysis`.  You have three choices:

  * Subclass `AbstractCompilationUnitAnalysis` if you would like to analyze entire compilation units (aka .java files) at once.  You only have to implement the method `analyzeCompilationUnit()` in this case, which will be called by the base class for each compilation unit to be analyzed.
  * Subclass `AbstractCrystalMethodAnalysis` if you want to analyze each method separately.  You only have to implement `analyzeMethod()` in this case, which will be called by the base class for each method to be analyzed.
  * Implement `ICrystalAnalysis` directly (discouraged, also for analyzing entire compilation units).

We recommend you subclass one of the two abstract base classes mentioned above.  This will save you from implementing all methods of `ICrystalAnalysis`.  Whatever your analysis needs to do for analyzing a compilation unit or method should go into the `analyzeCompilationUnit()` or `analyzeMethod()` method in `MyAnalysis`.

Most commonly, your analysis will want to visit the AST that is included in the `analyzeXXX` call.  You can do so by subclassing `org.eclipse.jdt.core.dom.ASTVisitor` with a class `MyVisitor` and overriding the `visit()` or `endVisit()` methods for AST nodes your analysis cares about, for example, all method or constructor calls.  You can then have the AST that Crystal passed to your analysis accept your visitor.  For example, the following code lets a new instance of your visitor, `MyVisitor`, run over every compilation unit:

```
@Override
public void analyzeCompilationUnit(CompilationUnit d) {
    d.accept(new MyVisitor());
}
```

You may want the visitor to be an inner class, as it will need many things from your Crystal analysis in order to get information and report errors.

Notice that both analysis classes define a number of additional methods that can be helpful at times, such as `before/afterAllCompilationUnits()`, which are called at the beginning and end of each analysis run, respectively.

Use `ICrystalAnalysis.getReporter()` to report problems to the user.

# Register Analysis with Crystal #

Every Crystal analysis you write (i.e., every subclass of `ICrystalAnalysis`) **must** be registered with Crystal so that Crystal can find and run your analysis.  To register your analysis:
  * Open 'MyAnalysisPlugin''s plugin configuration editor (MANIFEST.MF)
  * Go to the "Extensions" tab.
  * Click "Add," find the extension "edu.cmu.cs.crystal.CrystalAnalysis" and click "Finish"
  * The extensions list will now show "edu.cmu.cs.crystal.CrystalAnalysis". Drill into it, and you should see something with a name like "MyAnalysisPlugin.CrystalAnalysis (analysis)". Select that; this is your extension point.
  * The right side of the screen will now shown two properties for your analysis, "class" and "name".
  * Under "class", put the fully qualified name to your analysis (the one that implements ICrystalAnalysis).
  * Under "name", give it a sensible name.
  * Save. Your analysis is now registered with Crystal.

## What the heck was that about??? ##
The wizard you just went through set up the xml entry for this extension in your plugin.xml file. If you click in the tab "plugin.xml", you will see the raw xml, including something that looks like this:

```
   <extension
         point="edu.cmu.cs.crystal.CrystalAnalysis">
      <analysis
            class="MyAnalysis"
            name="My cool analysis">
      </analysis>
   </extension>
```

# Run Your Analysis #

We are almost there: now we need to run Crystal with your analysis.  To this end, you will start a second Eclipse instance inside the one in which you are developing your analysis.  We call this the "child Eclipse".  Your analysis will automatically be loaded into the child Eclipse, and running Crystal in the child Eclipse will run your analysis.  It's that easy.  You can even run the child Eclipse in debug mode to debug your analysis!

In order to run the child Eclipse, you will need a "run configuration".
  * Go to Run | Run Configurations...
  * Select "Eclipse Application" and hit the icon for a "new" configuration near the top of the dialog to create a new run configuration.
  * You should be able to just use the default settings, so simply give your run configuration a name, such as `MyAnalysisEclipse`, and hit the "Run" button.
  * This has saved this run configuration, and from now on you can simply run it by selecting it from Run | Run Configurations...  This should work even if you create additional plug-ins with more Crystal analyses or register additional analyses.
  * To debug instead, simply select the same configuration from Run |Debug Configuration...

When running `MyAnalysisEclipse`, a second Eclipse window should open.  The first time you do so, its workspace will be empty.  The child Eclipse uses a separate workspace directory, and whatever projects and files you create in the child Eclipse will be there the next time you start `MyAnalysisEclipse` (unless you change the child's workspace directory in the Run Configuration dialog or check the box that tells Eclipse to erase any existing workspace data when starting the child--usually you do not want to do either of these things!).

Now, click on the Crystal menu in the child Eclipse.  Your analysis should be listed there, with a checkmark next to it.  _That means all is well and your analysis is ready to go._  Now you can select some Java file, right-click it to bring up the pop-up menu for that file, and select Crystal | Run Analyses.  This will run all active Crystal analyses, including yours!  If your analysis creates warnings, they should show up in the problems view.

**You did it!  Your analysis is ready to save the world.**