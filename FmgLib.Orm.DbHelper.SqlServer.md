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
using FmgLib.Develop;
using FmgLib.Orm.DbHelper;
using FmgLib.Orm.DbHelper.SqlServer;
using FmgLib.TestWeb.Models;
using Microsoft.AspNetCore.Mvc;

namespace FmgLib.TestWeb.Controllers
{
    public class HomeController : Controller
    {
        public HomeController()
        {
        }

        public IActionResult Index()
        {
            List<Product> products = new List<Product>();
            for (int i = 1; i <= 50; i++)
            {
                var item = new Product
                {
                    ProductName = $"Deneme Test {i * 4}",
                    Price = i * 8,
                    CategoryId = i,
                };

                products.Add(item);
            }

            var query = Query.GetQuery<Product>()
                .Where(x => x.Price > 250 && x.ProductName.Contains("Dene"))
                .DistinctBy(x => x.Price)
                .DistinctBy(x => x.ProductName)
                .OrderByDescending(x => x.Price);

            #region SqlServer Queries

            #region Sync

            var qqqq = query.AsQuerySqlServer().First();
            var wwww = query.AsQuerySqlServer().First(x => x.Id > 10);
            var eeee = query.AsQuerySqlServer().FirstOrDefault();
            var rrrr = query.AsQuerySqlServer().FirstOrDefault(x => x.Id > 10);
            var tttt = query.AsQuerySqlServer().ElementAt(5);
            var yyyy = query.AsQuerySqlServer().ElementAtOrDefault(5);
            var uuuu = query.AsQuerySqlServer().Min(x => x.Price);
            var cccc = query.AsQuerySqlServer().MinBy(x => x.Price);
            var oooo = query.AsQuerySqlServer().Max(x => x.Price);
            var pppp = query.AsQuerySqlServer().MaxBy(x => x.Price);
            var llll = query.AsQuerySqlServer().Sum(x => x.Price);
            var zzzz = query.AsQuerySqlServer().Average(x => x.Price);
            var aaaa = query.AsQuerySqlServer().Count();
            var ssss = query.AsQuerySqlServer().Count(x => x.Price < 600);
            var dddd = query.AsQuerySqlServer().Any();
            var ffff = query.AsQuerySqlServer().Any(x => x.Price < 600);
            var gggg = query.AsQuerySqlServer().Skip(20);
            var hhhh = query.AsQuerySqlServer().Take(20);
            var xxxx = query.AsQuerySqlServer().SkipAndTake(10, 20);
            var kkkk = query.AsQuerySqlServer().ToList();

            #endregion

            #region Async

            var qqqq0 = await query.AsQuerySqlServer().FirstAsync();
            var wwww0 = await query.AsQuerySqlServer().FirstAsync(x => x.Id > 10);
            var eeee0 = await query.AsQuerySqlServer().FirstOrDefaultAsync();
            var rrrr0 = await query.AsQuerySqlServer().FirstOrDefaultAsync(x => x.Id > 10);
            var tttt0 = await query.AsQuerySqlServer().ElementAtAsync(5);
            var yyyy0 = await query.AsQuerySqlServer().ElementAtOrDefaultAsync(5);
            var uuuu0 = await query.AsQuerySqlServer().MinAsync(x => x.Price);
            var cccc0 = await query.AsQuerySqlServer().MinByAsync(x => x.Price);
            var oooo0 = await query.AsQuerySqlServer().MaxAsync(x => x.Price);
            var pppp0 = await query.AsQuerySqlServer().MaxByAsync(x => x.Price);
            var llll0 = await query.AsQuerySqlServer().SumAsync(x => x.Price);
            var zzzz0 = await query.AsQuerySqlServer().AverageAsync(x => x.Price);
            var aaaa0 = await query.AsQuerySqlServer().CountAsync();
            var ssss0 = await query.AsQuerySqlServer().CountAsync(x => x.Price < 600);
            var dddd0 = await query.AsQuerySqlServer().AnyAsync();
            var ffff0 = await query.AsQuerySqlServer().AnyAsync(x => x.Price < 600);
            var gggg0 = await query.AsQuerySqlServer().SkipAsync(20);
            var hhhh0 = await query.AsQuerySqlServer().TakeAsync(20);
            var xxxx0 = await query.AsQuerySqlServer().SkipAndTakeAsync(10, 20);
            var kkkk0 = await query.AsQuerySqlServer().ToListAsync();

            #endregion

            #endregion

            #region SqlServer Commands

            #region Sync

            var insertr = Command
                .Insert(new Product
                {
                    ProductName = "One item insert test",
                    Price = 1234,
                    CategoryId = 234
                })
                .AsCommandSqlServer()
                .Save();

            var insertrMulti = Command
                .InsertRange(products)
                .AsCommandSqlServer()
                .Save();

            var updater = Command
                .Update(new Product
                {
                    ProductName = "222 TEST DENEME 22",
                    Price = 6134,
                    CategoryId = 155
                }, x => x.Id == 102)
                .AsCommandSqlServer()
                .Save();

            var deleter = Command
                .Delete<Product>(x => x.Id == 102)
                .AsCommandSqlServer()
                .Save();

            #endregion

            #region Async

            var insertr0 = await Command
                .Insert(new Product
                {
                    ProductName = "One item insert test",
                    Price = 1234,
                    CategoryId = 234
                })
                .AsCommandSqlServer()
                .SaveAsync();

            var insertrMulti0 = await Command
                .InsertRange(products)
                .AsCommandSqlServer()
                .SaveAsync();

            var updater0 = await Command
                .Update(new Product
                {
                    ProductName = "222 TEST DENEME 22",
                    Price = 6134,
                    CategoryId = 155
                }, x => x.Id == 102)
                .AsCommandSqlServer()
                .SaveAsync();

            var deleter0 = await Command
                .Delete<Product>(x => x.Id == 102)
                .AsCommandSqlServer()
                .SaveAsync();

            #endregion

            #endregion

            return View();
        }
    }
}
```
