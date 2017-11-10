## Using helper in your project
You can either install the standalone helper plugin on your server, or shade the classes into your project.

You will need to add my maven repository to your build script, or install helper locally.
#### Maven
```xml
<repositories>
    <repository>
        <id>luck-repo</id>
        <url>http://repo.lucko.me/</url>
    </repository>
</repositories>
```

#### Gradle
```gradle
repositories {
    maven {
        name "luck-repo"
        url "http://repo.lucko.me/"
    }
}
```

Then, you can add dependencies for each helper module.

### helper
#### Maven
```xml
<dependencies>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper</artifactId>
        <version>3.1.5</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### Gradle
```gradle
dependencies {
    compile ("me.lucko:helper:3.1.5")
}
```

### helper-sql
#### Maven
```xml
<dependencies>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-sql</artifactId>
        <version>1.0.4</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### Gradle
```gradle
dependencies {
    compile ("me.lucko:helper-sql:1.0.4")
}
```

### helper-redis
#### Maven
```xml
<dependencies>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-redis</artifactId>
        <version>1.0.5</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### Gradle
```gradle
dependencies {
    compile ("me.lucko:helper-redis:1.0.5")
}
```

### helper-mongo
#### Maven
```xml
<dependencies>
    <dependency>
        <groupId>me.lucko</groupId>
        <artifactId>helper-mongo</artifactId>
        <version>1.0.2</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### Gradle
```gradle
dependencies {
    compile ("me.lucko:helper-mongo:1.0.2")
}
```