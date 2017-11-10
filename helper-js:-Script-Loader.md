The script loader represents an object capable of loading scripts and monitoring them for changes.

The HelperScriptLoader makes use of Java 8's WatchService to monitor script files for changes. When changes are detected, the script is automagically reloaded.