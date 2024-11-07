# Language Integrated Query (LINQ)
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

# Ways to write LINQ queries:
  <details>
  <summary>1. Query syntax</summary>
  Between the starting <code>from</code> clause, and the ending <code>select</code> or <code>group</code> clause, all other clauses (<code>where</code>, <code>join</code>, <code>orderby</code>, <code>from</code>, <code>let</code>) are optional
  
## Dynamic filter
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

## Group the results
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

## Order the groups by their key value
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

## Complex condition
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

## Handle null values
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
