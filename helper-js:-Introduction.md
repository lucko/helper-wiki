helper-js is an addon which allows you to create JavaScript plugins using helper.

To achieve this, helper uses the Java scripting API and Nashorn, Java 8's JavaScript engine.

Nashorn (in a nutshell) lets you:

* Invoke Java code from JS
* Invoke JS code from Java
* Use Java classes and APIs within JS
* Easily modify code at runtime

helper-js provides an interface around Nashorn, to aid with the specific task of writing Bukkit plugins (or "scripts" as they'll be called from now on) using the Bukkit API and helper, all in JavaScript. helper-js is a standalone plugin, which has to be installed separately from helper (and unlike helper cannot be shaded!).