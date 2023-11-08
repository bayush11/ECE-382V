# ECE 382V Root Causes of NOD Tests

# Backaground and Context

In this report, I will cover the project regarding the root cause of NOD flaky tests along with its specific details, implementation, and examples of what to expect. This project is being completed in the class ECE 382V in Fall 2023 under Dr. August Shi. 

The project I am working on is to find the root cause of NOD flaky tests. While researchers have been able to find NOD tests through automated techniques that involve rerunning tests many times as well as compiling them into large datasets like Idoft and FlakeFlagger, the root cause of these tests is still unknown. More specifically, the tests that are a part of the large datasets mentioned are known to be flaky, but the underlying reasoning is not clear for why that is. 

To solve this question, I will be conducting an empirical study that looks into these flaky tests and provides reasons and root causes for flakiness. Throughout my research, I have found five main categories into which these NOD flaky tests fall: Randomness, Network, Concurrency, Timing, and Not Specified. I will clarify these categories and provide a few examples below.

# Randomness

The category randomness relative to NOD flaky tests has been defined to mean a lack of promised order in which iterations or operations can happen. In other words, when looking at different executions of the same code, we expect an iteration or operation to have the same order; randomness in NOD flaky tests takes this away and introduces randomness in these iterations and operations from execution to execution. One example is a data structure that has a randomized iteration order. Another example is shown and explained below.

```bash
@Test
    public void testToStringShouldPrintMessageAndAllKeyAndValuePairs() {
        logMessage.setMsg(MESSAGE);
        logMessage.kv("key1", "value1");
        logMessage.kv("key2", "value2");
        String output = logMessage.toString();
        assertTrue(output.equals("messagekey1=value1||key2=value2") || output.equals("messagekey2=value2||key1=value1"));
    }
```
The above test is a NOD flaky test that exibhits randomness. The reason being is because of the non-deterministic ordering of the elements within the ConcurrentHashMap when the toString() method is called. Since the order of the elemnets within that data structure is not garunteed, converting it to a String will alwasy give it different outputs, leading to the NOD flaky resaon being randomness.
