# MockWebserver extensions Kotlin

This project was split out form [mockmvc-extensions-kotlin](http://github.com/Skatteetaten/mockmvc-extensions-kotlin) since we needed an extension without spring-web.

The code for this project is developed and built from and internal repository and synced to github for releases. 

## Custom Dispatcher DSL
An alternate way of mocking is by using our httpMockServer dsl that uses a custom dispatcher.
```
val server:MockWebServer = httpMockServer(8282) {
  rule({ path?.endsWith("sith")}) {
    MockResponse().setBody("Darth Vader")
  }
  rule {
    MockResponse().setBody("Yoda")
  }
}
```

This will create a MockWebServer that will evaluate all calls against the rules in the block in order. 
The optional parameter to the rule is guard, you can also return null inside a rule to ignore that rule

See the file httpMock.kt for more details. 

There is a convenience method `HttpMock.clearAllHttpMocks()` for clearing up all mocks created with the DSL.

It is also possible to initialize a `MockWebServer` that is not started, where you can add rules after it is created.
If you need to stop the server and clear the HttpMocks, for instance to avoid it bleeding into other tests,
use the `executeRulesAndClearMocks` function or manually call `HttpMock.clearAllHttpMocks()`.

```
val httpMock = initHttpMockServer {
    rule({ path?.endsWith("sith")}) {
        MockResponse().setBody("Darth Vader")
    }
}
httpMock.rule({ path?.endsWith("jedi") }) {
    MockResponse().setBody("Yoda")
}

httpMock.executeRules {
    val response1 = RestTemplate().getForEntity<String>("${it.url}/jedi")
    val response2 = RestTemplate().getForEntity<String>("${it.url}/sith")
    ...
}
```

## Custom ObjectMapper

In certain test cases it is useful to send in a custom `ObjectMapper` instance. This can either be done on individual functions:
```
server.execute(201 to TestObject("test"), objectMapper = jacksonObjectMapper()) { ... }
```

Or by setting the `ObjectMapper` before the actual unit tests run: 
```
init {
    TestObjectMapperConfigurer.objectMapper = customObjectMapper()
}

@AfterAll
fun tearDown() {
    TestObjectMapperConfigurer.reset()
}
```


## WireMock

For information on how to setup the contract consumer see the [Spring Cloud Contract documentation](https://cloud.spring.io/spring-cloud-contract/spring-cloud-contract.html#_client_side)

### Build script

The build script must package the stub-jar file,
for more details take a look at [this sample from Spring Cloud Contract](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/blob/master/producer_with_restdocs/build.gradle#L83)


