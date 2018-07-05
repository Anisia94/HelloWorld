# Automation project for U4A Application

U4A_AUT is an automation project which

## Tools

- [Selenium](https://www.seleniumhq.org)
- [WireMock](http://wiremock.org/)
- [ActiveJdbc](http://javalite.io/activejdbc)
- [JDBCMockRunner](http://mockrunner.github.io/)

## Requirements

In order to utilise this project you need to have the following installed locally:

- The application we're going to test
- Internet Explorer 11.0 or higher
- [JDK 8](http://www.oracle.com/technetwork/pt/java/javase/downloads/jdk8-downloads-2133151.html) is required to build and run this project
- [The Internet Explorer Driver Server](https://goo.gl/PdgGmZ) (downloaded on a known location from C: disk)
- [IntelliJ IDEA Community Edition](https://www.jetbrains.com/idea/download/#section=windows)
- [Maven 3](https://maven.apache.org/download.cgi) (this project is managed by maven)
- [Git](https://git-scm.com/download/win)


## Setup the projects
Firstly, we have to download the application in which the automated tests will be written:

1. Clone the [U4A_AUT](https://git.gcio.unicredit.eu/RO23/U4A_AUT) repository from GitLab
2. Import the project in Intellij IDEA as a maven project

Secondly, we need to have the application we test, up and running locally:

1. Clone all three modules [U4A_WEB](https://git.gcio.unicredit.eu/RO23/U4A_WEB), [U4A_CMN](https://git.gcio.unicredit.eu/RO23/U4A_CMN) and [U4A_PLF](https://git.gcio.unicredit.eu/RO23/U4A_PLF) repositories from GisoLab
2. Run the following command in U4A-WEB/source/U4A-EBA-ADV-MVN folder using git bash:

    `clean install -Dmaven.test.skip=true -Dliberty`

## Usage

To start developing and executing automated test cases we need to go through 2 phases:  **Recording and Playback**.
These approaches are used to mock external dependencies (webservices and stored procedures called during the tests execution) and then used them in our tests.

### 1. Recording

In this phase, we'll launch the U4A application locally and execute each test case manually in order to record all the webservices and SP called. These mocks will be used in the playback phase.
We have to navigate to U4A-WEB/source/U4A-EBA-ADV-EAR folder and execute in bash the following maven command:

   `mvn clean install -DautRec -Dliberty`

After that, we launch the application server using :

   `mvn liberty:run-server -Pliberty,development`

The recording of webservices is performed using several running wiremock servers, which will create stub mappings while we send requests to webservices. Basically,
the servers will capture a collection of responses from the real services, which we'll use offline in our tests.

To launch the servers we have to run the [src/test/resources/recording_wiremock.bat](recording_wiremock.bat) and then we have to execute the tests manually.
The recordings are stored in [src/test/resources/recorded_webservices](recorded_webservices).

Additionally and mandatory, for each test, we have to process the recorded webservices to substitute the containing dates with placeholders. This step is necessary because we have to
use dynamics dates according to the day in which tests are executed.
For this purpose the command bellow should be executed :
 `mvn -q compile -DskipTests exec:java -Dexec.mainClass="org.unicredit.u4a.automation.wiremock.services.Automation"`

This will add placeholders in recorded webservices. Then, in playback phase, these will be replaced with the corresponding dates.
Also the responses from called stored procedures are stored in the application we test and ready to be used in playback mode.

### 2. Playback

     In this phase, some configuration need to be done in order to use all the registered responses from webservices and SP from the previous step. These responses are used in automated tests.
     So let's run the app in playback mode:

   Using git bash go to U4A app in U4A-WEB/source/U4A-EBA-ADV-EAR module and run :

   `clean install -DautPlay -Dliberty`

   Start the application server as in the previous stage using :

     `mvn liberty:run-server -Pliberty,development`

   Now move back, on the current application U4A_AUT and run:

   `mvn -q compile -DskipTests exec:java -Dexec.mainClass="org.unicredit.u4a.automation.wiremock.services.PlaybackWiremock"`

   We've just launched a wiremock server which will serve us the recorded webservices.

   Finally, we are ready to execute automated tests.

















