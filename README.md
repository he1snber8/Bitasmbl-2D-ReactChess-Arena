# 2D-ReactChess-Arena

## Description
A web-based 2D multiplayer chess game built with React.js for the client, ASP.NET Core for the API, SignalR for real-time interactions, and PostgreSQL for durable game state storage.

## Tech Stack
- ASP.NET Core
- Entity Framework Core
- PostgreSQL
- SignalR
- React.js

## Requirements
- Real-time move broadcasting via SignalR hub (use Clients.Others.SendAsync to notify other clients)
- Define EF Core entity for game persistence (annotate primary key with [Key] or follow conventions)
- Implement basic client-side move validation (use regex to ensure moves follow algebraic notation)

## Installation
### Server
1. Clone the repo: `git clone https://github.com/your-org/Bitasmbl-2D-ReactChess-Arena.git`
2. Navigate to server directory: `cd server`
3. Set the database connection string in `appsettings.json` or via environment variable:
   - `ConnectionStrings__DefaultConnection='Host=<HOST>;Database=<DB>;Username=<USER>;Password=<PWD>'`
4. Install dependencies: `dotnet restore`
5. Apply migrations: `dotnet ef migrations add InitialCreate` and `dotnet ef database update`
6. Run the server: `dotnet run`

### Client
1. Navigate to client directory: `cd client`
2. Install dependencies: `npm install`
3. Run the client: `npm start`

## Usage
1. Open the client in your browser: `http://localhost:3000`
2. Create or join a game lobby.
3. Enter moves in algebraic notation (e.g., e4, Nf3).
4. Watch moves broadcast in real-time to other players.

## Implementation Steps
1. Scaffold an ASP.NET Core Web API project with SignalR and EF Core dependencies.
2. Configure EF Core to use PostgreSQL in `Startup.cs`:
   csharp
   services.AddDbContext<AppDbContext>(options =>
       options.UseNpgsql(Configuration.GetConnectionString("DefaultConnection")));
   
3. Define the `Game` entity:
   csharp
   public class Game
   {
       [Key]
       public Guid Id { get; set; }
       public string Moves { get; set; }
       public DateTime CreatedAt { get; set; }
   }
   
4. Implement a SignalR hub (`GameHub`) for move broadcasting:
   csharp
   public class GameHub : Hub
   {
       public async Task SendMove(string gameId, string move)
       {
           // Persist move to database
           await Clients.Others.SendAsync("ReceiveMove", move);
       }
   }
   
5. Add API endpoints in `GamesController` to handle game creation, retrieval, and moves.
6. Scaffold a React app with Create React App in the `client` folder.
7. Install `@microsoft/signalr` and establish a connection:
   js
   import { HubConnectionBuilder } from '@microsoft/signalr';
   const connection = new HubConnectionBuilder()
     .withUrl("/gamehub")
     .build();
   
8. Implement basic move validation in the client:
   js
   const moveRegex = /^[KQRBN]?[a-h][1-8]$/;
   if (!moveRegex.test(move)) {
     throw new Error("Invalid move notation");
   }
   
9. Handle incoming moves and update the UI in real-time.
10. Deploy server and client, setting the production database connection string accordingly.

## API Endpoints
- POST `/api/games`: Create a new game
- GET `/api/games/{id}`: Retrieve game state
- POST `/api/games/{id}/move`: Submit a move
- SignalR Hub endpoint: `/gamehub`