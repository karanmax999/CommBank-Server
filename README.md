# CommBank Goal Tracker

A full-stack application consisting of a React + TypeScript frontend and a .NET 6 + MongoDB backend designed for tracking financial goals and transactions.

---

## 🏗️ Project Structure

The repository is organized into the following main directories:

- **[`CommBank-Server/`](file:///d:/project/CommBank-Server/CommBank-Server)**: ASP.NET Core Web API backend using MongoDB Driver.
- **[`commbank-web/`](file:///d:/project/CommBank-Server/commbank-web)**: React single-page application built with TypeScript, Redux, and styled-components.
- **[`CommBank.Tests/`](file:///d:/project/CommBank-Server/CommBank.Tests)**: C# xUnit test project for testing controllers and business logic.
- **[`Server.sln`](file:///d:/project/CommBank-Server/Server.sln)**: Solution file containing the backend and test projects.

---

## 🛠️ Prerequisites

To run this project locally, ensure you have the following installed:

- **[.NET 6.0 SDK](https://dotnet.microsoft.com/download/dotnet/6.0)**
- **[Node.js](https://nodejs.org/)** (v16+ recommended)
- **[Git](https://git-scm.com/)**

---

## 🚀 Running the Application Locally

### 1. Start the Backend API
Navigate to the backend directory and run the .NET development server:

```bash
cd CommBank-Server
dotnet run
```
By default, the server will start and listen on:
- **Swagger UI**: [http://localhost:5203/swagger/index.html](http://localhost:5203/swagger/index.html)
- **API Base URL**: `http://localhost:5203` or `http://localhost:11366`

### 2. Start the Frontend Web Client
Navigate to the web application directory, install dependencies, and start the React dev server:

```bash
cd commbank-web
npm install
npm run start
```
By default, the web client runs on:
- [http://localhost:3000](http://localhost:3000)

*Note: The frontend is configured to automatically detect whether it is running on `localhost` and point API requests to `http://localhost:5203`. In other environments, it defaults to the Azure API endpoint.*

---

## ⚙️ Configuration & Secrets

### Backend Configuration
Database credentials and endpoints are configured in:
1. **[`appsettings.json`](file:///d:/project/CommBank-Server/CommBank-Server/appsettings.json)**
2. **[`Secrets.json`](file:///d:/project/CommBank-Server/CommBank-Server/Secrets.json)**

Connection strings default to a MongoDB Atlas cluster. To customize, update the `ConnectionStrings.CommBankDb` property in these configuration files.

### Frontend API URL
API routing configuration is managed in **[`commbank-web/src/api/lib.ts`](file:///d:/project/CommBank-Server/commbank-web/src/api/lib.ts)**.

---

## 🧪 Testing

The unit test suite uses **xUnit** to test backend API controllers with mock (fake) collections and services.

To run all unit tests:
```bash
dotnet test
```
The test suite resides in the **[`CommBank.Tests`](file:///d:/project/CommBank-Server/CommBank.Tests)** project.
