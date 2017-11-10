The whole helper-js system is based around `Script`s. These in a more abstract sense are the javascript files which contain the code responsible for implementing the desired behaviour.

However, programmatically they are formed of:

* **a name** - the name of the script, usually formed by taking the script name (e.g. "myscript.js") and removing the extension (e.g. "myscript")
* **a path** - the path where the actual script file is located
* **bindings** - these are explained later
* **a logger** - plugins have loggers, scripts do too!
* **dependencies** - the other scripts currently in the system which are depended upon by the script

Scripts also implement `Runnable`, which when invoked, evaluates the script file.