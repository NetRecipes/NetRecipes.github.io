---
date: '2025-11-15T19:54:22+05:30'
title: 'Create New Game'
summary: 'Endpoint to `Create a new Game`'
tags: ['Design', 'Architecture']
weight: 10
---

```mermaid
sequenceDiagram
  participant BFF as "Blazor Server App – UI + BFF"
  participant API as "ASP.NET 9 Web API"
  participant Store as "State Store"
  participant PubSub as "Pub/Sub Broker"

  BFF->>API: POST /games
  alt Semantically invalid data  
    API-->>BFF: 422 Unprocessable Entity  
    Note right of API: Payload → Error  
  else Success  
    API->>Store: SAVE game:{gameId} {initialState}  
    API-->>PubSub: PUBLISH GameCreated:{gameId}
    API-->>BFF: 201 Created  
    Note right of API: Payload → GameResponse  
  end
```

---

## Request

```mermaid
classDiagram
    class CreateGameRequest {
      + string player
    }
```

---

## Response

```mermaid
classDiagram
    class GameResponse {
      + string gameId
      + string[] players
      + DateTime createdAt
      + Vote[] votes
    }
    class Vote {
      + string player
      + int? value
    }

    GameResponse o-- Vote : votes
```

---

## Error

```mermaid
classDiagram
    class Error {
      + string Code
      + string Message
    }
```

---

## Publish - GameCreated

```mermaid
classDiagram
    class GameCreatedEvent {
      + string gameId
    }
```
