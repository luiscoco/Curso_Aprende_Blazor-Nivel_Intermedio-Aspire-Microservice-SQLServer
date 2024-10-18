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

```


### 2.7. We create the Migration

```
Add-Migration DatabaseInitialization
```

```
Upadate-Database
```





## 3. We add the Aspire Host project


## 4. We add the ServiceDefaults project


## 5. We add .NET Aspire Orchestrator support in the .NET Web API project


## 6. We add .NET Aspire Orchestrator support in the Blazor Web project


## 7. We modify the Aspire Host project middleware to add SQL Server and Database


## 8. We modify the .NET Web API project middleware to add the SQL Server DbContext

## 9 Set the SQL Server database connection string in appSettings.json in the .NET Web API project



## 10. Running the application


### 10.1. We run the application



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
