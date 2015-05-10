# Introduction #

We maintain the Eclipse update site for Crystal directly in the Subversion repository, in the CrystalUpdate module.  In order to deploy a new version of Crystal to the update site, the project needs to be rebuild, and the changes committed to SVN.

# Generating the Update Site files #

The best way to build an update site is to actually go to the Crystal feature project, and "Export" a "Deployable Feature." These screens will give you options not only to include source (yay!) but also to generate the metadata necessary to build an update site. Moreover, it allows you to save an ANT script which will automate the procedure in the future. This is exactly what Nels did, and he left this script in the Crystal update site project.

# Outdated Instructions (delete?) #

The following instructions are **outdated** because: the normal Eclipse Update Site plugin, which allows you to determine which features should be included, and gives you a button to build the site, is very basic. It does not expose all of the available options.

However, in Eclipse 3.4, there are some additional hurdles to take, which are explained below.  It's probably a good idea to bump Crystal's lowermost version number to x.y.++z **after** every deployment to the update site.

  * Checkout (or update) the projects CrystalFeature and EclipseUpdate
  * Update Crystal plugin. Make sure you have no local changes.
  * Run the Ant build file javadoc.xml to generate the newest javadoc, and commit it.
  * **Outside eclipse** delete EclipseUpdate/artifacts.xml and content.xml (do not do it in eclipse as that causes Subclipse to mark these files as deleted in the repository)
  * In EclipseUpdate->site.xml, click "Add Feature..." and find Crystal (it should have the desired version number).
  * Click the "Build All" button that appears in the site.xml editor.
  * Let it do it's thang.
  * In EclipseUpdate/content.xml, replace the repository name with "Crystal Static Analysis Framework".  You can do the same in artifacts.xml.
  * Commit EclipseUpdate, and we'll have an update site for the current version of Crystal.
  * Now, get yourself a fresh eclipse and check that this actualy worked.
  * If desired, bump the version number of the CrystalPlugin and CrystalFeature projects _for the next release_.  Update the code template for "types" (in the Eclipse project preferences) accordingly.  That way, the version number of the released plugin will always be different from the version number of the plugin run from source.