# How to create with .NET Aspire a Blazor WebAssambly application consuming a MicroService CRUD SQL Server



## 1. Create a Blazor WebAssembly application

We verify the project folder and files structure

![image](https://github.com/user-attachments/assets/81411a27-74c7-40d9-b629-b6cd7014143b)

### 1.2. We create Data Model and the CRUD Service

![image](https://github.com/user-attachments/assets/83ba5b9d-ffa4-4d15-81da-db76b140a13b)

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

These two packages are automatically loaded in your application when you create it in Visual Studio

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

### 1.6. We modify the NavMenu.razor 

```razor
 <div class="nav-item px-3">
     <NavLink class="nav-link" href="examplecomponent">
         <span class="bi bi-list-nested-nav-menu" aria-hidden="true"></span> AzureSQL_CRUD
     </NavLink>
 </div>
```

## 2. Create .NET Core Web API application

This is the project folders and files structure

![image](https://github.com/user-attachments/assets/2d793140-d7ad-4c1f-a044-c2390882f848)

### 2.1. We create new folders (Controllers, Data, Models and Services) 

![image](https://github.com/user-attachments/assets/a4498ef6-30e3-44d6-9849-2e63324b92c5)

### 2.2. We load the Nuget packages

![image](https://github.com/user-attachments/assets/8d19ea40-c4e6-4260-917a-86f2af646f14)

### 2.3. We create the Data Model

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

IMPORTANT NOTE: we run the SQL Server docker container with the **password Luiscoco123456** and in the **port 1433**

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

We run the applicagion

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




## 5. We add the ServiceDefaults project


## 6. We add .NET Aspire Orchestrator support in the .NET Web API project


## 7. We add .NET Aspire Orchestrator support in the Blazor Web project


## 8. We modify the Aspire Host project middleware to add SQL Server and Database


## 9. We modify the .NET Web API project middleware to add the SQL Server DbContext

## 10 Set the SQL Server database connection string in appSettings.json in the .NET Web API project



## 11. Running the application


### 11.1. We run the application



We check the SQL Server docker container is running




### 10.2. We create the database in SSMS

```sql
CREATE DATABASE sqldb
GO

USE sqldb
GO

-- Create the ExampleModels table
CREATE TABLE ExampleModels (
    Id INT IDENTITY(1, 1) NOT NULL PRIMARY KEY,
    Name NVARCHAR(100) NOT NULL,
    Description NVARCHAR(255) NOT NULL,
    CreatedDate DATETIME2 NOT NULL DEFAULT GETDATE()
);

-- Insert initial data
INSERT INTO ExampleModels (Name, Description, CreatedDate)
VALUES
    ('Sample Name 1', 'Sample Description 1', '2024-01-10 22:50:19.1711895'),
    ('Sample Name 2', 'Sample Description 2', '2024-01-10 22:50:19.1711946');
```
