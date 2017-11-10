helper includes a system which allows you to magically download dependencies for your plugins at runtime.

This means you don't have to shade MBs of libraries into your jar. It's as simple as adding an annotation to your plugins class.

```java
@MavenLibrary(groupId = "org.mongodb", artifactId = "mongo-java-driver", version = "3.4.2")
@MavenLibrary(groupId = "org.postgresql", artifactId = "postgresql", version = "9.4.1212")
public class ExamplePlugin extends JavaPlugin {

    @Override
    public void onLoad() {

        // Downloads and installs all dependencies into the classloader!
        // Not necessary if you extend helper's "ExtendedJavaPlugin" instead of "JavaPlugin"
        LibraryLoader.loadAll(this);
    }
}
```