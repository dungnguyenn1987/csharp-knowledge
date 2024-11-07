# Asynchronous programming
## Why we need
Cooking breakfast is a good example of asynchronous work that isn't parallel.For example, The synchronously prepared breakfast took roughly 30 minutes because the total is the sum of each task.

![image](https://github.com/user-attachments/assets/0f2876a3-19a9-484d-aa8a-06cd5f16e815)

If you have experience with cooking, you'd execute those instructions asynchronously and it took roughly 20 minutes, this time savings is because some tasks ran concurrently.
![image](https://github.com/user-attachments/assets/a6aa6209-a68a-4c5a-b32f-08a98280ea2f)

## Asynchronous Process
![image](https://github.com/user-attachments/assets/0682e1b7-af0d-4733-94e2-217af0ed48a9)

<details>
<summary>Explaination</summary>
The numbers in the diagram correspond to the following steps, initiated when a calling method calls the async method.

1. A calling method calls and awaits the GetUrlContentLengthAsync async method.

2. GetUrlContentLengthAsync creates an HttpClient instance and calls the GetStringAsync asynchronous method to download the contents of a website as a string.

3. Something happens in GetStringAsync that suspends its progress. Perhaps it must wait for a website to download or some other blocking activity. To avoid blocking resources, GetStringAsync yields control to its caller, GetUrlContentLengthAsync.

GetStringAsync returns a Task<TResult>, where TResult is a string, and GetUrlContentLengthAsync assigns the task to the getStringTask variable. The task represents the ongoing process for the call to GetStringAsync, with a commitment to produce an actual string value when the work is complete.

4. Because getStringTask hasn't been awaited yet, GetUrlContentLengthAsync can continue with other work that doesn't depend on the final result from GetStringAsync. That work is represented by a call to the synchronous method DoIndependentWork.

5. DoIndependentWork is a synchronous method that does its work and returns to its caller.

6. GetUrlContentLengthAsync has run out of work that it can do without a result from getStringTask. GetUrlContentLengthAsync next wants to calculate and return the length of the downloaded string, but the method can't calculate that value until the method has the string.

Therefore, GetUrlContentLengthAsync uses an await operator to suspend its progress and to yield control to the method that called GetUrlContentLengthAsync. GetUrlContentLengthAsync returns a Task<int> to the caller. The task represents a promise to produce an integer result that's the length of the downloaded string.
Inside the calling method the processing pattern continues. The caller might do other work that doesn't depend on the result from GetUrlContentLengthAsync before awaiting that result, or the caller might await immediately. The calling method is waiting for GetUrlContentLengthAsync, and GetUrlContentLengthAsync is waiting for GetStringAsync.

7. GetStringAsync completes and produces a string result. The string result isn't returned by the call to GetStringAsync in the way that you might expect. (Remember that the method already returned a task in step 3.) Instead, the string result is stored in the task that represents the completion of the method, getStringTask. The await operator retrieves the result from getStringTask. The assignment statement assigns the retrieved result to contents.

8. When GetUrlContentLengthAsync has the string result, the method can calculate the length of the string. Then the work of GetUrlContentLengthAsync is also complete, and the waiting event handler can resume. In the full example at the end of the topic, you can confirm that the event handler retrieves and prints the value of the length result. If you are new to asynchronous programming, take a minute to consider the difference between synchronous and asynchronous behavior. A synchronous method returns when its work is complete (step 5), but an async method returns a task value when its work is suspended (steps 3 and 6). When the async method eventually completes its work, the task is marked as completed and the result, if any, is stored in the task.
</details>


## async and await


* ``async`` modifier to specify that a method, lambda expression, or anonymous method is asynchronous.
* An ``async`` method runs **synchronously** until it reaches its first ``await`` expression, at which point the method is suspended until the awaited task is complete. In the meantime, control returns to the caller of the method

```c#
static async Task Main(string[] args)
{
    Coffee cup = PourCoffee();
    Console.WriteLine("coffee is ready");

    // 3 tasks run asynchronously
    var eggsTask = FryEggsAsync(2);
    var baconTask = FryBaconAsync(3);
    var toastTask = MakeToastWithButterAndJamAsync(2);
    
    Juice oj = PourOJ();
    Console.WriteLine("oj is ready");

    var eggs = await eggsTask;
    Console.WriteLine("eggs are ready");

    var bacon = await baconTask;
    Console.WriteLine("bacon is ready");

    var toast = await toastTask;
    Console.WriteLine("toast is ready");

    Console.WriteLine("Breakfast is ready!");
}
```
