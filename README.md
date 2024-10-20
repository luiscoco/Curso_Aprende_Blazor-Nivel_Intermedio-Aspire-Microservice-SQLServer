# How to create with Aspire .NET9 Blazor WebAssambly application consuming a MicroService CRUD SQL Server

## 1. Create a Blazor WebAssembly application with Visual Studio 2022

We verify the project folder and files structure

![image](https://github.com/user-attachments/assets/8feb95cc-b7d3-4c65-bdd8-ec51f7a10355)

We select the Blazor WebAssembly project template

![image](https://github.com/user-attachments/assets/20bf4dbd-eed9-4645-8d54-8976ee85b6ba)

We input the project name and folder in the hard disk, and press the Next button

![image](https://github.com/user-attachments/assets/30c7d13d-6482-46d7-a21c-b85794e3211f)

We select the .NET 9 framework and press the Create button

![image](https://github.com/user-attachments/assets/d6a32eb1-ce9c-4cf8-9af0-162b3e69ecd9)

We see the project folders and files structure

![image](https://github.com/user-attachments/assets/66e52bff-bbc5-4090-913f-401fdcbf4ae1)

### 1.2. We create Data Model and the CRUD Service

![image](https://github.com/user-attachments/assets/83ba5b9d-ffa4-4d15-81da-db76b140a13b)

This folder typically contains classes that represent the data structure (models) used throughout the application

The ExampleModel class is a simple model that represents an entity with four properties:

**Id** (an integer identifier)

**Name** (optional string for the name)

**Description** (optional string for the description)

**CreatedDate** (a DateTime indicating when the object was created)

This class would typically be used to pass data around within a Blazor WebAssembly application, such as between components or to/from an API

```csharp
namespace BlazorWebAssemblyUI.Models
{
    public class ExampleModel
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        public string? Description { get; set; }
        public DateTime CreatedDate { get; set; }

    }
}
```

This service (ExampleModelService) acts as a bridge between the Blazor WebAssembly front end and the API back end

It provides methods to perform CRUD operations (create, read, update, delete) on ExampleModel objects by sending HTTP requests to the API

Each method uses async/await for asynchronous operations, ensuring the application remains responsive while waiting for API responses

```csharp
using BlazorWebAssemblyUI.Models;
using System.Net.Http.Json;

namespace BlazorWebAssemblyUI.Services
{
    public class ExampleModelService
    {
        private readonly HttpClient _httpClient;

        public ExampleModelService(HttpClient httpClient)
        {
            _httpClient = httpClient;
        }

        // Get all ExampleModels
        public async Task<List<ExampleModel>> GetAllExampleModels()
        {
            return await _httpClient.GetFromJsonAsync<List<ExampleModel>>("api/ExampleModels");
        }

        // Get ExampleModel by ID
        public async Task<ExampleModel> GetExampleModelById(int id)
        {
            return await _httpClient.GetFromJsonAsync<ExampleModel>($"api/ExampleModels/{id}");
        }

        // Add new ExampleModel
        public async Task<HttpResponseMessage> AddExampleModel(ExampleModel model)
        {
            return await _httpClient.PostAsJsonAsync("api/ExampleModels", model);
        }

        // Update ExampleModel
        // Update ExampleModel
        public async Task<HttpResponseMessage> UpdateExampleModel(int id, ExampleModel model)
        {
            return await _httpClient.PutAsJsonAsync($"api/ExampleModels/{id}", model);
        }

        // Delete ExampleModel
        public async Task<HttpResponseMessage> DeleteExampleModel(int id)
        {
            return await _httpClient.DeleteAsync($"api/ExampleModels/{id}");
        }
    }
}
```


### 1.3. We load the Nuget Packages

These two packages are automatically loaded in your application when you create a Blazor WebAssembly application in Visual Studio 2022 

![image](https://github.com/user-attachments/assets/4925a6ee-510c-4b63-b3fd-37a7e2025867)

### 1.4. We add a new razor component for CRUD operations

![image](https://github.com/user-attachments/assets/72b0b82d-9026-4ea9-9486-c2dc86e0023c)

![image](https://github.com/user-attachments/assets/d71a7095-95d1-4935-91a0-9bdb262d819a)

```razor
@page "/examplecomponent"

@using BlazorWebAssemblyUI.Services
@using BlazorWebAssemblyUI.Models
@inject ExampleModelService ExampleModelService

<h3 class="text-center">Example Models</h3>

@if (exampleModels == null)
{
    <p><em>Loading...</em></p>
}
else
{
    <div class="table-responsive">
        <table class="table table-striped table-hover">
            <thead class="table-dark">
                <tr>
                    <th>Model Name</th>
                    <th>Description</th>
                    <th>Actions</th>
                </tr>
            </thead>
            <tbody>
                @foreach (var model in exampleModels)
                {
                    <tr>
                        <td>@model.Name</td>
                        <td>@model.Description</td>
                        <td>
                            <button class="btn btn-warning btn-sm me-2" @onclick="() => ShowUpdateForm(model)">Update</button>
                            <button class="btn btn-danger btn-sm" @onclick="() => DeleteExampleModel(model.Id)">Delete</button>
                        </td>
                    </tr>
                }
            </tbody>
        </table>
    </div>
}

<div class="mt-3">
    <button class="btn btn-primary" @onclick="ShowCreateForm">Create New Item</button>
    <button class="btn btn-secondary" @onclick="FetchExampleModels">Refresh</button>
</div>

@if (isFormVisible)
{
    <div class="mt-3">
        <h4>@modalTitle</h4>
        <div class="mb-3">
            <label for="modelName" class="form-label">Model Name</label>
            <input type="text" class="form-control" id="modelName" @bind="currentModel.Name" />
        </div>
        <div class="mb-3">
            <label for="modelDescription" class="form-label">Description</label>
            <input type="text" class="form-control" id="modelDescription" @bind="currentModel.Description" />
        </div>
        <div class="mb-3">
            <label for="createdDate" class="form-label">Created Date</label>
            <input type="datetime-local" class="form-control" id="createdDate" @bind="currentModel.CreatedDate" />
        </div>
        <button class="btn btn-primary" @onclick="SaveModel">Save changes</button>
        <button class="btn btn-secondary" @onclick="HideForm">Cancel</button>
        <p class="text-danger mt-3">@message</p>
    </div>
}

@code {
    private List<ExampleModel> exampleModels;
    private ExampleModel currentModel = new ExampleModel();
    private bool isCreateMode = true;
    private bool isFormVisible = false;
    private string modalTitle = "Create New Item";
    public string message = "";

    protected override async Task OnInitializedAsync()
    {
        await FetchExampleModels();
    }

    private async Task FetchExampleModels()
    {
        exampleModels = await ExampleModelService.GetAllExampleModels();
    }

    // Show Create Form
    private void ShowCreateForm()
    {
        currentModel = new ExampleModel
            {
                CreatedDate = DateTime.Now // Set default CreatedDate to the current date and time
            };
        modalTitle = "Create New Item";
        isCreateMode = true;
        isFormVisible = true;
    }

    // Show Update Form
    private void ShowUpdateForm(ExampleModel model)
    {
        currentModel = new ExampleModel
            {
                Id = model.Id,
                Name = model.Name,
                Description = model.Description,
                CreatedDate = model.CreatedDate
            };
        modalTitle = "Update Item";
        isCreateMode = false;
        isFormVisible = true;
    }

    // Hide form
    private void HideForm()
    {
        isFormVisible = false;
        message = "";
    }

    // Save model (Create or Update)
    private async Task SaveModel()
    {
        // Ensure all required fields are set
        if (string.IsNullOrWhiteSpace(currentModel.Name) ||
            string.IsNullOrWhiteSpace(currentModel.Description) ||  // Ensure Description is set
            currentModel.CreatedDate == default)
        {
            message = "Missing required fields.";
            return;
        }

        // Debugging output to verify Description value
        Console.WriteLine($"Updating Model: Id = {currentModel.Id}, Name = {currentModel.Name}, Description = {currentModel.Description}");

        HttpResponseMessage response;

        if (isCreateMode)
        {
            response = await ExampleModelService.AddExampleModel(currentModel);
        }
        else
        {
            response = await ExampleModelService.UpdateExampleModel(currentModel.Id, currentModel);
        }

        if (response.IsSuccessStatusCode)
        {
            await FetchExampleModels(); // Refresh list
            HideForm(); // Hide form after success
        }
        else
        {
            message = "Error: " + response.ReasonPhrase;
        }
    }

    private async Task DeleteExampleModel(int id)
    {
        var response = await ExampleModelService.DeleteExampleModel(id);

        if (response.IsSuccessStatusCode)
        {
            await FetchExampleModels(); // Refresh the list
        }
        else
        {
            message = "Error deleting model: " + response.ReasonPhrase;
        }
    }
}
```

### 1.5. We modify the middleware(Program.cs)

```csharp
using BlazorWebAssemblyUI;
using BlazorWebAssemblyUI.Services;
using Microsoft.AspNetCore.Components.Web;
using Microsoft.AspNetCore.Components.WebAssembly.Hosting;

var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri("https://localhost:7217/") });
builder.Services.AddScoped<ExampleModelService>();

await builder.Build().RunAsync();
```

### 1.6. We modify the NavMenu.razor component to navigate to the new razor component in section 1.5

```razor
 <div class="nav-item px-3">
     <NavLink class="nav-link" href="examplecomponent">
         <span class="bi bi-list-nested-nav-menu" aria-hidden="true"></span> AzureSQL_CRUD
     </NavLink>
 </div>
```

## 2. Create .NET Core Web API application

We run Visual Studio 2022 and create a new project

![image](https://github.com/user-attachments/assets/8a267524-e858-4f0b-ad30-ff37d5bc403d)

We select the  project template ASP.NET Core Web API

![image](https://github.com/user-attachments/assets/584955c3-b6ab-45fb-9200-22b877cde646)

We input the project name and location

![image](https://github.com/user-attachments/assets/3a32f1ae-593d-4a44-9b19-0f71f579c08f)

We select the .NET9 framework and press the Create button

![image](https://github.com/user-attachments/assets/4fab5bd7-1a34-42f2-ac56-63138d49740b)

This is the project folders and files structure

![image](https://github.com/user-attachments/assets/2d793140-d7ad-4c1f-a044-c2390882f848)

### 2.1. We create new folders (Controllers, Data, Models and Services) 

![image](https://github.com/user-attachments/assets/a4498ef6-30e3-44d6-9849-2e63324b92c5)

### 2.2. We load the Nuget packages

![image](https://github.com/user-attachments/assets/8d19ea40-c4e6-4260-917a-86f2af646f14)

### 2.3. We create the Data Model

The ExampleModel class is a simple C# model representing a data entity with four properties:

Id (an integer identifier)

Name (an optional string)

Description (an optional string)

CreatedDate (a DateTime object for when the entity was created)

This class would be used in an Azure SQL Web API microservice to define how data is structured, typically when retrieving or storing information in a database through API endpoints

```csharp
namespace AzureSQLWebAPIMicroservice.Models
{
    public class ExampleModel
    {
        public int Id { get; set; }
        public string? Name { get; set; }
        public string? Description { get; set; }
        public DateTime CreatedDate { get; set; }
    }
}
```

The ExampleDbContext class is an Entity Framework Core database context that represents a session with the database

It has a constructor that accepts configuration options (like the connection string), allowing the application to connect to an Azure SQL database

The DbSet<ExampleModel> property maps the ExampleModel class to the corresponding table in the database (ExampleModels), enabling the application to perform CRUD operations (Create, Read, Update, Delete) on that table

In essence, this class acts as the bridge between the application's code and the underlying database

```csharp
using Microsoft.EntityFrameworkCore;
using AzureSQLWebAPIMicroservice.Models;

namespace AzureSQLWebAPIMicroservice.Data
{
    public class ExampleDbContext:DbContext
    {
        public ExampleDbContext(DbContextOptions<ExampleDbContext> options)
        : base(options)
        {
        }

        public DbSet<ExampleModel> ExampleModels { get; set; }
    }
}
```

### 2.4. We create the SQL CRUD Service

The ExampleModelService class encapsulates the logic for interacting with the database for the ExampleModel entity

It supports CRUD operations:

Create (AddExampleModel) adds a new model to the database

Read (GetAllExampleModels and GetExampleModelById) fetches records from the database

Update (UpdateExampleModel) modifies an existing record

Delete (DeleteExampleModel) removes a record from the database

This service simplifies database interaction for other parts of the application and ensures that changes are handled efficiently and asynchronously

```charp
using AzureSQLWebAPIMicroservice.Data;
using AzureSQLWebAPIMicroservice.Models;
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace AzureSQLWebAPIMicroservice.Services
{
    public class ExampleModelService
    {
        private readonly ExampleDbContext _context;

        public ExampleModelService(ExampleDbContext context)
        {
            _context = context;
        }

        // Create
        public async Task<ExampleModel> AddExampleModel(ExampleModel model)
        {
            // The Id should not be set explicitly, let the database handle it
            _context.ExampleModels.Add(model);
            await _context.SaveChangesAsync();

            return model;  // The Id will be auto-generated after SaveChangesAsync
        }


        // Read all
        public async Task<List<ExampleModel>> GetAllExampleModels()
        {
            return await _context.ExampleModels.ToListAsync();
        }

        // Read by ID
        public async Task<ExampleModel> GetExampleModelById(int id)
        {
            return await _context.ExampleModels.FirstOrDefaultAsync(e => e.Id == id);
        }

        // Update
        // Update
        public async Task<ExampleModel> UpdateExampleModel(int id, ExampleModel model)
        {
            var existingModel = await _context.ExampleModels.FirstOrDefaultAsync(e => e.Id == id);
            if (existingModel == null)
            {
                return null;
            }

            // Update all necessary fields
            existingModel.Name = model.Name;
            existingModel.Description = model.Description; // Ensure description is updated
            existingModel.CreatedDate = model.CreatedDate; // Update other fields as necessary

            _context.Entry(existingModel).State = EntityState.Modified;
            await _context.SaveChangesAsync();

            return existingModel;
        }

        // Delete
        public async Task<bool> DeleteExampleModel(int id)
        {
            var model = await _context.ExampleModels.FindAsync(id);
            if (model == null)
            {
                return false;
            }

            _context.ExampleModels.Remove(model);
            await _context.SaveChangesAsync();

            return true;
        }
    }
}
```

### 2.7. We create the Database Migration

As first step is required to **Run Docker Desktop** and **run SQL Server docker container** with this command:

```
docker run ^  -e "ACCEPT_EULA=Y" ^  -e "MSSQL_SA_PASSWORD=Luiscoco123456" ^  -p 1433:1433 ^  -d mcr.microsoft.com/mssql/server:2022-latest
```

![image](https://github.com/user-attachments/assets/ea27877a-30d6-4cac-a64e-c9abee749856)

**IMPORTANT NOTE**: we run the SQL Server docker container with the **password Luiscoco123456** and in the **port 1433**

We verify SQL Server docker container is running:

![image](https://github.com/user-attachments/assets/038eb540-0c28-440e-9238-84bc4be398c9)

![image](https://github.com/user-attachments/assets/45dd3b5f-bcf5-41ed-89f5-a7e240b35bae)

We can also connect in **SSMS** to the SQL Server database docker container

![image](https://github.com/user-attachments/assets/39509732-7832-4960-a2dc-a989b4e8045c)

![image](https://github.com/user-attachments/assets/7bb71418-6088-476b-a7d6-5508b41c93dd)

This is the connection string in **appSettings.json** file

```json
 "ConnectionStrings": {
   "sqldata": "Server=127.0.0.1,1433;Database=sqldb;User Id=sa;Password=Luiscoco123456;Trusted_Connection=False;TrustServerCertificate=True;"
 }
```

See in the Web API

![image](https://github.com/user-attachments/assets/9829d9ea-9b21-4fed-b796-89fa506b81eb)

We first set the **API project** as the **StartUp project**

![image](https://github.com/user-attachments/assets/1f8e469b-32ce-49b4-ad37-eb94221c693f)

We **open Packages Manager** and run these commands:

![image](https://github.com/user-attachments/assets/54312109-6541-467b-b380-11cc7e8f5c56)

We select API project in the dropdown list

![image](https://github.com/user-attachments/assets/4ebd46bb-105e-4a7d-b048-99eaca821a62)

And we run this command to create the **Migration** folder and files

```
Add-Migration DatabaseInitialization
```

We verify the Migrations were created

![image](https://github.com/user-attachments/assets/0cc6d296-4b39-482f-9351-d63b4e33d97b)

Then run this command to apply the migrations changes in the database

```
Upadate-Database
```

![image](https://github.com/user-attachments/assets/c3682522-ada1-45a3-8003-bde83dcd7b4f)

We connect to the database in SSMS and confirm the migrations were applied

![image](https://github.com/user-attachments/assets/69f6c637-9776-4b85-90e3-0e41abae3e9e)

![image](https://github.com/user-attachments/assets/91fcc36f-f614-4938-8e5c-68193f58cfd3)

![image](https://github.com/user-attachments/assets/1407646c-44fa-4cbe-a1f4-f4796690bde9)

## 2.8 We modify the Web API middleware 

We have to include the DbContext (EntityFramework reference in the application)

We also have to reference the connection string

```csharp
builder.Services.AddDbContext<ExampleDbContext>(options =>
{
    var connectionString = builder.Configuration.GetConnectionString("sqldata");
    options.UseSqlServer(connectionString);
});
```

We have to register the CRUD operations Service

```csharp
builder.Services.AddScoped<ExampleModelService>();
```

See the whole middleware Program.cs code:

```csharp
using Microsoft.EntityFrameworkCore;
using AzureSQLWebAPIMicroservice.Data;
using AzureSQLWebAPIMicroservice.Services;
using Microsoft.OpenApi.Models;
using Microsoft.EntityFrameworkCore.SqlServer;

var builder = WebApplication.CreateBuilder(args);

//builder.AddServiceDefaults();

// Add services to the container.
builder.Services.AddControllers();

//builder.AddSqlServerDbContext<ExampleDbContext>("sqldata");

builder.Services.AddDbContext<ExampleDbContext>(options =>
{
    var connectionString = builder.Configuration.GetConnectionString("sqldata");
    options.UseSqlServer(connectionString);
});

//builder.Services.AddDbContext<ExampleDbContext>();

builder.Services.AddScoped<ExampleModelService>();

// Add CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy("BlazorPolicy", policy =>
    {
        policy.WithOrigins("https://localhost:7130") // Your Blazor app's URL
              .AllowAnyMethod()
              .AllowAnyHeader();
    });
});

builder.Services.AddEndpointsApiExplorer();

// Add Swagger for API documentation
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "My API", Version = "v1" });
});

var app = builder.Build();

//app.MapDefaultEndpoints();

// Configure the HTTP request pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Use defined CORS policy
app.UseCors("BlazorPolicy");

app.UseAuthorization();

app.MapControllers();

app.Run();
```

## 3. We configure the StartUp project and Run the application

We configure the startup projects

![image](https://github.com/user-attachments/assets/186341d8-01e0-4ff5-aac6-f0be595613dc)

![image](https://github.com/user-attachments/assets/0d9d8274-ae5e-458b-8133-8b1534fd783e)

We run the application

![image](https://github.com/user-attachments/assets/415c7396-e1ac-4fe4-aaaf-e1b642ddd6d5)

![image](https://github.com/user-attachments/assets/bbf10a6c-583c-4ffa-a041-dd861bb01726)

![image](https://github.com/user-attachments/assets/7390a559-a8e4-483c-b474-27af6296f6c2)

Now we create new items 

![image](https://github.com/user-attachments/assets/d4dd9b62-7c4e-49b8-8a6c-0f375002cbc8)

![image](https://github.com/user-attachments/assets/08e4ab32-157c-4aeb-bc46-91009533cf32)

![image](https://github.com/user-attachments/assets/5813aada-0182-4f21-a805-69f87c0d8dcc)

We confirm in SSMS

![image](https://github.com/user-attachments/assets/7f9d4814-240f-46c1-a22e-faac211c4e0e)

Note: the Id is 2 because when creating this document I previously created an Item and I deleted it

## 4. We add the Aspire Host project

We add a new project in the solution

![image](https://github.com/user-attachments/assets/acd642f9-4c84-43bb-8b56-f30fc7c7097d)

We select the Aspire Host .NET9 project template and we press the Next button

![image](https://github.com/user-attachments/assets/17065022-f7ff-43ca-a291-c524bc3475ea)

We input the project name and location and press the Next button

![image](https://github.com/user-attachments/assets/37ed5bdc-250c-47b7-a3b7-86a6d433a971)

We select the .NET9 framework and press the Create button

![image](https://github.com/user-attachments/assets/76a3527d-ba1d-47b6-bc70-e202e4a72624)

See the new project folders and files structure

![image](https://github.com/user-attachments/assets/609197c1-2e4b-4106-9bda-ca38441e8302)

![image](https://github.com/user-attachments/assets/2007f834-81dc-436f-b137-ee404ed36011)

## 5. We add the ServiceDefaults project

We add a new project in the solution

![image](https://github.com/user-attachments/assets/df6aa532-ad41-4f2a-b81d-380d83639fb8)

We select the Aspire .NET Service Default project template

![image](https://github.com/user-attachments/assets/87f8b2ce-3042-4eaa-9568-2510f30f4e84)

We input the project name and location

![image](https://github.com/user-attachments/assets/49962b3f-ad0a-454f-8adc-9f05b4963f6c)

We select the .NET9 framework and press the Create button

![image](https://github.com/user-attachments/assets/f8b18c56-5cd8-41d8-a30e-9f6cba07eacb)

We verify the new projecr folders and files structure

![image](https://github.com/user-attachments/assets/a06e4e0e-b907-4f69-848c-be8ee3a9251e)

And also we verify in the Web API project the reference to the Aspire .NET Service Default projec

![image](https://github.com/user-attachments/assets/34498001-d2bd-4de1-bcf8-9606ee12a03e)

## 6. We add .NET Aspire Orchestrator support in the .NET Web API project

![image](https://github.com/user-attachments/assets/947ad58d-dd02-45cc-a344-c15d786a6945)

We confirm the new project was added in the Aspire Host middleware

![image](https://github.com/user-attachments/assets/b68239ee-0f40-4e6e-8e67-211b130df6bf)

We also confirm the Web API middleware was modified

![image](https://github.com/user-attachments/assets/4ea3988c-404a-4e4b-930a-f5773be6eae4)

![image](https://github.com/user-attachments/assets/a6b1fc8a-71ae-4c7e-bea6-4025f79a1ead)

## 7. We add .NET Aspire Orchestrator support in the Blazor Web project

![image](https://github.com/user-attachments/assets/70471491-dc3d-42c8-812b-5e8650cc1044)

We confirm the new project was added in the Aspire Host middleware

![image](https://github.com/user-attachments/assets/6799448f-5d37-4526-b117-1348b67ccd8b)

## 8. We modify the Aspire Host project middleware

We first set the **Database Password**

```
var sqlPassword = builder.AddParameter("sql-password");
```

We register and configure the **SqlServer** database

```
var sqldb = builder.AddSqlServer("sql", sqlPassword, port: 1234)
                       .WithDataVolume("MyDataVolume").AddDatabase("sqldb");
```

We register and configure the **Web API** project

```
var northernTradersCatalogAPI = builder.AddProject<Microservice_AzureSQL>("microservice-azuresql")
                                       .WithExternalHttpEndpoints()
                                       .WithReference(sqldb);
```

We register and configure the **FronT-End** project

```
builder.AddProject<BlazorWebAssemblyUI>("blazorwebassemblyui").WithReference(sqldb);
```

See the middleware whole code:

```csharp
using Aspire.Hosting; // Ensure this namespace is correct
using Projects; // Make sure you're referencing the correct project namespace

var builder = DistributedApplication.CreateBuilder(args);

var sqlPassword = builder.AddParameter("sql-password");

var sqldb = builder.AddSqlServer("sql", sqlPassword, port: 1234)
                       .WithDataVolume("MyDataVolume").AddDatabase("sqldb");

var northernTradersCatalogAPI = builder.AddProject<Microservice_AzureSQL>("microservice-azuresql")
                                       .WithExternalHttpEndpoints()
                                       .WithReference(sqldb);

builder.AddProject<BlazorWebAssemblyUI>("blazorwebassemblyui").WithReference(sqldb);

builder.Build().Run();
```

## 9. Set the SQL Server database connection string in appSettings.json in the .NET Web API project

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "sqldata": "Server=127.0.0.1,1234;Database=sqldb;User Id=sa;Password=Luis9876;Trusted_Connection=False;TrustServerCertificate=True;"
  }
}
```

## 10. Running the application

### 10.1. The SQL Server Docker container is run automatically when you run the application

We verify in Docker Desktop

![image](https://github.com/user-attachments/assets/961d0796-1184-44d0-9682-e725bbd632ff)

![image](https://github.com/user-attachments/assets/3031dd32-775f-4904-8840-5e03d6814227)

We also defined a Volume for assuring the data persistance

![image](https://github.com/user-attachments/assets/4b5adb5a-9024-443a-b31d-adb6fd944d04)

### 10.1. We connect to the Sql container with SSMS and we create the database and seed with data

We connect to the SQL Server docker container with **SSMS**

We have to consider the paramters in the connection string **appsettings.json** file

```json
 "ConnectionStrings": {
   "sqldata": "Server=127.0.0.1,1234;sqldb=Database;User Id=sa;Password=Luis9876;Trusted_Connection=False;TrustServerCertificate=True;"
 }
```

See the **SSMS input data**

![image](https://github.com/user-attachments/assets/e162c196-080c-466e-ab2c-dfa6a01e24aa)

We also select the **Trusted Certificate** option

![image](https://github.com/user-attachments/assets/e804ec90-113e-47f0-a2f4-e73fa0c79b01)

We verify the database connection

![image](https://github.com/user-attachments/assets/b93c6a83-60f2-495f-a4d4-1e3dc9fc299b)

Instead of migrating the dagabase we **create the database**, **create the table** and **seed with data** with sql queries:

We first create the database and use it

```sql
CREATE DATABASE sqldb
GO

USE sqldb
GO
```

We create the table

```sql
-- Create the ExampleModels table
CREATE TABLE ExampleModels (
    Id INT IDENTITY(1, 1) NOT NULL PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Description NVARCHAR(255) NOT NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE()
);
```

We seed the table with data

```sql
-- Insert initial data
INSERT INTO ExampleModels (Name, Description, CreatedDate)
VALUES
    ('Sample Name 1', 'Sample Description 1', '2024-01-10 22:50:19.1711895'),
    ('Sample Name 2', 'Sample Description 2', '2024-01-10 22:50:19.1711946');
```

We verify the above queries were run successfully

![image](https://github.com/user-attachments/assets/7d1d6b2c-37dd-4be8-a9d0-f2a46d39f1b1)

### 10.2. We run the application

We open the application Dashboard

https://localhost:17175/

![image](https://github.com/user-attachments/assets/69b39042-4399-4d23-8008-b29cc73b5444)

We also access the Web API

https://localhost:7217/swagger/index.html

![image](https://github.com/user-attachments/assets/1672f33b-4887-4998-b907-d45a9036d476)

![image](https://github.com/user-attachments/assets/adff93a9-5f68-4707-91d0-7f171604bc80)

We also access the Application

![image](https://github.com/user-attachments/assets/c0300f2b-3b69-4eb7-9dda-b3e81a565d34)

