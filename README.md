# concur-smart-connector
Concur connector using *Smart Connector* feature to showcase it.

This connector implementation relies in
some core connectors (`HTTP Connector`, `Object Store Connector`) to perform the requests and store some temporal information (such as the session),  and on others (such as `File Connector`) to mimic a usage from a user POV while working with binary data.

The entire Concur connector has been tested using MUnit.
********
### Project structure
This is a multi module project, with the following structure:

    concur-smart-connector
    ├── README.md
    ├── pom.xml
    ├── smart-connector   <[Concur connector project]>
    │   ├── pom.xml
    │   └── src
    │       └── main
    │           └── resources
    │               ├── module-concur-catalog.xml <[Concur connector's type catalog]>
    │               ├── module-concur.xml  <[Concur connector's code]>
    │               └── schema
    │                   ├── get-report-list-result.xsd
    │                   ├── post-entry.json
    │                   ├── post-generic-result-v1.xsd
    │                   ├── post-generic-result.xsd
    │                   └── post-report.json
    └── smart-connector-it <[Concur connector integration test project]>
        ├── pom.xml
        └── src
            ├── main
            │   ├── mule
            │   │   └── mule-config.xml <[Concur connector integration test project's flows]>
            │   └── resources
            │       └── automation-credentials.properties  <[Concur credentials, must provide valid ones before running the tests]>
            └── test
                ├── munit
                │   └── assertion-munit-test.xml <[Concur connector integration test project's assertions]>
                └── resources
                    ├── log4j2-test.xml
                    └── mulesoft-logo-test.jpg

### Build and run the tests
1. Git clone this repo
2. Edit `[..]/concur-smart-connector/smart-connector-it/src/main/resources/automation-credentials.properties` putting valid `username`, `password` and `consumerKey`.
3. run `mvn clean install` from the root element (`[..]/concur-smart-connector`).
                             
#### Run partial tests
As this API often fails (server errors, aka: 500), the easiest way to at least check if it builds on your machine is to, having previously put the valid credentials under `automation-credentials.automation` file, run the `test-login` call which it does not throw a 500 error (from my current knowledge..).

To do so, from the folder `[..]/concur-smart-connector/smart-connector-it` execute `mvn clean test -Dmunit.test=assertion-munit-test.xml#test-login `. 

The syntax to run a single test is 
`mvn clean test -Dmunit.test=assertion-munit-test.xml#<nameOfTheTest>`. More samples:
      
      mvn clean test -Dmunit.test=assertion-munit-test.xml#test-get-reports
      mvn clean test -Dmunit.test=assertion-munit-test.xml#test-post-report
      mvn clean test -Dmunit.test=assertion-munit-test.xml#test-post-entry
      mvn clean test -Dmunit.test=assertion-munit-test.xml#test-get-report-list-flow
      mvn clean test -Dmunit.test=assertion-munit-test.xml#test-submit-report-flow
#### Debugging the tests
If some test fails, enable the logs under `[..]/concur-smart-connector/smart-connector-it/src/main/test/log4j2-test.xml` to see what's being sent to the Concur's API and what's coming back from it by uncommenting the following lines:
 
     <AsyncLogger name="org.mule.service.http" level="DEBUG"/>
     <AsyncLogger name="org.mule.extension.http" level="DEBUG"/>
     
Notice that enabling these logs will make the `test-post-receipt` test show some weird stack in the console as an image will be sent to the API through a POST request, causing the terminal go nuts due to the weird characters. 
      
********
### Missing features
* Reconnection, relying with the `<try>` or `<until-successful>` blocks
* Lots of API endpoints that are not mapped, this is a simple POC.

********
### Concur API's issues (500, or "Unexpected error")
Concur API might fail due to unknown issues, if that's the case an error similar to the following could appear on any call:
```
<Error>
    <Message>Unexpected error.Please contact Developer Support</Message>
    <Server-Time>2017-12-22T16:28:53</Server-Time>
    <Id>A6B61B80-D057-4AE4-B44D-63AA0713B00C</Id>
</Error>
```
If that's the case, you must either (a) talk to Concur support and provide the `ID` or (b) pray to some god and wait a few hours until the hiccup gets fixed.
This is **way** too common, so the majority of the tests might fail all the time, those I've seen work all the time are:
`test-login`, `test-logout-without-login`, `test-logout-with-login`

