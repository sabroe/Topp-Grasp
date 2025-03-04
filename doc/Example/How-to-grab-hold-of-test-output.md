# How to grab hold of test output

Motivation:
Gradle formats nice test reports from the output of running tests with JUnit.
How can we grab hold of, say, structure being sent and received, or structure converted from and to,
so that it is possible to see this just by clicking in the report generated?

Solution:
We can take advantage of JUnit recording STDOUT, `System.out`, for us.
Using Lombok to get access to `@Slf4J`,
we can log things of immediate interest normally.

Consider adding this to ~~~~the module "build.gradle" file:

```
...

dependencies {
    compileOnly 'org.projectlombok:lombok:1.18.36'
    annotationProcessor 'org.projectlombok:lombok:1.18.36'

    implementation 'org.slf4j:slf4j-api:2.0.17'

    ...

    testCompileOnly 'org.projectlombok:lombok:1.18.36'
    testAnnotationProcessor 'org.projectlombok:lombok:1.18.36'

    testImplementation(platform('org.junit:junit-bom:5.12.0'))
    testImplementation 'org.junit.jupiter:junit-jupiter'
    testImplementation 'org.junit.jupiter:junit-jupiter-engine'
    testImplementation 'org.junit.jupiter:junit-jupiter-api'
    testImplementation 'org.junit.jupiter:junit-jupiter-params'
    testImplementation 'org.junit.platform:junit-platform-suite'
    testImplementation 'org.junit.platform:junit-platform-suite-api'

//    testImplementation 'org.slf4j:slf4j-simple:2.0.17'
    testImplementation 'ch.qos.logback:logback-classic:1.5.17'

    test {
        useJUnitPlatform()

        testLogging {
//            events "passed", "skipped", "failed"
            events "skipped", "failed"
//            showStandardStreams = true
        }

        filter {
            includeTestsMatching "*TestSuite"
        }
    }
}
```

We can now write log statement whose output may become interesting when tests are negative 
-- "gee, which data did this weird, unreadble code sent?" --
and when tests are positive
-- "gee, I have this nice example to show, outcome is even positive, but what did actually get sent?".
The style may become something along these lines:

```
    /**
     * Tests the sending of a normal, valid BUM message.
     * @throws Exception Thrown in case of error.
     */
    @DisplayName("Send valid BUM.")
    @Test
    void sendBUM() throws Exception {
        int messageCount0=...
        log.atInfo().setMessage("Initial message count is {}.").addArgument(messageCount0).log();

        Path tempFile=...

        BIPSHelper.validate(tempFile);  //Yes, do XML validation against the XML Schema, please!
        log.atInfo().log("BIPS message is valid when validated against the XML Schema.");

        String bipsMessage=Texts.read(tempFile);
        log.atDebug().setMessage("BIPS message to be send is -\n{}").addArgument(()->Texts.indent(bipsMessage)).addKeyValue("bipsMessage",()->bipsMessage).log();

        ...

        RP1745Message expectedMessage=RP1745MessageResources.setTimestamps(DEFAULT_RP_1745_MESSAGE,now,bagCheckInTime,departureTime);
        log.atDebug().setMessage("Expected RP1745 message is -\n{}").addArgument(()->Texts.indent(expectedMessage.toFramedMessage())).addKeyValue("expectedMessage",()->expectedMessage).log();

        RP1745Message actualMessage=RP1745Message.of(newMessages.getFirst().get("Text").toString());
        log.atDebug().setMessage("Actual RP1745 message is -\n{}").addArgument(()->Texts.indent(actualMessage.toFramedMessage())).addKeyValue("actualMessage",()->actualMessage).log();

        Assertions.assertEquals(expectedMessage,actualMessage,"Failure to verify that the expected RP1745 message is equal to the actual RP1745 message.");
    }
```

Now, none of `slf4j-simple` and `logback-classic` are super-fantastic to hand multiline messages,
nor can they be trusted to show a key-value text in any sound way.
Multiline messages may not be terminated with a newline character,
key-value texts tends to be silently discard.

Best results so far are with `logback-classic`!

A possible resource for `logback-classic` is this "logback-test.xml" put under module path "test/resources":

```
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%level] %logger{36}.%method - %msg%n</pattern>
        </encoder>
    </appender>
    <root level="debug">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

For `slf4j-simple`, this "test/resources/simplelogger.properties" can be tried:

```
org.slf4j.simpleLogger.defaultLogLevel=debug
org.slf4j.simpleLogger.logFile=System.out
org.slf4j.simpleLogger.showDateTime=true
org.slf4j.simpleLogger.dateTimeFormat=yyyy-MM-dd HH:mm:ss.SSS
org.slf4j.simpleLogger.showThreadName=true
org.slf4j.simpleLogger.showLogName=false
org.slf4j.simpleLogger.showShortLogName=true
org.slf4j.simpleLogger.levelInBrackets=true
```

Last checked 2025-03-04.
* Quarkus 3.19.1
* Gradle 8.13
* Java SE 23

TODO: Consider creating a complete, standalone example in the form of a Gradle module!
