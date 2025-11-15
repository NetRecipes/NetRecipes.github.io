---
date: '2025-11-15T20:00:12+05:30'
title: 'Submit Vote'
summary: 'Endpoint to `Submit a Vote`'
tags: ['Design', 'Architecture']
weight: 30
---

```mermaid
sequenceDiagram
  participant BFF as "Blazor Server App – UI + BFF"
  participant API as "ASP.NET 9 Web API"
  participant Store as "State Store"
  participant PubSub as "Pub/Sub Broker"

  BFF->>API: POST /games/{gameId}/vote  
  API->>Store: GET game:{gameId}

  alt Semantically invalid data  
    API-->>BFF: 422 Unprocessable Entity  
    Note right of API: Payload → Error  
  else Game not found  
    API-->>BFF: 404 Not Found  
    Note right of API: Payload → Error  
  else Player not in game  
    API-->>BFF: 409 Conflict  
    Note right of API: Payload → Error  
  else Success  
    API->>Store: SAVE game:{gameId} {updatedState}  
    API-->>PubSub: PUBLISH VoteSubmitted:{gameId,player,value}
    API-->>BFF: 200 OK  
    Note right of API: Payload → GameResponse  
  end
```

---

## Request

```mermaid
classDiagram
    class VoteRequest {
      + string player
      + int value
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

## Publish - VoteSubmitted

```mermaid
classDiagram
    class VoteSubmittedEvent {
      + string gameId
      + string player
      + int value
    }
```
