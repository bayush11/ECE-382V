| Fully Qualified Test Name | Project Name | SHA | Lines that cause failure | Root Cause | Source or Test Code | Why is the test flaky? | Full Line in Idoft |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| org.platformlambda.automation.tests.WebSocketTests.userChannelTest | https://github.com/Accenture/mercury | 6b744cdbb2206feca62848df92b3bf542f890be5 | 129, 130, 131 | Asynchronous Timing | Source | The reason why there was an asynchronous timing error at these three lines was that in the original snippet of code, the get() function was invoked immediately after the asynchronous BlockingQueue.poll function was called. The .poll function's time could depend on many factors, and so by checking right after the .poll function was called, there was not enough time given for this function to complete. In doing this, the .get() call would non-deterministically fail. The fix comes from giving the asynchronous call enough time to finish before checking the values of the .get() function. |





