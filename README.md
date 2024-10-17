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

### 2.6. We create the database Migrations

**IMPORTANT NOTE**: Add the Initial Migration and Update the database before running the application

Before running the appplication we have to run **Docker Desktop** and run a **SQL Server Docker image** withe the following command

```
docker run ^  -e "ACCEPT_EULA=Y" ^  -e "MSSQL_SA_PASSWORD=Luiscoco123456" ^  -p 1433:1433 ^  -d mcr.microsoft.com/mssql/server:2022-latest
```

We verify the SQL Server docker container is running in **Docker Desktop**

![image](https://github.com/user-attachments/assets/8e25a2d3-0c5b-4bec-9102-0650c23d8d31)

![image](https://github.com/user-attachments/assets/4ae42ca1-0150-4aeb-b003-1c949358e667)

Then we have to select as StartUp project the CRUD Web API project 

![image](https://github.com/user-attachments/assets/2b7fe699-bb55-4b0e-acc1-a4ffe0558d6b)

And then we open the **Packets Manager Console**

![image](https://github.com/user-attachments/assets/daebb5f9-7417-4d1a-be5c-4d3d623c5cfa)

Please verify se selected the Web API CRUD projec in the **Packets Manager Console**

![image](https://github.com/user-attachments/assets/c6b5d7ce-65b8-4d60-a106-2a24410c362f)

Then we first add a database initialization migration

![image](https://github.com/user-attachments/assets/df3b0edd-27e0-4466-8907-35153cff9b4f)

```
Add-Migration InitialMigration
```

And the we update the database

![image](https://github.com/user-attachments/assets/f0db4928-0054-4210-9fe6-36b415952557)

```
Update-Database
```

Now we can connect to the database with **SSMS** to verify the data

![image](https://github.com/user-attachments/assets/1c49a347-6cfd-4cff-8915-4ffa1e13855f)

We alredy set the password(**Luiscoco123456**) when running the Docker container

![image](https://github.com/user-attachments/assets/fb3fcf37-bc24-4a9c-b7bd-03152e8c1ae2)

## 3. We add the Aspire Host project


## 4. We add the ServiceDefaults project


## 5. We add .NET Aspire Orchestrator support in the .NET Web API project


## 6. We add .NET Aspire Orchestrator support in the Blazor Web project


## 7. We modify the Aspire Host project middleware to add the SQL Server Database


## 8. We modify the .NET Web API project middleware to add the SQL Server DbContext

## 9 Set the SQL Server database connection string in the .NET Web API project

```
 "ConnectionStrings": {
   "sqldata": "Server=localhost,1433;Database=YourDatabase;User Id=sa;Password=Luiscoco123456;Trusted_Connection=False;TrustServerCertificate=True;"
 }
```

## 10. Running the application

We set as StartUp Project the Aspire Host project

![image](https://github.com/user-attachments/assets/c986ef73-f470-45cd-9881-3a2bf350d4b0)

We run the application with the HTTP protocol

![image](https://github.com/user-attachments/assets/f33660e8-610f-4b4a-9ebe-867253f1d5f5)

We verify the application

![image](https://github.com/user-attachments/assets/c7de94c1-6589-4fa2-87e7-026c9ea9351b)

![image](https://github.com/user-attachments/assets/33a4579a-55da-4644-85f0-e07fe64e676b)

![image](https://github.com/user-attachments/assets/f185ef94-3c41-4b14-9242-b6c6b8173202)

![image](https://github.com/user-attachments/assets/e61d1211-4a81-4520-97f5-ced41d82d622)
