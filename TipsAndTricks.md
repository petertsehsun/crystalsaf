# User Output #
Notice that if your analysis uses the **user output**, you need to manually open the "Crystal Console" view _first_.  You can do so with Windows | Show View | Other... | Crystal | Crystal Console.

# Exceptions #
If your analysis throws **exceptions your code doesn't catch**, they will abort the current run of your analysis class.  Crystal will not catch these exceptions and instead, they will cause the child Eclipse to open a dialog informing you of an error.  Open the child's "Error Log" view (using Windows | Show View | Error Log) to see information about the exception that was thrown.

Alternatively, you can **catch exceptions yourself**.  The `analyzeCompilationUnit` and `analyzeMethod` methods in your analysis class are good places for catching "all" exceptions.

# Logging #
You can do your own **logging** using java.util.logging or another convenient package.  Crystal will output some amount of logging information in the "parent" Eclipse's console, currently using java.util.logging.

# Debugging #