Since scripts are reloaded often during runtime, their entire state has to be considered entirely transient. However, this is a problem if you need to have state that persists over the servers lifetime.

The family of classes in the exports package solve this, by providing a registry of 'exports' (basically an object) which are shared by all scripts, and persist between reloads.

The [`ScriptExportRegistry`](https://github.com/lucko/helper/blob/master/helper-js/src/main/java/me/lucko/helper/js/exports/ScriptExportRegistry.java) is provided as a binding in all scripts, and contains methods to create and retrieve exports.