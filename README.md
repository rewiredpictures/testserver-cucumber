## Cucumber/BDD testing for REST APIs 

This project provides a generic Cucumber vocabulary for testing APIs with the Ready! API TestServer,
with special support for Swagger to remove some of the technicalities required to define scenarios. 

A quick example for the Petstore API at http://petstore.swagger.io, testing of the 
/pet/findByTags resource could be defined withe following Scenario:

```gherkin
  Scenario: Find pet by tags
    Given the API running at http://petstore.swagger.io/v2
    When a GET request to /pet/findByTags is made
    And the tags parameter is test
    And the request expects json
    Then a 200 response is returned within 50ms
    And the response type is json
```

Using the integrated Swagger support this can be shortened to

```gherkin
  Scenario: Find pet by tags
    Given the Swagger definition at http://petstore.swagger.io/v2/swagger.json
    # deducts path and method from Swagger definition by operationId
    When a request to findPetsByTags is made
    # deducts type of "tags" parameter (query/path/parameter/body) from Swagger definition
    And tags is test
    And the request expects json
    Then a 200 response is returned within 500ms
    And the response type is json
```

Not a huge difference - but as you can see by the comments the Swagger support removes some of the 
technicalities; read more about Swagger specific steps and vendor extensions below!

Check out the [samples](modules/samples) submodule for more examples.

### Usage with maven

If you want to run scenarios as part of a maven build you need to add the following 
dependency to your pom:

```xml
<dependency>
    <groupId>com.smartbear.readyapi.testserver.cucumber</groupId>
    <artifactId>testserver-cucumber-core</artifactId>
    <version>1.0.0</version>
    <scope>test</scope>
</dependency>
```

Then create a JUnit runner class that uses Cucumber and point it to your feature files:
 
```java
@RunWith(Cucumber.class)
@CucumberOptions(
    plugin = {"pretty", "html:target/cucumber"},
    features = {"src/test/resources/cucumber"})
public class CucumberTest {
}
```

(see the included samples module for a working project)

### Usage without maven

If you're not running java or simply want to run cucumber tests from the command-line you can use the 
testserver-cucumber-all jar file which includes all required libraries including the Cucumber
runtime. Run tests with:

```
java -jar testserver-cucumber-all-1.0.0.jar <path to feature-files>
```

Internally this will call the regular cucumber.api.cli.Main class with an added -g argument to the
included glue-code, all other options are passed as usual, see https://cucumber.io/docs/reference/jvm#cli-runner

(you will need java8 installed on your path)

### Configuring Ready! API TestServer access
 
