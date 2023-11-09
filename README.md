# ECE 382V Root Causes of NOD Tests

# Backaground and Context

In this report, I will cover the project regarding the root cause of NOD flaky tests along with its specific details, implementation, and examples of what to expect. This project is being completed in the class ECE 382V in Fall 2023 under Dr. August Shi. 

The project I am working on is to find the root cause of NOD flaky tests. While researchers have been able to find NOD tests through automated techniques that involve rerunning tests many times as well as compiling them into large datasets like Idoft and FlakeFlagger, the root cause of these tests is still unknown. More specifically, the tests that are a part of the large datasets mentioned are known to be flaky, but the underlying reasoning is not clear for why that is. 

To solve this question, I will be conducting an empirical study that looks into these flaky tests and provides reasons and root causes for flakiness. Throughout my research, I have found five main categories into which these NOD flaky tests fall: Randomness, Network, Concurrency, Timing, and Not Specified. I will clarify these categories and provide a few examples below.

# Randomness

The category randomness is relative to NOD flaky tests and has been defined to mean a lack of promised order in which iterations or operations can happen. In other words, when looking at different executions of the same code, we expect an iteration or operation to have the same order; randomness in NOD flaky tests takes this away and introduces randomness in these iterations and operations from execution to execution. One example of where randomness can become a problem within NOD flaky tests is when a data structure has a randomized iteration order, which can create a different structure listing for each execution. This example is shown and explained below.

```bash
private final Random random = new Random();
    @Test
    public void testRandomNumberIsAlwaysEven() {
        int randomNumber = random.nextInt();
        assertTrue(randomNumber % 2 == 0);  
    }
```
The above test is a NOD flaky test that exhibits randomness. The reason is because of the non-deterministic ordering of the elements within the ConcurrentHashMap when the toString() method is called. Since the order of the elements within that data structure is not guaranteed, converting it to a String will always give it different outputs, leading to the NOD flaky reason being randomness.

# Network

The category network relative to NOD flaky tests has been defined to mean a flaky test that depends on unpredictable network connections or any unreliable external web services. To be more precise, network issues in an NOD flaky test mean that the test in particular has some dependence on an external network that is not guaranteed to always work. Since the question test depends on the specified unstable network, this allows for it to become flaky due to network issues. An example of this is when a test makes a call to an unstable API. Since the API won't always work properly, this leads to different results for each execution. An example is shown and explained below.

```bash
@Test
    public void whenCallingApi_thenSuccess() throws IOException {
        URL url = new URL("http://api.example.com/resource");
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("GET");
        connection.connect();

        int responseCode = connection.getResponseCode();
        assertTrue(responseCode == 200, "API did not respond with 200 OK");
    }
```
The test above is an example of a NOD test that is flaky due to network errors. The reason is that there is a GET request to an external API and asserting that the response code is HTTP 200 (OK). With that in mind though, network flakiness will occur if there are timeouts or network connectivity problems, which is assumed to happen in this case.

# Concurrency

The category concurrency relative to NOD flaky tests has been defined to mean tests that deal with different types of race conditions and deadlocks. These factors can cause lots of problems as race conditions and deadlock can result in high levels of wait time and unwanted modification of variables between executions. An example could be where a static variable is being accessed concurrently in two different threads. This opens the potential for some executions to have different variable values compared to other executions, creating flakiness. An example is shown below.

```bash
private static int sharedCounter = 0;

    @Test
    public void testIncrementCounter() {
        sharedCounter++;
        assertEquals(1, sharedCounter);
    }

    @Test
    public void anotherTestThatIncrementsCounter() {
        sharedCounter++;
        assertEquals(1, sharedCounter);
    }
```

In the example shown, the tests are comparing the value of the static variable sharedCounter to a constant 1. The issue of concurrency arises though from the fact that two different threads are accessing the static variable. Doing this opens the possibility of concurrency issues happening, which will then make the test flaky because of a concurrency problem.

# Timing

The category timing is a category that can have multiple meanings. The context in which timing will be used in this repository will be about asynchronous timing. In other words, when a flaky test or source code has problems with timing, that means that there is a dependency on the asynchronous function to work properly, or have consistent timing. The timing, however, will not always be the same and so a deviation from what is expected can allow the test to pass or fail, creating flakiness. An example is when there is a specific timeout that is given and there is also a dependency on a asynchronous operation. As stated before though, the asynchronous operation will not always take consistent time, creating an unstable pass or fail for the test. An example of this is shown below.

```bash
@Test(timeout = 5000)
  public void testHandshakeRejectionTestCase10() throws Exception {
    testProtocolRejection(10, new Draft_6455(Collections.<IExtension>emptyList(),
        Collections.<IProtocol>singletonList(new Protocol("chat"))));
  }
```

In this example, the timeout is set to 5000 milliseconds, but the testProtocolRejection is an asynchronous operation that takes a variable amount of time depending on the execution. In other words, the time could be longer or shorter than the specified timeout, so in the times when the time is greater than the specified timeout, the test will fail, creating flakiness.

# Not Specified

The category of not specified is a category that defines a test to not have a reason for flaky failure that is categorized within the four categories of network failure, randomness, timing, and concurrency. I had this category to show that all the tests will not fall into these four categories and although they make up a large majority of the tests, some tests can fail from events like an environment change needed. An example is shown below.

```bash
@BeforeClass
    public static void setUpClass() throws IOException {
        FileUtils.delete(new File("target/derbydb"));
        FileUtils.delete(new File("target/lucene3"));
        AnnotationConfiguration cfg = new AnnotationConfiguration();
        cfg.addAnnotatedClass(User.class);
        Properties props = new Properties();
```
The issue in this test is that at the new File creation of lucene3, the correct file that needs to have been made is lucene. There is the assumption that the interactions will always be with lucene3 but there needs to be a clean-up of the files so that there is a creation to the correct directory. This is an example of a flaky test that does not directly fall into the categories that are mentioned.

# Methodology for Finding NOD Flaky Tests

I wanted to provide the method that I am using to find tests through the Idoft repository. There are multiple ways that this can be done and I am finding tests through a specific approach. The way that I first started was to start at the top of Idoft and see what NOD flaky tests exist. From there, I will continue to find repositories present and classify the NOD tests given. If I find 5 tests in a row that are of a certain category (concurrency, timing, randomness, network, or not specified), I will then move 10 repositories forward in idoft and start there again. I believe that this allows for a proper inclusion of all different types of tests. If the scenario of me getting the same type of test again even after I move, then I will move 10 repositions forward again and repeat the process. I still keep track of where I left off for documentation purposes, but I have found this to allow for a high range of NOD flaky tests that are of different categories. Another methodology I follow is that if I am within one project for 10 flaky tests, I will go to the next repository as I want to follow a mix of a breadth-first and depth-first search. Doing this allows me to cover multiple repositories without just deep diving into one.
