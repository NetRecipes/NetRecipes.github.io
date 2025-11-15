---
title: "Let's design a Scrum Poker 🎴"
summary: "Scrum Poker is an online, real-time, multiplayer game."
weight: 2
---

Endpoints, Service Contracts and Sequence Diagrams for building a `Scrum Poker` app.

---

## High level design

```mermaid
graph TD
  Browser["Browser Clients"]
  BFF["Blazor Server App – UI + BFF"]
  API["ASP NET 9 Web API"]
  Store["State Store"]
  PubSub["Pub/Sub Broker"]

  Browser -->|User interaction| BFF
  BFF -->|HTTP REST calls| API
  API -->|Save / retrieve state| Store
  API -.->|Publish events| PubSub
  PubSub -.->|Event notification| BFF
  BFF -->|Push UI updates| Browser
```

---

## Endpoints

| Operation | Method & Path |
| --- | --- |
| [Create New Game](./create-new-game) | **POST** `/games` |
| [Join Existing Game](./join-existing-game) | **POST** `/games/{gameId}/join` |
| [Submit Vote](./submit-vote) | **POST** `/games/{gameId}/vote` |
| [Reset Votes](./reset-votes) | **POST** `/games/{gameId}/votes/reset` |
| [Get Game State](./get-game-state) | **GET** `/games/{gameId}` |
| [Remove a Player](./remove-a-player) | **DELETE** `/admin/games/{gameId}/players/{player}` |
| [End Game](./end-game) | **DELETE** `/admin/games/{gameId}` |

---

## Client Updates - Real-time

This flow is common for all endpoints

```mermaid
sequenceDiagram
  participant PubSub as "Pub/Sub Broker"
  participant API as "ASP.NET 9 Web API"
  participant Store as "State Store"
  participant Hub as "SignalR Hub"
  participant Clients as "Connected Clients"

  PubSub->>API: Event:{gameId,...}
  API->>Store: GET game:{gameId}

  alt Game not found
    API-->>Hub: SendAsync("GameDeleted", gameId)
    Hub-->>Clients: GameDeleted event pushed
  else Game exists
    Store-->>API: GameResponse
    API-->>Hub: SendAsync("GameUpdated", GameResponse)
    Hub-->>Clients: GameUpdated event pushed
  end
```
