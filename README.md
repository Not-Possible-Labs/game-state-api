# Game State API Service

## Overview
This service is designed to sync game state in real-time, store moves, and manage timers for a chess application where users can place bets on the game. The system leverages WebSockets and Redis to ensure efficient and reliable communication between components.

## Features
- Real-time game state synchronization
- Move and timer storage
- WebSocket communication
- Redis-based server registration and matchmaking
- Region-aware and load-balanced server selection

## Architecture
The architecture consists of two main components:

1. **Game State API**: Handles WebSocket connections and manages game state.
2. **Matchmaking API**: Selects the best game state server for players to connect to.

## Dependencies
- Redis
- WebSockets

## Installation
1. Clone the repository.
2. Install dependencies.
3. Configure Redis and WebSocket settings.
4. Run the server.

## Usage
- **Registering Servers**: Game state API servers register themselves in Redis with their ID, IP, port, region, and load.
- **Heartbeat**: Servers update their status and TTL periodically to maintain active status.
- **Matchmaking**: The matchmaking API queries Redis to find the best server for a match, considering load and region.
- **Connection**: Players receive connection info and connect to the assigned game state API instance.
- **Gameplay**: All real-time events are managed through the game state API.
- **Reconnection**: If a player disconnects, they can reconnect using the matchmaking API to retrieve server info.

## System Flow Diagrams

### 1. Game State API Logic
```mermaid
graph TD;
    A[Start] --> B[Receive WebSocket connection]
    B --> C{Validate connection}
    C -->|Valid| D[Subscribe to Redis channel]
    C -->|Invalid| E[Close connection]
    D --> F[Listen for gameplay events]
    F --> G{Process event}
    G -->|Move| H[Update game state]
    G -->|Timer| I[Update timer]
    G -->|Resignation| J[Handle resignation]
    H --> K[Broadcast updated state]
    I --> K
    J --> K
    K --> L[End]
    E --> L
```

### 2. Matchmaking Service Logic
```mermaid
graph TD;
    A[Start] --> B[Receive match request]
    B --> C{Check available servers}
    C -->|Yes| D[Query Redis for server list]
    C -->|No| E[Wait and retry]
    D --> F[Evaluate server load and region]
    F --> G{Select best server}
    G -->|Success| H[Assign server to match ID]
    G -->|Failure| I[Log error and notify user]
    H --> J[Store server info in Redis]
    J --> K[Send connection info to players]
    K --> L[End]
    E --> B
    I --> L
```

### 3. Complete System Communication Flow
```mermaid
graph TD;
    subgraph Redis
        A[game-state-api servers register] --> B[ID, IP, port, region, load]
        C[game-state-api servers update status and TTL]
    end
    subgraph Matchmaking API
        D[Queries Redis for active servers] --> E[Selects best game-state-api instance]
        E --> F[Stores server info under match ID]
    end
    subgraph Players
        G[Receive WebSocket connection info] --> H[Connect to game-state-api instance]
    end
    subgraph Game State API
        I[Handles WebSocket connection] --> J[Subscribes to Redis channel]
        J --> K[Real-time gameplay events flow]
    end
    F --> G
    H --> I
    I --> K
    L[Player disconnects] --> M[Client calls matchmaking-api to rejoin]
```

## Contributing
Contributions are welcome! Please submit a pull request or open an issue for any bugs or feature requests.

## License
This project is licensed under the MIT License.
