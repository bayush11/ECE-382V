# ECE 382V Root Causes of NOD Tests

# Backaground and Context

In this report, I will cover the project regarding the root cause of NOD flaky tests along with its specific details, implementation, and examples of what to expect. This project is being completed in the class ECE 382V in Fall 2023 under Dr. August Shi. 

The project I am working on is to find the root cause of NOD flaky tests. While researchers have been able to find NOD tests through automated techniques that involve rerunning tests many times as well as compiling them into large datasets like Idoft and FlakeFlagger, the root cause of these tests is still unknown. More specifically, the tests that are a part of the large datasets mentioned are known to be flaky, but the underlying reasoning is not clear for why that is. 

To solve this question, I will be conducting an empirical study that looks into these flaky tests and provides reasons and root causes for flakiness. Throughout my research, I have found five main categories into which these NOD flaky tests fall: Randomness, Network, Concurrency, Timing, and Not Specified. I will clarify these categories and provide a few examples below.