The included [Cucumber StepDefs](modules/core/src/main/java/com/smartbear/readyapi/testserver/cucumber/GenericRestStepDefs.java) 
build and execute test recipes agains the Ready! API TestServer using the 
[testserver-java-client](https://github.com/SmartBear/ready-api-testserver-client), by default they 
will submit recipes to the publicly available TestServer at http://testserver.readyapi.io. If you 
want to run against your own TestServer instance to be able to access internal APIs or not run into 
throttling issues you need to download and install the TestServer from 
https://smartbear.com/product/ready-api/testserver/overview/ and configure access to it by 
specifying the corresponding system properties when running your tests:

- testserver.endpoint=...url to your testserver installation...
- testserver.user=...the configured user to use...
- testserver.password=...the configured password for that user...

(these are picked up by [CucumberRecipeExecutor](modules/core/src/main/java/com/smartbear/readyapi/testserver/cucumber/CucumberRecipeExecutor.java) during execution)

### Building 

Clone this project and and run
 
```
mvn clean install 
```

To build and install it in your local maven repository.

## API Testing Vocabulary
 
The included glue-code for API testing adds the following vocabulary:

##### Given statements

- "the Swagger definition at &lt;swagger endpoint&gt;"
    - The specified endpoint must reference a valid Swagger 2.0 definition
    - Example: "the Swagger definition at http://petstore.swagger.io/v2/swagger.json"

- "the API running at &lt;API endpoint&gt;"
    - Example: "the API running at http://petstore.swagger.io/v2"

##### When/And statements

- "a &lt;HTTP Method&gt; request to &lt;path&gt; is made"
    - Example: "a GET request to /test/search is made"
    
- "a request to &lt;Swagger OperationID&gt; is made"
    - will fail if no Swagger definition has been Given
    - Example: "a request to findPetById is made"

- "the request body is" &lt;text block&gt;
    - Example: "the request body is
    ```
    """
    { "id" : "123" }
    """
    ```"
    
- "the &lt;parameter name&gt; parameter is &lt;parameter value&gt;"
    - adds the specified parameter as a query parameter
    - Example: "the query parameter is miles davis"
    
- "the &lt;http header&gt; is &lt;header value&gt;
    - Example: "the Encoding header is UTF-8"
    
- "the type is &lt;content-type&gt;
    - single word types will be expanded to "application/&lt;content-type&gt;"
    - Example: "the type is json"

- "&lt;parameter name&gt; is &lt;parameter value&gt;"
    - if a valid OperationId has been given the type of parameter will be deduced from its list of parameters
    - if no OperationId has been given this will be added to a map of values that will be sent as the request body
    - Example: "name is John"
    
- "&lt;the request expects &lt;content-type&gt;"
    - adds an Accept header
    - Example "the request expects yaml"

##### Then/And statements:

- "a &lt;HTTP Status code&gt; response is returned"
    - Example: "a 200 response is returned"
    
- "a &lt;HTTP Status code&gt; response is returned within &lt;number&gt;ms"
    - Example: "a 404 response is returned within 10ms"

- "the response is &lt;a valid Swagger Response description for the specified operationId&gt;"
    - Requires that a valid OperationId has been Given
    - Example: "the response is a list of people"

- "the response body contains" &lt;text block&gt;
    - Example: "the response body contains
    ```
    """
    "id" : "123"
    """
    ```"

- "the response body matches" &lt;regex text block&gt;
    - Example: "the response body matches
    ```
    """
    .*testing.*
    """
    ```"

- "the response type is &lt;content-type&gt;"
    - Example: "the response type is application/yaml"

- "the response contains a &lt;http-header name&gt; header"
    - Example: "the response contains a Cache-Control header"

- "the response &lt;http header name&gt; is &lt;http header value&gt;"
    - Example: "the response Cache-Control header is None"

- "the response body contains &lt;text token&gt;"
    - Example: "the response body contains Testing text"

### Complete Example:

Below is the [swaggerhub.feature](modules/samples/src/test/resources/cucumber/swaggerhub.feature) in the 
[samples](modules/samples) submodule.

```gherkin
Feature: SwaggerHub REST API

  Background:
    Given the Swagger definition at https://api.swaggerhub.com/apis/swagger-hub/registry-api/1.0.10

  Scenario: Default API Listing
    When a request to searchApis is made
    Then the response is a list of APIs in APIs.json format

  Scenario: Owner API Listing
    When a request to getOwnerApis is made
    And owner is swagger-hub
    Then the response is a list of APIs in APIs.json format

  Scenario: API Version Listing
    When a request to getApiVersions is made
    And owner is swagger-hub
    And api is registry-api
    Then the response is a list of API versions in APIs.json format
    And the response body contains
      """
      "url":"/apis/swagger-hub/registry-api"
      """

  Scenario Outline: API Retrieval
    When a request to getDefinition is made
    And owner is <owner>
    And api is <api>
    And version is <version>
    Then a 200 response is returned within 500ms
    And the response type is json
    And the response body contains
    """
    "description":"<description>"
    """
    Examples:
    | owner       | api          | version  | description                       |
    | swagger-hub | registry-api | 1.0.10   | The registry API for SwaggerHub   |
    | fehguy      | sonos-api    | 1.0.0    | A REST API for the Sonos platform |
```


## Contribute!

If you're missing something please contribute or open an issue!
