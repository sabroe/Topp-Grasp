# How to build a distribution package for a Quarkus application

Motivation:
Using the Gradle plugin "distribution" to create a distribution package using e.g. task "distZip" 
with a Quarkus application built in the normal, straightforward "fast" way.

For something build normally:

```
gradle build
```

This can be run like:

```
java -jar app/quarkus-run.jar
```


Solution:
To get a distribution package under "build/distributions",
consider adding these lines to the module "build.gradle" file:

```
tasks.named("distZip") {
    dependsOn("quarkusBuild")
}

tasks.named("distTar") {
    dependsOn("quarkusBuild")
}

distributions {
    main {
        contents {
            from("build/quarkus-app") {
                into "app"
            }
        }
    }
}
```

This makes the package contain everything,
the built Quarkus application,
except a Java VM.

You still have to remember how to invoke the application:

```
java -jar app/quarkus-run.jar
```

Last checked 2025-03-04.
* Quarkus 3.19.1
* Gradle 8.13
* Java SE 23

TODO: Consider creating a complete, standalone example in the form of a Gradle module!
