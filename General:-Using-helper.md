## Using helper in your project
You can either install the standalone helper plugin on your server, or shade the classes into your project.

In order to import helper into your project, I recommend using a build system such as Maven or Gradle. helper artifacts are published to the Maven Central repository - you do not need to depend on a 3rd party repository.

The latest versions of each module can be found on the [README](https://github.com/lucko/helper#modules) page.

If you do choose to shade the helper dependency into your plugin, please ensure you relocate the shaded classes to avoid conflicts. I do however recommend just installing the standalone plugins. These can be obtained from [Jenkins](https://ci.lucko.me/job/helper/).

#### Maven
```xml
<dependencies>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper</artifactId>
        <version>x.y.z</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-sql</artifactId>
        <version>x.y.z</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-redis</artifactId>
        <version>x.y.z</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-mongo</artifactId>
        <version>x.y.z</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-lilypad</artifactId>
        <version>x.y.z</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-profiles</artifactId>
        <version>x.y.z</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### Gradle
```gradle
repositories {
    mavenCentral()
}

dependencies {
    compile ("me.lucko:helper:x.y.z")
    compile ("me.lucko:helper-sql:x.y.z")
    compile ("me.lucko:helper-redis:x.y.z")
    compile ("me.lucko:helper-mongo:x.y.z")
    compile ("me.lucko:helper-lilypad:x.y.z")
    compile ("me.lucko:helper-profiles:x.y.z")
}
```
