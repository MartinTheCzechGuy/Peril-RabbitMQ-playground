# Peril - RabbitMQ Learning Project

A distributed strategy game built with Go and RabbitMQ to demonstrate publish-subscribe messaging patterns and fundamentals of RabbitMQ.

## Overview

Peril is a multiplayer strategy game where players control armies and battle for territory. The game demonstrates key RabbitMQ concepts including:

- **Publisher-Subscriber pattern**: Game events are published and consumed by multiple clients
- **Exchange types**: Direct and Topic exchanges for different routing scenarios
- **Message serialization**: Both JSON and GOB encoding
- **Queue durability**: Transient vs durable queues
- **Message acknowledgment**: Proper handling of message processing

## Architecture

The project consists of:

- **Game Server** (`cmd/server`): Manages game state and broadcasts pause/resume events
- **Game Clients** (`cmd/client`): Players that can move armies, spawn units, and participate in battles
- **RabbitMQ**: Message broker handling all inter-service communication

### Message Flow

1. **Army Movements**: Published to topic exchange with routing key `army_moves.{username}`
2. **War Declarations**: Published to topic exchange with routing key `war.{username}`
3. **Game State Changes**: Published to direct exchange for pause/resume functionality
4. **Game Logs**: Published to topic exchange with routing key `game_logs.{username}`

## Prerequisites

- **Go 1.22+**
- **Docker** (for RabbitMQ)
- **Git**

## Setup & Installation

1. **Clone the repository**:
   ```bash
   git clone https://github.com/MartinTheCzechGuy/Peril-RabbitMQ-playground.git
   cd Peril-RabbitMQ-playground
   ```
  
2. **Start RabbitMQ**:
   Make sure Docker Desktop is running on your system.
   
   ```bash
   ./rabbit.sh start
   ```
   This will create and start a RabbitMQ container with management interface available at http://localhost:15672 (guest/guest).

3. **Install Go dependencies**:
   ```bash
   go mod download
   ```

## Running the Game

### Start the Game Server

```bash
go run ./cmd/server
```

The server supports these commands:
- `pause` - Pause the game for all players
- `resume` - Resume the game for all players
- `quit` - Shutdown the server

### Start Game Clients

In separate terminals, start multiple clients:

```bash
go run ./cmd/client
```

Each client will prompt for a username and then accept these commands:
- `move <location> <unitID>...` - Move units to a location (e.g., `move asia 1`)
- `spawn <location> <rank>` - Spawn new units (e.g., `spawn europe infantry`)
- `status` - Show current player state and units
- `spam <n>` - Send n malicious log messages for testing
- `help` - Show available commands
- `quit` - Disconnect from the game

### Multiple Server Instances

You can run multiple server instances for testing:

```bash
./multiserver.sh 3  # Starts 3 server instances
```

## Project Structure

```
├── cmd/
│   ├── client/          # Game client application
│   └── server/          # Game server application
├── internal/
│   ├── gamelogic/       # Game rules and state management
│   ├── pubsub/          # RabbitMQ publish/subscribe utilities
│   └── routing/         # Message routing constants and models
├── rabbit.sh            # RabbitMQ container management script
├── multiserver.sh       # Script to run multiple server instances
└── Dockerfile           # RabbitMQ container configuration
```

## RabbitMQ Concepts Demonstrated

### Exchange Types

- **Direct Exchange** (`peril_direct`): Used for pause/resume commands with exact routing key matching
- **Topic Exchange** (`peril_topic`): Used for army movements, wars, and logs with pattern-based routing

### Queue Types

- **Durable Queues**: Survive broker restarts (used for war declarations)
- **Transient Queues**: Deleted when consumers disconnect (used for army movements and pause events)

### Message Serialization

- **JSON**: Human-readable format for most game events
- **GOB**: Go's binary format for game logs (more efficient for Go-to-Go communication)

### Routing Patterns

- `army_moves.{username}` - Player-specific army movements
- `war.*` - All war-related events
- `game_logs.{username}` - Player-specific game logs
- `pause` - Global pause/resume events

## Troubleshooting

- **Connection refused**: Ensure RabbitMQ is running (`./rabbit.sh start`)
- **Port conflicts**: Default RabbitMQ ports are 5672 (AMQP) and 15672 (Management UI)
- **Management Interface**: Access the RabbitMQ management interface at http://localhost:15672 with credentials `guest/guest`