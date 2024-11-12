#	$${\color{green}Language \space Integrated \space Query \space (LINQ)}$$
## Definitions
A set of technologies based on the integration of query capabilities directly into the C# language. With LINQ, a query is a first-class language construct, just like classes, methods, and events.

A query is an expression that retrieves data from a data source. Different data sources have different native query languages, for example SQL for relational databases and XQuery for XML. Developers must learn a new query language for each type of data source or data format that they must support. LINQ simplifies this situation by offering a consistent C# language model for kinds of data sources and formats. In a LINQ query, you always work with C# objects.

In LINQ, a query variable is any variable that stores a query instead of the results of a query. And always an enumerable type.

All LINQ query operations consist of three distinct actions:

  1. Obtain the data source.
  2. Create the query.
  3. Execute the query.

  ```c#
  using System.Linq;

  // The Three Parts of a LINQ Query:
  // 1. Data source.
  int[] numbers = [ 5, 10, 8, 3, 6, 12 ];

  // 2. Query creation.
  // numQuery is an IEnumerable<int>
  //Query syntax:
  IEnumerable<int> numQuery1 = //query variable, NOT stores no actual result data
      from num in numbers //required
      where num % 2 == 0 // optional
      orderby num descending // optional
      select num; //must end with select or group

  //Method syntax:
  IEnumerable<int> numQuery2 = //query variable, NOT stores no actual result data
      numbers.Where(num => num % 2 == 0).OrderBy(n => n);

  // 3. Query execution. Execute the query to produce the results
  foreach (int i in numQuery1)
  {
      Console.Write(i + " ");
  }
  Console.WriteLine(System.Environment.NewLine);
  foreach (int i in numQuery2)
  {
      Console.Write(i + " ");
  }
  ```

## Ways to write LINQ queries:
  <details>
  <summary>1. Query syntax</summary>
  Between the starting <code>from</code> clause, and the ending <code>select</code> or <code>group</code> clause, all other clauses (<code>where</code>, <code>join</code>, <code>orderby</code>, <code>from</code>, <code>let</code>) are optional
  
### Dynamic filter
  ```c#
  using System.Linq;

  int[] ids = [111, 114, 112];

  var queryNames =
      from student in students
      where ids.Contains(student.ID) //filter
      select new
      {
          student.LastName,
          student.ID
      };

  foreach (var name in queryNames)
  {
      Console.WriteLine($"{name.LastName}: {name.ID}");
  }
  ```

  ```c#
  IEnumerable<Student> students =
[
    new Student(First: "Svetlana", Last: "Omelchenko", ID: 111, Scores: [97, 92, 81, 60]),
    new Student(First: "Claire",   Last: "O'Donnell",  ID: 112, Scores: [75, 84, 91, 39])
];
    // The first line could also be written as "var studentQuery ="
IEnumerable<Student> studentQuery =
    from student in students
    where student.Scores[0] > 90 && student.Scores[3] < 80  
    orderby student.Last ascending
    select student;

  ```

### Group the results
A query with a group clause produces a sequence of groups, and each group itself contains a **Key** and a sequence that consists of all the members of that group. 
  ```c#
  IEnumerable<IGrouping<char, Student>> studentQuery =
    from student in students
    group student by student.Last[0];

foreach (IGrouping<char, Student> studentGroup in studentQuery)
{
    Console.WriteLine(studentGroup.Key);
    foreach (Student student in studentGroup)
    {
        Console.WriteLine($"   {student.Last}, {student.First}");
    }
}
  ```

### Order the groups by their key value
Provide an **orderby** clause after the **group** clause. But need an identifier that serves as a reference to the groups created by the group clause. You provide the identifier by using the **into** keyword
  ```c#
var studentQuery4 =
    from student in students
    group student by student.Last[0] into studentGroup
    orderby studentGroup.Key
    select studentGroup;

foreach (var groupOfStudents in studentQuery4)
{
    Console.WriteLine(groupOfStudents.Key);
    foreach (var student in groupOfStudents)
    {
        Console.WriteLine($"   {student.Last}, {student.First}");
    }
}
  ```

### Complex condition
Use the let keyword to introduce an identifier for any expression result in the query expression
  ```c#
var studentQuery5 =
    from student in students
    let totalScore = student.Scores[0] + student.Scores[1] +
        student.Scores[2] + student.Scores[3]
    where totalScore / 4 < student.Scores[0]
    select $"{student.Last}, {student.First}";

foreach (string s in studentQuery5)
{
    Console.WriteLine(s);
}
  ```

### Handle null values
  ```c#
  var query1 =
    from c in categories 
    where c != null // filters out all null elements
    join p in products on c.ID equals p?.CategoryID //Products.CategoryID is of type int?, which is shorthand for Nullable<int>.
    select new
    {
        Category = c.Name,
        Name = p.Name
    };
  ```

  </details>
  <details>
  <summary>  2. Method syntax</summary>

  <b> Expression lambda </b>

  <code>(input-parameters) => expression</code>
  
  ```c#
Action line = () => Console.WriteLine(); //zero input parameters with empty ()

Func<double, double> cube = x => x * x * x; //only one input parameter, () are optional

Func<int, int, bool> testForEquality = (x, y) => x == y; //Two or more input parameters are separated by commas

var IncrementBy = (int source, int increment = 1) => source + increment;
Console.WriteLine(IncrementBy(5)); // 6
Console.WriteLine(IncrementBy(5, 2)); // 7
```
  
  <b> Statement lambda </b>

  <code>(input-parameters) => { < sequence-of-statements> }</code>
  
  ```c#
var sum = (params IEnumerable<int> values) =>
{
    int sum = 0;
    foreach (var value in values) 
        sum += value;
    
    return sum;
};

var empty = sum();
Console.WriteLine(empty); // 0

var sequence = new[] { 1, 2, 3, 4, 5 };
var total = sum(sequence);
Console.WriteLine(total); // 15
```
  
  <b> Query Expression </b>

Refer https://learn.microsoft.com/en-us/dotnet/csharp/linq/standard-query-operators/?redirectedfrom=MSDN#query-expression-syntax-table
  ```c#
string sentence = "the quick brown fox jumps over the lazy dog";
// Split the string into individual words to create a collection.
string[] words = sentence.Split(' ');

// Using query expression syntax.
var query = from word in words
            group word.ToUpper() by word.Length into gr
            orderby gr.Key
            select new { Length = gr.Key, Words = gr };

// Using method-based query syntax.
var query2 = words.
    GroupBy(w => w.Length, w => w.ToUpper()).
    Select(g => new { Length = g.Key, Words = g }).
    OrderBy(o => o.Length);

foreach (var obj in query)
{
    Console.WriteLine("Words of length {0}:", obj.Length);
    foreach (string word in obj.Words)
        Console.WriteLine(word);
}
```
    
  </details>
  <details>
  <summary> 3. Mixed query and method syntax</summary>

  ```c#
  var numCount1 = (
    from num in numbers1
    where num is > 3 and < 7
    select num
).Count();
```
  
  </details>

# $${\color{green}Asynchronous \space programming}$$
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
