DbHelper is a modern object-relational mapper (O/RM) for .NET. It supports updates, and schema migrations. DbHelper  works with SQL Server, SQLite, MySQL, PostgreSQL, and Oracle through a provider plugin API.

To use it, you first need to download the `FmgLib.Orm.DbHelper.SQLite` package from Nuget.

You can easily include the package in your application in ***Program.cs***. Example usage is as follows:

```csharp
using FmgLib.Orm.DbHelper.SQLite;

var builder = WebApplication.CreateBuilder(args);
// Add services to the container.
builder.Services.AddControllersWithViews();

await builder.Services.DbEntegrationSQLiteAsync(builder.Configuration.GetConnectionString("SQLITE")); // OR
// builder.Services.DbEntegrationSQLite(builder.Configuration.GetConnectionString("SQLITE")); // Asynchronous and synchronous usage is up to your preference.

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

`builder.Services.DbEntegrationSQLite(builder.Configuration.GetConnectionString("SQLITE"));` we ensure integration by adding this line to your code.



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
using FmgLib.Orm.DbHelper.SQLite;
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

            #region SQLite Queries

            #region Sync

            var qqqq = query.AsQuerySQLite().First();
            var wwww = query.AsQuerySQLite().First(x => x.Id > 10);
            var eeee = query.AsQuerySQLite().FirstOrDefault();
            var rrrr = query.AsQuerySQLite().FirstOrDefault(x => x.Id > 10);
            var tttt = query.AsQuerySQLite().ElementAt(5);
            var yyyy = query.AsQuerySQLite().ElementAtOrDefault(5);
            var uuuu = query.AsQuerySQLite().Min(x => x.Price);
            var cccc = query.AsQuerySQLite().MinBy(x => x.Price);
            var oooo = query.AsQuerySQLite().Max(x => x.Price);
            var pppp = query.AsQuerySQLite().MaxBy(x => x.Price);
            var llll = query.AsQuerySQLite().Sum(x => x.Price);
            var zzzz = query.AsQuerySQLite().Average(x => x.Price);
            var aaaa = query.AsQuerySQLite().Count();
            var ssss = query.AsQuerySQLite().Count(x => x.Price < 600);
            var dddd = query.AsQuerySQLite().Any();
            var ffff = query.AsQuerySQLite().Any(x => x.Price < 600);
            var gggg = query.AsQuerySQLite().Skip(20);
            var hhhh = query.AsQuerySQLite().Take(20);
            var xxxx = query.AsQuerySQLite().SkipAndTake(10, 20);
            var kkkk = query.AsQuerySQLite().ToList();

            #endregion

            #region Async

            var qqqq0 = await query.AsQuerySQLite().FirstAsync();
            var wwww0 = await query.AsQuerySQLite().FirstAsync(x => x.Id > 10);
            var eeee0 = await query.AsQuerySQLite().FirstOrDefaultAsync();
            var rrrr0 = await query.AsQuerySQLite().FirstOrDefaultAsync(x => x.Id > 10);
            var tttt0 = await query.AsQuerySQLite().ElementAtAsync(5);
            var yyyy0 = await query.AsQuerySQLite().ElementAtOrDefaultAsync(5);
            var uuuu0 = await query.AsQuerySQLite().MinAsync(x => x.Price);
            var cccc0 = await query.AsQuerySQLite().MinByAsync(x => x.Price);
            var oooo0 = await query.AsQuerySQLite().MaxAsync(x => x.Price);
            var pppp0 = await query.AsQuerySQLite().MaxByAsync(x => x.Price);
            var llll0 = await query.AsQuerySQLite().SumAsync(x => x.Price);
            var zzzz0 = await query.AsQuerySQLite().AverageAsync(x => x.Price);
            var aaaa0 = await query.AsQuerySQLite().CountAsync();
            var ssss0 = await query.AsQuerySQLite().CountAsync(x => x.Price < 600);
            var dddd0 = await query.AsQuerySQLite().AnyAsync();
            var ffff0 = await query.AsQuerySQLite().AnyAsync(x => x.Price < 600);
            var gggg0 = await query.AsQuerySQLite().SkipAsync(20);
            var hhhh0 = await query.AsQuerySQLite().TakeAsync(20);
            var xxxx0 = await query.AsQuerySQLite().SkipAndTakeAsync(10, 20);
            var kkkk0 = await query.AsQuerySQLite().ToListAsync();

            #endregion

            #endregion

            #region SQLite Commands

            #region Sync

            var insertr = Command
                .Insert(new Product
                {
                    ProductName = "One item insert test",
                    Price = 1234,
                    CategoryId = 234
                })
                .AsCommandSQLite()
                .Save();

            var insertrMulti = Command
                .InsertRange(products)
                .AsCommandSQLite()
                .Save();

            var updater = Command
                .Update(new Product
                {
                    ProductName = "222 TEST DENEME 22",
                    Price = 6134,
                    CategoryId = 155
                }, x => x.Id == 102)
                .AsCommandSQLite()
                .Save();

            var deleter = Command
                .Delete<Product>(x => x.Id == 102)
                .AsCommandSQLite()
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
                .AsCommandSQLite()
                .SaveAsync();

            var insertrMulti0 = await Command
                .InsertRange(products)
                .AsCommandSQLite()
                .SaveAsync();

            var updater0 = await Command
                .Update(new Product
                {
                    ProductName = "222 TEST DENEME 22",
                    Price = 6134,
                    CategoryId = 155
                }, x => x.Id == 102)
                .AsCommandSQLite()
                .SaveAsync();

            var deleter0 = await Command
                .Delete<Product>(x => x.Id == 102)
                .AsCommandSQLite()
                .SaveAsync();

            #endregion

            #endregion

            return View();
        }
    }
}
```
