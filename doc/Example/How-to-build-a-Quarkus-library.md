# How to build a Quarkus library

Motivation:
It may not be straightforward to build a Quarkus library to be used by Quarkus applications.
The Quarkus Gradle plugin kind-of throws everything at you all at once
while paying little-to-no attention to the granularity with which applications are being build.

Solution:
Given a context of, say, a module with a number of Protocol Buffers ".proto" files to be process.
We would like to have the Quarkus Gradle plugin do all the work setting this up since it may be cumbersome to do by hand and directly against Google libraries.
For coherent versioning, we would also like to benefit from Quarkus delivering a platform BOM.
Hence, we can get by with dependencies like this,
using "quarkus-grpc" and "quarkus-junit5", here in a context of using Grasp too:

```
plugins {
    id 'java'
    id 'java-library-distribution'
    id 'jacoco'
    id 'maven-publish'
    id 'signing'

    id 'io.quarkus' version "3.19.1"
}

dependencies {
    implementation enforcedPlatform("${quarkusPlatformGroupId}:${quarkusPlatformArtifactId}:${quarkusPlatformVersion}")
    api 'io.quarkus:quarkus-grpc'

    ...

    testImplementation 'io.quarkus:quarkus-junit5'
}

...

afterEvaluate {
//    if (project.hasProperty('skipQuarkusBuild')) {
        tasks.named('quarkusGenerateAppModel').configure {
//            enabled = false
        }
        tasks.named('quarkusGenerateDevAppModel').configure {
            enabled = false
        }

        tasks.named('quarkusGenerateCode').configure {
//            enabled = false
        }
        tasks.named('quarkusGenerateCodeDev').configure {
            enabled = false
        }

        tasks.named('quarkusBuildAppModel').configure {
            enabled = false
        }
        tasks.named('quarkusAppPartsBuild').configure {
            enabled = false
        }
        tasks.named('quarkusDependenciesBuild').configure {
            enabled = false
        }
        tasks.named('quarkusBuild').configure {
            enabled = false
        }

//    }
}
```

Of special interest is the list of Quarkus-specific Gradle tasks,
which leads to a clean artifact being build.

Last checked 2025-03-04.
* Quarkus 3.19.1
* Gradle 8.13
* Java SE 23

Not checked or tried in the context of a module introducing a valid "module-info.java".

TODO: Consider creating a complete, standalone example in the form of a Gradle module!
