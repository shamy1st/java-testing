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

### Controller Unit Test

        @RestController
        public class ItemController {

            @Autowired
            private ItemService service;

            @GetMapping("/items-list")
            public List<Item> getItems() {
                return service.getItems();
            }
        }

        @ExtendWith(MockitoExtension.class)
        @WebMvcTest(ItemController.class)
        class ItemControllerTest {

            @MockBean
            private ItemService service;

            @Autowired
            private MockMvc mockMvc;

            void test(List<Item> list, String expected) throws Exception {
                when(service.getItems()).thenReturn(list);

                RequestBuilder request = MockMvcRequestBuilders
                        .get("/items-list")
                        .accept(MediaType.APPLICATION_JSON);

                MvcResult result = mockMvc.perform(request)
                        .andExpect(status().isOk())
                        .andExpect(content().json(expected))
                        .andReturn();
            }

            @Test
            void getItems() throws Exception {
                List<Item> list = new ArrayList<>();
                list.add(new Item(10001, "item1", 10.5f, 20));
                list.add(new Item(10002, "item2", 14.6f, 100));
                list.add(new Item(10003, "item3", 6.4f, 150));

                String expected = "[{id:10001,name:item1,price:10.5,quantity:20}" +
                        ",{id:10002,name:item2,price:14.6,quantity:100}" +
                        ",{id:10003,name:item3,price:6.4,quantity:150}]";

                test(list, expected);
            }
        }

### Service Unit Test

        @Service
        public class ItemService {

            @Autowired
            private ItemRepository repository;

            public List<Item> getItems() {
                return repository.findAll();
            }
        }

        @ExtendWith(MockitoExtension.class)
        class ItemServiceTest {

            @InjectMocks
            private ItemService service;

            @Mock
            private ItemRepository repository;

            void test(List<Item> list) throws Exception {
                when(repository.findAll()).thenReturn(list);
                List<Item> actualList = service.getItems();
                assertEquals(actualList, list);
            }

            @Test
            void getItems() throws Exception {
                List<Item> list = new ArrayList<>();
                list.add(new Item(10001, "item1", 10.5f, 20));
                list.add(new Item(10002, "item2", 14.6f, 100));
                list.add(new Item(10003, "item3", 6.4f, 150));
                test(list);
            }
        }

### Repository Unit Test

        public interface ItemRepository extends JpaRepository<Item, Integer> {

        }

        @ExtendWith(MockitoExtension.class)
        @DataJpaTest
        class ItemRepositoryTest {

            @Autowired
            private ItemRepository repository;

            void test(List<Item> list) {
                List<Item> actualList = repository.findAll();
                assertEquals(list, actualList);
            }

            @Test
            public void findAll() {
                List<Item> list = new ArrayList<>();
                list.add(new Item(10001, "item1", 10.5f, 20));
                list.add(new Item(10002, "item2", 14.6f, 100));
                list.add(new Item(10003, "item3", 6.4f, 150));
                test(list);
            }
        }

### Integration Test using @SpringBootTest (in memory database h2)

        @ExtendWith(MockitoExtension.class)
        @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
        class ItemControllerIT {

            @Autowired
            private TestRestTemplate restTemplate;

            @Test
            public void getItems() throws JSONException {
                String response = this.restTemplate.getForObject("/items-list", String.class);

                String expected = "[{id:10001,name:item1,price:10.5,quantity:20}" +
                        ",{id:10002,name:item2,price:14.6,quantity:100}" +
                        ",{id:10003,name:item3,price:6.4,quantity:150}]";

                JSONAssert.assertEquals(expected, response, false);
            }
        }

### Integration Test using MockBean

        @ExtendWith(MockitoExtension.class)
        @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
        class ItemControllerIT {

            @Autowired
            private TestRestTemplate restTemplate;

            @MockBean
            private ItemRepository repository;

            @Test
            public void getItems() throws JSONException {
                List<Item> list = new ArrayList<>();
                list.add(new Item(10001, "item1", 10.5f, 20));
                list.add(new Item(10002, "item2", 14.6f, 100));
                list.add(new Item(10003, "item3", 6.4f, 150));
                when(repository.findAll()).thenReturn(list);

                String response = this.restTemplate.getForObject("/items-list", String.class);

                String expected = "[{id:10001,name:item1,price:10.5,quantity:20}" +
                        ",{id:10002,name:item2,price:14.6,quantity:100}" +
                        ",{id:10003,name:item3,price:6.4,quantity:150}]";

                JSONAssert.assertEquals(expected, response, false);
            }
        }

### application.properties for testing

1. for different properties while running tests, create **resources** directory under **src/test/**
2. then copy the **application.properties** to it, and modify it as you want.

### application.properties for a specific test

under **src/test/** create **test-configuration.properties** then:

        @ExtendWith(MockitoExtension.class)
        @SpringBootTest
        @TestPropertySource(locations = {"classpath:test-configuration.properties"})
        class ItemControllerIT {
            
        }

### Hamcrest

        import static org.hamcrest.MatcherAssert.assertThat;
        import static org.hamcrest.Matchers.*;
        
        public class HamcrestTest {

            @Test
            public void test() {
                List<Integer> list = Arrays.asList(10, 20, 30);
                assertThat(list, hasSize(3));
                assertThat(list, hasItems(10, 20));
                assertThat(list, everyItem(greaterThan(8)));
                assertThat(list, everyItem(lessThan(40)));

                assertThat("ABCDE", containsString("BCD"));
                assertThat("ABCDE", startsWith("ABC"));
                assertThat("ABCDE", endsWith("CDE"));
            }
        }

### AssertJ

        import static org.assertj.core.api.Assertions.assertThat;

        public class AssertJTest {

            @Test
            public void test() {
                List<Integer> list = Arrays.asList(10, 20, 30);

                assertThat(list).hasSize(3)
                                .contains(10, 20)
                                .allMatch(item -> (item > 8) && (item < 40));

                assertThat("ABCDE").contains("BCD")
                                   .startsWith("ABC")
                                   .endsWith("CDE");
            }
        }

### JSONPath

        import static org.assertj.core.api.Assertions.assertThat;

        public class JsonPathTest {

            @Test
            public void test() {
                String response = "[{id:10001,name:item1,price:10.5,quantity:20}" +
                        ",{id:10002,name:item2,price:14.6,quantity:100}" +
                        ",{id:10003,name:item3,price:6.4,quantity:150}]";

                DocumentContext context = JsonPath.parse(response);

                int length = context.read("$.length()");
                assertThat(length).isEqualTo(3);

                List<Integer> ids = context.read("$..id");
                assertThat(ids).containsExactly(10001, 10002, 10003);

                System.out.println(context.read("$.[1]").toString());
                System.out.println(context.read("$.[0:2]").toString());
                System.out.println(context.read("$.[?(@.name=='item3')]").toString());
                System.out.println(context.read("$.[?(@.quantity==100)]").toString());
            }
        }

### xUnit Patterns

http://xunitpatterns.com/

### Test Coverage

Run 'All Tests' with Coverage, this show the coverage reports of your code in percentage and also source code which is covered and which is not.
