DbHelper is a modern object-relational mapper (O/RM) for .NET. It supports updates, and schema migrations. DbHelper  works with SQL Server, SQLite, MySQL, PostgreSQL, and Oracle through a provider plugin API.

To use it, you first need to download the `FmgLib.Orm.DbHelper.SqlServer` package from Nuget.

You can easily include the package in your application in ***Program.cs***. Example usage is as follows:

```csharp
using FmgLib.Orm.DbHelper.SqlServer;

var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddControllersWithViews();

await builder.Services.DbEntegrationSqlServerAsync(builder.Configuration.GetConnectionString("MSSQL")); // OR
// builder.Services.DbEntegrationSqlServer(builder.Configuration.GetConnectionString("MSSQL")); // Asynchronous and synchronous usage is up to your preference.

var app = builder.Build();
// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
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

`builder.Services.DbEntegrationSqlServer(builder.Configuration.GetConnectionString("MSSQL"));` we ensure integration by adding this line to your code.



Let's create an example entity:
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

    [Column(IsNotNull = true)]
    public double Price { get; set; }

    public int CategoryId { get; set; }
}
```

You can specify a special name for the table that will be created on the database side for the entity you created with the table attribute.
`[Table(TableName="TABLE_NAME")]`

We can make special definitions for the column that will be created on the database side for the property you created with the column attribute.
`[Column(IsNotNull = true, IsPrimaryKey = true, IsAutoIncrement = true)]`

The entity to be created must be implemented from the **IDbEntity** interface. Entities that are not implemented through the **IDbEntity** interface will not be migrated.

To benefit from these attributes, you will need to download the **`FmgLib.Orm.Common`** package from Nuget.

Let's take an example code:
```csharp
using FmgLib.Orm.DbHelper;
using FmgLib.Orm.DbHelper.SqlServer;
using FmgLib.TestWeb.Models;
using Microsoft.AspNetCore.Mvc;

namespace FmgLib.TestWeb.Controllers
{
    public class HomeController : Controller
    {
        private readonly IDbCenter _dbCenter;

        public HomeController(IDbCenter dbCenter)
        {
            _dbCenter = dbCenter;
        }

        public IActionResult Index()
        {
            var queryGenerate = new SqlQueryGenerate();
            List<Product> products = new List<Product>();
            for (int i = 1; i <= 100; i++)
            {
                var product = new Product
                {
                    ProductName = $"Deneme Test {i * 4}",
                    Price = i * 8,
                    CategoryId = i,
                };

                products.Add(product);
            }

            var sqlserverinsert = queryGenerate.SqlInsertRange(products); // Insert Range Operation
            var resultInsertSS = _dbCenter.SendQuery<Product, int>(sqlserverinsert); // Send SQL Query

            var queryString = queryGenerate.GetQuery<Product>(x => x.Price >= 200, x => x.Price, Orm.Common.OrderedType.Desc);
            var result = _dbCenter.SendQuery<Product, List<Product>>(queryString);
            return View();
        }
    }
}
```
In this code, methods to generate SQL are provided to us via the 'queryGenerate' object. `SqlInsertRange` function takes List as parameter. As a result, the SQL query is returned.

`GetQuery<TEntity>` method helps for listing. As the first parameter, you can write your **WHERE** condition with a lambda expression. As the second parameter, the **Order** operation, that is, the sorting criterion, is determined (lambda is selected with an expression.) and **ASC** is handled by default. If you want to perform the reverse operation, you can give `OrderedType` as the third parameter.
#
**All written SQL queries are sent to the database with the `SendQuery<TEntity, TResult>(string query)` method of the `_dbCenter` object. `TEntity` here represents the Model to be processed and `TResult` represents the value to be returned.**
