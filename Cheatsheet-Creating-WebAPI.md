# Cheatsheet Creating a WebAPI.NET Project
## Create a Database
1. Open a SQL server and create a **Database**:

``` SQL
CREATE DATABASE Database_NAME
```
2. Using the **Database** you just created create a **Table**:

``` SQL
USE Database_NAME
 
CREATE TABLE Table_NAME
(Column1 int primary key identity(1,1) ,
Column2 int NOT NULL CONSTRAINT fk_Table_NAME_Table2 REFERENCES [Table2](id),
Column3 varchar(100) NOT Null, 
Column4 varchar (255) NOT NULL, 
Column5 date NOT NULl, 
Column6 varchar(25) NOT NULL DEFAULT 'Pickup', 
Column7 bit NOT NULL,
Column8 decimal(10,2) NOT NULL DEFAULT 0,

```
3. Insert **Data** into the **Table** you just created:

``` SQL
INSERT INTO Table_NAME
(Column2, Column3, Column4, Column5) 
VALUES
(1, 'Some Words', 'More Words', '0001/01/01');

```
4. Check that the **Table** was created and has **Data**.  

### Find the Connection String in SQL
Right click on the **Server** and open **Properties**.
Click **View connect properties**
Copy the **Connection Name**.
         "*Server Name=DESKTOP-VDMQ3OL\SQLEXPRESS*"

## Create a new project in Visual Studio

Open Visual Studio 
* From the **File** menu, select **New > Project.**
* Enter *Web API* in the search box.
* Select the **ASP.NET Core Web API** that supports C# template and select **Next**.
* In the **Configure your new project dialog**, name the project (*ProjectName*) and select **Next**.
* In the **Additional information** dialog:
    * Confirm the **Framework** an even numbered **.NET 6.0 or Higher (Long Term Support)**.
    * Confirm the checkbox for Use controllers is checked.
    * Confirm the checkbox for Enable OpenAPI support is unchecked.
    * Select Create.

### Add NuGet packages
NuGet packages must be added to support the connection to SQL

* From the **Tools** menu, select **NuGet Package Manager > Manage NuGet Packages for Solution.**
* Select the **Browse** tab.
* Enter **SqlServer** in the search box, and then select **Microsoft.EntityFrameworkCore.SqlServer**.
* After selecting the right NuGet Package make sure the  *ProjectName* is checkmarked.
* Go to the **Version** drop-down and select the latest **6.0 Version** to match the **Framework** of the project.
* Select the **Project** checkbox in the right pane and then select **Install**.

### Remove the WeatherForecast 
The project template creates a `WeatherForecast` API. 
* Delete the `WeatherForecast.cs` **Class**
* Delete the `WeatherForecastController.cs` **Controller**

Also remove the `WeatherForecast` from the Url.
* From the **Project** menu, select **Project Properties** 
* Select **Debug** then **Open debug launch profiles UI**
* Scroll down to **Url** and remove `WeatherForecast`

### Add Connection String 
Go to the **Solution Explorer** and open the *appsettings.json* file.
Insert the following code:
``` 
 "AllowedHosts": "*",
  "ConnectionStrings": {
    "PRSConnectionString": "server=DESKTOP-VDMQ3OL\\SQLEXPRESS;database=PRSHaines;Integrated Security=True;TrustServerCertificate=False;"
  }
}
```
### Add a PrsDbContext Class
 A model is a set of classes that represent the data that the app manages.
* In **Solution Explorer**, right-click the project. Select **Add > New Folder**. Name the folder `Models`.
* Right-click the `Models` folder and select **Add > Class**. Name the class *PrsDbContext.cs* and select **Add**.

Replace the template code with the following:
``` C#
namespace ProjectName.Models
{
    public class PrsDbContext : DbContext
    {

        public PrsDbContext(DbContextOptions<PrsDbContext> options) : base(options)
        {

        }
    }
}
```
If the code shows red squigglies hoover over it and click on **Show potenital fixes** select **Using Microsoft.EntityFrameworkCore**.

### Register the Database 
Update `Program.cs` by replacing the following code:
``` C#
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();

var app = builder.Build();

// Configure the HTTP request pipeline.

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();

```
with: 
``` C#

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<PrsDbContext>(
     // lambda
     options => options.UseSqlServer(builder.Configuration.GetConnectionString("PRSConnectionString"))
     );

var app = builder.Build();

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();
```
If you get red squigglies add using directives:
`using Microsoft.EntityFrameworkCore;`
`using WebAPIDELETE.Models;`

### Create Model Class for the 
* Right-click the `Models` folder and select **Add > Class**. Name the class after the table from SQL *Table_NAME.cs* and select **Add**.
    Replace the template code with the following:
``` C#
namespace ProjectName
{
    [Table("Table_NAME")]
    public class Table_NAME
    {
        [Key] 
        public int Column1 { get; set; }

        [Required]
        public int Column2 { get; set; }

        [Required]
        [StringLength(100)]
        public string Column3 { get; set; } //to encrypt

        [Required]
        [StringLength(255)]
        public string Column4 { get; set; }

        [Required]
        public DateTime Column5 { get; set; }
        
        [Required]
        [StringLength(25)]
        public string Column6 { get; set; }

        [Required]
        public bool Column7 { get; set; } = false;

        [Required]
        public decimal Column8 { get; set; } = 0;


    }
}
```
If the code does not work add the following using statements: 
`using System.ComponentModel.DataAnnotations;`
`using System.ComponentModel.DataAnnotations.Schema;`

### Add a Controller
Add a [ApiController] to create [actions] 

* Right-click the Controllers folder.

* Select **Add > Controller**.

* Select **API Controller with actions, using Entity Framework**, and then select **Add**.

    In the Add API Controller with actions, using Entity Framework dialog:
        Select *Table-NAME(ProjectName.Models)* in the **Model class**.
        Select *PrsDbContext(ProjectName.Models)* in the **Data context class**.
        Next to **Controller name** name the control so it corresponds to the model name *Table_NAMEsController*
        Select Add.

    If adding the Controller operation fails, select Add to try Controller a second time.
The ASP.NET Core templates for:

* Controllers with views include [action] in the route template.
* API controllers don't include [action] in the route template.

When the [action] token isn't in the route template, the action name (method name) isn't included in the endpoint. That is, the action's associated method name isn't used in the matching route.

inside the controller make sure to add `using Microsoft.Data.SqlClient;`

### Add 

In the `PrsDbContext.cs` model add ```public DbSet<*Table-NAME*> *Table-NAMEs* { get; set; }```


