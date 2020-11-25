# Java Testing

## Overview

### Why?
* improve quality
* prove that the code do what you think
* industry best practice

### Terminology

* **Unit Tests / Unit Testing**: code written to test "code under test".

* **Integration Tests**: test behaviors between objects and parts of the overall system.

* **Functional Tests**: testing the running application.

### Agile Testing Methods

* **TDD** Test Driven Development: write tests first, code to fix tests, refactor code to cleanup, improve ...

* **BDD** Behavior Driven Development: similar to TDD, describe the expected behavior of software.

* **Which is better?** use both / depending on what you do.

### Java Testing Frameworks

* **JUnit** the most popular testing framework for java, JUnit4 widely used, JUnit5 released in Sep. 2017

* **TestNG** functionality very close to JUnit.

* **Spock** follows BDD approach, includes own mocking framework.

* **Cucumber** BDD testing framework, using Gherkin syntax (like natural English)

* **Mockito** mocking framework for testing, only does mocks, need to use with testing framework (JUnit or TestNG)

* **Spring MVC Test** provides mock servlet environment, used in conjunction with a testing framework (JUnit, TestNG, Spock)

* **REST Assured** java framework for testing RESTful web services, powerful DSL, used with Spring Mock MVC, follow BDD syntax

* **Selenium** web browser Automation, functional tests against web apps, only web browser Automation need testing framework (JUnit, TestNG, Spock)

* **GEB** Groovy Browser Automation, uses Selenium under the cover, need a Test framework, popular with Spock.

* **Test Containers** launch Docker containers from JUnit Tests, allow message brokers, for integration and functional tests, combined with Selenium for testing web apps.


### Testing with CI/CD

* **Continuous Integration (CI)**: is a development practice that require developers to integrate code into a shared repository several times a day. Each check-in is then verified by an automated build, allowing teams to detect problems early.
  * **Tools**:
      * **Slef-Hosted**: Jenkins, Bamboo, TeamCity, Hudson
      * **Cloud Based**: CircleCI, TravisCI, Codeship, GitLab CI, AWS CodeBuild

* **Continuous Development (CD)**: will automatically deploy build artifacts after all CI tests have run with every commit / completely automated.
  * **Tools**: AWS CodePipeline

* **Continuous Delivery (CD)**: process to automatically deliver code changes directly to the production environment, High degree of Automation Testing and Deployment / must have a very mature process.
  * Difficult in some industries due to regulatory requirements.


### Test Driven Development
Book: "Test-Driven Development by Examply- Kent Beck" 2002

* **TDD Cycle** Goal "Clean Code that works." - Ron Jeffries
  1. write a Test
  2. make it Run
  3. make it Right


## Spring Boot

### Test HelloController

        //@RunWith(SpringRunner.class)      //JUnit4
        @ExtendWith(MockitoExtension.class) //JUnit5
        @WebMvcTest(HelloController.class)
        class HelloControllerTest {

            @Autowired
            private MockMvc mockMvc;

            @Test
            public void hello() throws Exception {
                RequestBuilder request = MockMvcRequestBuilders
                        .get("/hello")
                        .accept(MediaType.APPLICATION_JSON);
                MvcResult result = mockMvc.perform(request).andReturn();
                String response = result.getResponse().getContentAsString();

                assertEquals("Hello Testing!", response);
            }
        }

        @RestController
        public class HelloController {

            @GetMapping("/hello")
            public String hello() {
                return "Hello Testing!";
            }
        }

###





