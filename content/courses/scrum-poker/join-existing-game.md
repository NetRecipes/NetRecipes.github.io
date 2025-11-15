---
date: '2025-11-15T19:58:20+05:30'
draft: true
title: 'Join Existing Game'
summary: 'Endpoint to `Join an existing Game`'
tags: ['Design', 'Architecture']
weight: 20
---

```mermaid
sequenceDiagram
  participant BFF as "Blazor Server App – UI + BFF"
  participant API as "ASP.NET 9 Web API"
  participant Store as "State Store"
  participant PubSub as "Pub/Sub Broker"

  BFF->>API: POST /games/{gameId}/join
  API->>Store: GET game:{gameId}

  alt Semantically invalid data  
    API-->>BFF: 422 Unprocessable Entity  
    Note right of API: Payload → Error  
  else Game not found  
    API-->>BFF: 404 Not Found  
    Note right of API: Payload → Error  
  else Player already exists  
    API-->>BFF: 409 Conflict  
    Note right of API: Payload → Error  
  else Success  
    API->>Store: SAVE game:{gameId} {updatedState}  
    API-->>PubSub: PUBLISH PlayerJoined:{gameId,player}
    API-->>BFF: 200 OK  
    Note right of API: Payload → GameResponse  
  end
```

---

## Request

```mermaid
classDiagram
    class JoinGameRequest {
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

## Publish - PlayerJoined

```mermaid
classDiagram
    class PlayerJoinedEvent {
      + string gameId
      + string player
    }
```
