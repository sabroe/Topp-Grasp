# How to use "decycle" with a Quarkus application

Motivation:
Using the Gradle plugin "de.obqo.decycle" may cause trouble with modules building Quarkus applications.

Solution:
To address the problem,
consider adding this the module "build.gradle" file:

```
afterEvaluate {
    tasks.named('decycleQuarkus-generated-sources') {~~~~
        dependsOn 'quarkusGenerateCodeDev', 'quarkusGenerateCode'
    }
}
```

Seen in the context of Grasp, may occur with standalone setups involving Quarkus and the "decycle" plugin.

Last checked 2025-03-04.
* "de.obqo.decycle" version 1.2.2
* Quarkus 3.19.1
* Gradle 8.13
* Java SE 23

TODO: Consider creating a complete, standalone example in the form of a Gradle module!
