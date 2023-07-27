---
title: Intro to Entity Framework Core in .NET 6
date: '2023-07-19'
tags: ['C#', '.NET', 'efcore', 'Csharp']
draft: false
summary: Entity framework is a .NET tool to interact with various kind of databases, providing the same api for each of these database providers. In this blog post, you will see how to set un Entity Framework and build your first CRUD operation over a Sqlite database
images: []
layout: PostLayout
canonicalUrl: ''
authors: ['djoufson']
---

# Introduction to Entity Framework Core in .NET 6

![Intro to Entity Framework Core in .NET 6](/static/images/posts/intro-to-entity-framework-core-in-net-6/banner.jpg)

Entity framework is a .NET tool to interact with various kind of databases, providing the same api for each of these database providers. In this blog post, you will see how to set un Entity Framework and build your first CRUD operation over a Sqlite database

# Overview

- [Requirements](#requirements)
- [Project setup](#project-setup)
- [Create the entities](#create-the-entities)
- [Connect to the database](#connect-to-the-database)
- [Test over the database](#test-over-the-database)
- [Next steps](#next-steps)

# Requirements

For this tutorial I'll be using Vs Code as my code editor, and all the stuff will be made through CLI. You can use your favorite Code editor, and your favorite CLI tool (Powershell, CMD, cmdr ...)

## Install .NET

Since this is a demo of efcore using .NET 6, you first need to download and install the [.NET 6 SDK](). Make sure you install the one compatible with your processor architecture.

## Install efcore cli tool

To install the CLI tool of Entity framework, you need to run the following commands

```shell
dotnet tool install --global dotnet-ef
```

```shell
dotnet tool update --global dotnet-ef
```

After those two commands, you can run the next one to confirm that everything was installed successfully:

```shell
dotnet ef
```

And the output should be:
![Test Ef core installation](/static/images/posts/intro-to-entity-framework-core-in-net-6/ef-test.png)

# Project setup

Once the required tools are installed, we can go ahead and create our C# project.

To follow along with me, you should run the exact commands shown below. I will obviously explain why I run a specific command, and what is it for.

## Create a Console project

```shell
dotnet new console -o "Ef-Intro"
```

```shell
cd Ef-Intro
```

Those two commands are to create a blank .NET Console application. Open the `Ef-Intro` created folder in your code editor.

To run the project, type:

```shell
dotnet run
```

`Output`

```
Hello, World!
```

## Install Entity Framework Packages

Type the following commands in the root directory of your project

```shell
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 7.0.5
```

```shell
dotnet add package Microsoft.EntityFrameworkCore.Design --version 7.0.5
```

```shell
dotnet add package Microsoft.EntityFrameworkCore --version 7.0.5
```

```shell
dotnet add package Microsoft.EntityFrameworkCore.Sqlite.Core --version 7.0.5
```

After that, your Ef-Intro.csproj file should look like:

```XML
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <RootNamespace>Ef_Intro</RootNamespace>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="7.0.5" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="7.0.5">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.EntityFrameworkCore.Sqlite" Version="7.0.5" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="7.0.5">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>

</Project>

```

Those are the required packages you have to install in order to properly work with Ef Core

# Create the entities

The application we are going to build is a simple stock application, that manage to stores the products list and prices in a database.To do so, we will need a single `Product` entity.

The entity will be the model used to materialize the results coming from our database

In the root project directory, create a folder called `Entities` and place there the `Product.cs` class as follows:

```
Entities/
|  |__ Product.cs
|
| Program.cs

```

The content of this class should look like:

```cs
namespace Ef_Intro.Entities;

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Quantity { get; set; }
}
```

# Connect to the database

## Create the DbContext

The `DbContext` is a class that allows us to have database access over the existing tables, and this class is in charge of tracking modifications on the database and sync those ones.

I a folder called `Data`, create a ew file `AppDbContext.cs`.

```
Data/
|  |__ AppDbContext.cs
|
Entities/
|  |__ Product.cs
|
| Program.cs

```

and fill it with the following content

```cs
using Ef_Intro.Entities;
using Microsoft.EntityFrameworkCore;

namespace Ef_Intro.Data;

public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; } = null!;
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlite("Data Source=app.db");
    }
}
```

Here we initialize our DbContext to have a single DbSet mapped to the `Product` class, and the resulting Sqlite database will be stored in the `app.db` file.

## Create and Run the migrations

A migration is a way to design the database scheme, and keep track of the updates we made over our database scheme and design. It has indications about how the database should look like, which tables has which columns.

**To create a migration, we run the command**

```shell
dotnet ef migrations add "Initial Create"
```

Where `"Initial Create"` is just the given name of our migration.

**To run our migration, we run the command**

```shell
dotnet ef database update
```

Once those two commands completed, you should be able to see the `app.db` file created in your root project directory, and if you open this file in your favorite Sqlite db explorer, you should see:

![Database scheme](/static/images/posts/intro-to-entity-framework-core-in-net-6/dbscheme.png)

This means that our Database has been updated successfully, according to the Migration specifications.

We can finally test it.

# Test over the database

Now that we have a running database, we can test our database.

First, as we do things the right way, we will create in a `Services` folder, all the interfaces and their concrete implementations. We will need two services, the `IInputManager` responsible to proceed an action based on the selected option, and the `IProductsService` responsible of the CRUD operations over the `Product` entity table in our database.

Here is the content of those files.

### IInputManager

```cs
namespace Ef_Intro.Services;

public interface IInputManager
{
    bool Proceed(int input);
}
```

### IProductsService

```cs
using Ef_Intro.Entities;

namespace Ef_Intro.Services;

public interface IProductsService
{
    Product Add(Product product);
    Product? Get(int id);
    Product? Update(int id, decimal newPrice);
    bool Delete(int id);
    IReadOnlyList<Product> GetAll();
}
```

Then, in the same folder we create the corresponding implementations of each service like follows

### InputManager

```cs
using Ef_Intro.Entities;
using Ef_Intro.Utilities;

namespace Ef_Intro.Services;

public class InputManager : IInputManager
{
    private readonly IProductsService _productsService;

    public InputManager(IProductsService productsService)
    {
        _productsService = productsService;
    }

    public bool Proceed(int input)
    {
        Console.Clear();
        return input switch
        {
            1 => PrintProducts(),
            2 => AddProduct(),
            3 => UpdateProductPrice(),
            4 => DeleteProduct(),
            _ => Exit()
        };
    }

    private static bool Exit()
    {
        return false;
    }

    private bool AddProduct()
    {
        string name = GetStringInput("Enter Product Name: ");
        decimal price = GetDecimalInput("Enter Price: ");
        int quantity = GetIntInput("Enter Quantity: ");

        if(price == -1 && quantity == -1)
        {
            Display.InvalidInput();
            return false;
        }
        else
        {
            var product = new Product()
            {
                Name = name,
                Price = price,
                Quantity = quantity
            };
            return _productsService.Add(product) is not null;
        }
    }

    private bool DeleteProduct()
    {
        int id = GetIntInput("Enter Product Id: ");
        if(id == -1)
            return false;
        else
            return _productsService.Delete(id);
    }

    private bool UpdateProductPrice()
    {
        int id = GetIntInput("Enter Product Id: ");
        if(id == -1)
            return false;
        else
        {
            decimal price = GetIntInput("Enter Price: ");
            if(price == -1)
                return false;
            else
                return _productsService.Update(id, price) is not null;
        }
    }

    private bool PrintProducts()
    {
        var products = _productsService.GetAll();
        if(products is null)
            return false;
        else
        {
            Display.Products(products);
            return true;
        }
    }

    #region  Helper Methods
    private static string GetStringInput(string message)
    {
        Console.Write(message);
        return Console.ReadLine() ?? string.Empty;
    }

    private static int GetIntInput(string message)
    {
        Console.Write(message);
        return int.TryParse(Console.ReadLine(), out var result) ?
            result : -1;
    }

    private static decimal GetDecimalInput(string message)
    {
        Console.Write(message);
        return decimal.TryParse(Console.ReadLine(), out var result) ?
            result : -1;
    }
    #endregion
}
```

And the other one

### ProductsService

```cs
using Ef_Intro.Data;
using Ef_Intro.Entities;
using Microsoft.EntityFrameworkCore;

namespace Ef_Intro.Services;

public class ProductsService : IProductsService
{
    private readonly AppDbContext _dbContext;

    public ProductsService(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public Product Add(Product product)
    {
        _dbContext.Products.Add(product);
        _dbContext.SaveChanges();
        return product;
    }

    public bool Delete(int id)
    {
        int records = _dbContext.Products.Where(p => p.Id == id).ExecuteDelete();
        return records > 0;
    }

    public Product? Get(int id)
    {
        return _dbContext.Products.Find(id);
    }

    public IReadOnlyList<Product> GetAll()
    {
        return _dbContext.Products.ToArray();
    }

    private Product? Update(Product product)
    {
        _dbContext.Products.Update(product);
        _dbContext.SaveChanges();
        return product;
    }

    public Product? Update(int id, decimal newPrice)
    {
        Product? product = Get(id);
        if (product == null)
        {
            return null;
        }
        product.Price = newPrice;
        return Update(product);
    }
}
```

Last but not least, we add some Utility classes in the root `Utilities` folder

### Database.cs

```cs
using Ef_Intro.Data;
namespace Ef_Intro.Utilities;

public static class Database
{
    public static void InitDatabase(AppDbContext dbContext)
    {
        bool isEmpty = dbContext.Products.Any();
        if (!isEmpty)
        {
            dbContext.Products.AddRange(
                new() { Name = "Laptop", Price = 1000, Quantity = 50 },
                new() { Name = "Mouse", Price = 50, Quantity = 25 },
                new() { Name = "Keyboard", Price = 100, Quantity = 10 }
            );

            dbContext.SaveChanges();
        }
    }
}
```

### Display.cs

```cs
using Ef_Intro.Data;
using Ef_Intro.Entities;

namespace Ef_Intro.Utilities;

public static class Display
{
    public static void Print(string message)
    {
        Console.WriteLine(message);
    }

    public static void Products(IEnumerable<Product> products)
    {
        foreach (var product in products)
        {
            Console.WriteLine("Id: " + product.Id);
            Console.WriteLine("Name: " + product.Name);
            Console.WriteLine("Price: " + product.Price);
            Console.WriteLine("Quantity: " + product.Quantity);
            Console.WriteLine("===============================");
        }
    }

    public static int Menu()
    {
        Console.WriteLine("1. Print products");
        Console.WriteLine("2. Add product");
        Console.WriteLine("3. Update product price");
        Console.WriteLine("4. Delete product");
        Console.WriteLine("5. Exit");
        Console.Write("Enter your choice: ");
        if (int.TryParse(Console.ReadLine(), out int choice))
        {
            return choice;
        }

        return -1;
    }

    public static void ResetConsole()
    {
        Console.WriteLine("\nPress any key to continue . . .");
        Console.ReadKey();
        Console.Clear();
    }

    internal static void InvalidInput()
    {
        Console.WriteLine("The input you submitted is not in a valid format");
        Console.WriteLine("\nPress any key to continue . . .");
        Console.ReadKey();
        Console.Clear();
    }
}
```

The entry point finally looks like.

### `Program.cs`

```cs
using Ef_Intro.Data;
using Ef_Intro.Services;
using Ef_Intro.Utilities;

internal class Program
{
    private static void Main()
    {
        bool exit = false;
        // We instantiate our db context
        using var dbContext = new AppDbContext();
        IProductsService productsService = new ProductsService(dbContext);
        IInputManager inputManager = new InputManager(productsService);

        // We initialize the database
        Database.InitDatabase(dbContext);

        while (!exit)
        {
            int input = Display.Menu();
            exit = !inputManager.Proceed(input);
            Display.ResetConsole();
        }

        // We close the db context
        dbContext.Dispose();
    }
}
```

And by running the command

```shell
dotnet run
```

The output should be

![Output result](/static/images/posts/intro-to-entity-framework-core-in-net-6/output.png)

A fully implemented CRUD application using .NET 6 and Entity Framework Core.
You can find the complete code [here](https://github.com/Djoufson/Ef-Intro)
