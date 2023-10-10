
# FmgLib.Orm.DbHelper

DbHelper is a micro ORM library for .NET. It supports various database types such as MySql, PostgreSql, Oracle, SqlServer, SQLite. It provides support for database creation using the First Code approach, but this is only valid for Oracle databases. DbHelper provides multiple database support, allowing you to easily apply the same query structure to different database types.

You can access the [FmgLib.Orm.DbHelper](https://www.nuget.org/packages?q=FmgLib.Orm) NuGet package through the link.

To include the DbHelper library in your project, you need to download the relevant library from NuGet for the database you will be working with. For example:
`FmgLib.Orm.DbHelper.MySql`,
`FmgLib.Orm.DbHelper.PostgreSql`,
`FmgLib.Orm.DbHelper.SqlServer`,
`FmgLib.Orm.DbHelper.SQLite`,
`FmgLib.Orm.DbHelper.Oracle`
You can include the latest version of the library in your project by downloading the appropriate one.

First, let's create a model for the table on the database side (regardless of the First Code approach):

```csharp
using FmgLib.Orm.Common;

namespace FmgLib.TestWeb.Models;

[Table("PRODUCTS")]
public class Product : IDbEntity
{
    [Column(IsNotNull = true, IsPrimaryKey = true, IsAutoIncrement = true)]
    public int Id { get; set; }

    [Column(IsNotNull = true)]
    public string ProductName { get; set; }

    [Column(IsNotNull = false)]
    public double Price { get; set; }

    public int? CategoryId { get; set; }
}
```
Here are some details:
-  You can specify a custom name for the table to be created on the database side by using the `Table("TABLE_NAME")` attribute on the class. If not provided, the default name will be the class name.
-  You can customize the table on the property level using the `Column([bool IsNotNull = false], [bool IsPrimaryKey = false], [bool IsAutoIncrement = false])` attribute. **IsNotNull** controls the nullable property, and properties like primary key and auto-increment are controlled by this attribute. You don't need to use the `Column()` attribute specifically to check if a column is nullable. You can make a column nullable by adding the **?** character after the value type. 
-  The **most important thing** here is that the model must derive from the **`IDbEntity`** type; otherwise, the model will not be created on the database side.

Now, let's integrate the library into your project. To do this, you need to make the necessary integrations in the `Program.cs` file:
```csharp
using FmgLib.Orm.DbHelper.MySql;
using FmgLib.Orm.DbHelper.PostgreSql;
using FmgLib.Orm.DbHelper.SQLite;
using FmgLib.Orm.DbHelper.SqlServer;
using FmgLib.Orm.DbHelper.Oracle;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

await builder.Services.DbIntegrationMySqlAsync(builder.Configuration.GetConnectionString("MYSQL")); // OR
// builder.Services.DbIntegrationMySql(builder.Configuration.GetConnectionString("MYSQL")); // Asynchronous and synchronous usage is up to your preference.

await builder.Services.DbIntegrationPostgreSqlAsync(builder.Configuration.GetConnectionString("POSTGRESQL")); // OR
// builder.Services.DbIntegrationPostgreSql(builder.Configuration.GetConnectionString("POSTGRESQL")); // Asynchronous and synchronous usage is up to your preference.

await builder.Services.DbIntegrationSQLiteAsync(builder.Configuration.GetConnectionString("SQLITE")); // OR
// builder.Services.DbIntegrationSQLite(builder.Configuration.GetConnectionString("SQLITE")); // Asynchronous and synchronous usage is up to your preference.

await builder.Services.DbIntegrationSqlServerAsync(builder.Configuration.GetConnectionString("SQLSERVER")); // OR
// builder.Services.DbIntegrationSqlServer(builder.Configuration.GetConnectionString("SQLSERVER")); // Asynchronous and synchronous usage is up to your preference.

await builder.Services.DbIntegrationOracleAsync(builder.Configuration.GetConnectionString("ORACLE")); // OR
// builder.Services.DbIntegrationOracle(builder.Configuration.GetConnectionString("ORACLE")); // Asynchronous and synchronous usage is up to your preference.

var app = builder.Build();
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

The `DbEntegrationMySql(CONN_STR)`, `DbEntegrationPostgreSql(CONN_STR)`, `DbEntegrationSqlServer(CONN_STR)`, `DbEntegrationSQLite(CONN_STR)`, `DbEntegrationOracle(CONN_STR)` functions integrate the database and create the models as tables on the database side. When you add a new model, running the program again triggers these functions, automatically creating new tables in the database. The integration part can be completed in this simple way.

If we look at some example usages:
First, let's consider Query and Command in two parts, and then we'll show how they vary according to the database types.

-  For Query:

You can create a query using `Query.GetQuery<TEntity>()`. You can apply special conditions to the query afterward.

```csharp
using FmgLib.Orm.DbHelper;

var query = Query.GetQuery<Product>() // Query created.
                .Where(x => x.Price > 250 && x.ProductName.Contains("Dene")) // Filtering applied.
                .DistinctBy(x => x.Price) // Adding a column for DISTINCT.
                .DistinctBy(x => x.ProductName) // Adding a column for DISTINCT.
                .Select(x => x.CategoryId) // Adding a column for SELECT.
                .OrderBy(x => x.Id) // Adding sorting (ascending order).
                .OrderByDescending(x => x.Price); // Adding sorting (descending order).
```

Due to some rules in SQL, all queries created here are filtered and applied to adhere to SQL query logic. For example, if you want to get the Max value or the total value of a specific column, using **ORDER BY** would be incorrect. The SqlBuilder layer handles the necessary operations to ensure that your query works correctly without breaking the logic.

After creating the query, it's up to you to send requests to which database. You can continue the code as follows:
```csharp
using FmgLib.Orm.DbHelper.MySql;
using FmgLib.Orm.DbHelper.PostgreSql;
using FmgLib.Orm.DbHelper.SQLite;
using FmgLib.Orm.DbHelper.SqlServer;
using FmgLib.Orm.DbHelper.Oracle;

/*
var queryMySql = query.AsQueryMySql(); // Converted to a MySql query.
var queryPostgreSql = query.AsQueryPostgreSql(); // Converted to a PostgreSql query.
var querySqlServer = query.AsQuerySqlServer(); // Converted to a SqlServer query.
var querySQLite = query.AsQuerySQLite(); // Converted to an SQLite query.
var queryOracle = query.AsQueryOracle(); // Converted to an Oracle query.
*/

// From here on, we will continue with SqlServer. The remaining steps are the same for other database types. You only need to call the relevant method instead of AsQueryMySql(). The methods are listed above.
// AsQuerySqlServer() -> SqlServer
// AsQuerySQLite() -> SQLite
// AsQueryPostgreSql() -> PostgreSql
// AsQueryOracle() -> Oracle
// AsQueryMySql() -> MySql

var qqqq = query.AsQuerySqlServer().First(); // Get the first record.
var wwww = query.AsQuerySqlServer().First(x => x.Id > 10); // Get the first record with an additional filter.
var eeee = query.AsQuerySqlServer().FirstOrDefault(); // Get the first record or create a default one if it doesn't exist.
var rrrr = query.AsQuerySqlServer().FirstOrDefault(x => x.Id > 10); // Get the first record with an additional filter or create a default one if it doesn't exist.
var tttt = query.AsQuerySqlServer().ElementAt(5); // Get the record at index 5.
var yyyy = query.AsQuerySqlServer().ElementAtOrDefault(5); // Get the record at index 5 or create a default one if it doesn't exist.
var uuuu = query.AsQuerySqlServer().Min(x => x.Price); // Get the minimum value in the specified column.
var cccc = query.AsQuerySqlServer().MinBy(x => x.Price); // Get the record with the minimum value in the specified column.
var oooo = query.AsQuerySqlServer().Max(x => x.Price); // Get the maximum value in the specified column.
var pppp = query.AsQuerySqlServer().MaxBy(x => x.Price); // Get the record with the maximum value in the specified column.
var llll = query.AsQuerySqlServer().Sum(x => x.Price); // Get the sum of values in the specified column.
var zzzz = query.AsQuerySqlServer().Average(x => x.Price); // Get the average of values in the specified column.
var aaaa = query.AsQuerySqlServer().Count(); // Get the count of records.
var ssss = query.AsQuerySqlServer().Count(x => x.Price < 600); // Get the count of records with an additional filter.
var dddd = query.AsQuerySqlServer().Any(); // Check if there are any records (true/false).
var ffff = query.AsQuerySqlServer().Any(x => x.Price < 600); // Check if there are any records with an additional filter (true/false).
var gggg = query.AsQuerySqlServer().Skip(20); // Get the remaining data by skipping 20 records.
var hhhh = query.AsQuerySqlServer().Take(20); // Get the first 20 records.
var xxxx = query.AsQuerySqlServer().SkipAndTake(10, 20); // Get 20 records starting from the 10th record.
var kkkk = query.AsQuerySqlServer().ToList(); // Get records as a list.
var xxxx = query.AsQuerySqlServer().ToArray(); // Get records as an array.
var vvvv = query.AsQuerySqlServer().ToQuery(); // Get records as IQueryable.
var bbbb = query.AsQuerySqlServer().ToQuery("SELECT * FROM PRODUCTS ORDER BY Price DESC"); // Get the result of the SQL query provided as a parameter, independent of previous queries, as IQueryable.

// Async versions of these methods are also available.
```
-  For Command:
```csharp

// The example continues with SqlServer. For other databases, you can use the relevant method instead of AsCommandSqlServer(). There are no differences in the remaining processes.
// AsCommandSqlServer() -> SqlServer
// AsCommandSQLite() -> SQLite
// AsCommandPostgreSql() -> PostgreSql
// AsCommandOracle() -> Oracle
// AsCommandMySql() -> MySql

var insertr = Command
                .Insert(new Product
                {
                    ProductName = "One item insert test",
                    Price = 1234,
                    CategoryId = 234
                }) // Create an insert command.
                .AsCommandSqlServer() // Transfer the command to SqlServer.
                .Save(); // Execute the command on the database. Returns an int value.

var insertrMulti = Command
                    .InsertRange(/*products list*/) // Create multiple insert commands by sending a list.
                    .AsCommandSqlServer() // Transfer the command to SqlServer.
                    .Save(); // Execute the command on the database. Returns an int value.

var updater = Command
                .Update(new Product
                {
                    ProductName = "222 TEST DENEME 22",
                    Price = 6134,
                    CategoryId = 155
                }, x => x.Id == 102) // Create an update command with a filter.
                .AsCommandSqlServer() // Transfer the command to SqlServer.
                .Save(); // Execute the command on the database. Returns an int value.

var deleter = Command
                .Delete<Product>(x => x.Id == 102) // Create a delete command with a filter.
                .AsCommandSqlServer() // Transfer the command to SqlServer.
                .Save(); // Execute the command on the database. Returns an int value.

// There is also an asynchronous version of the Save() method: SaveAsync().
```
