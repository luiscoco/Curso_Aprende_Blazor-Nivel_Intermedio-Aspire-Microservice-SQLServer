# How to create with .NET Aspire a Blazor WebAssambly application consuming a MicroService CRUD SQL Server



## 1. Create a Blazor WebAssembly application

### 1.2. We create the folders (Models and services)

### 1.3. We load the Nuget Packages

### 1.4. We add a new razor component for CRUD operations

### 1.5. We modify the middleware(Program.cs)

### 1.6. We modify the NavMenu.razor 


## 2. Create .NET Core Web API application



### 2.1. We create new folders (Controllers, Data, Migrations, Models and Services) 

### 2.2. We load the Nuget packages

### 2.3. We create the Data Model

### 2.4. We create the SQL Service CRUD Service

### 2.5. We create the Service

### 2.6. We create the database in SSMS

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


## 3. We add the Aspire Host project


## 4. We add the ServiceDefaults project


## 5. We add .NET Aspire Orchestrator support in the .NET Web API project


## 6. We add .NET Aspire Orchestrator support in the Blazor Web project


## 7. We modify the Aspire Host project middleware to add SQL Server and Database


## 8. We modify the .NET Web API project middleware to add the SQL Server DbContext

## 9 Set the SQL Server database connection string in appSettings.json in the .NET Web API project



## 10. Running the application


