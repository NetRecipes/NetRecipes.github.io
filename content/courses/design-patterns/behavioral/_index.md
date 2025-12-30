---
title: "🎭 Behavioral Patterns"
weight: 3
summary: 'Define how objects interact and communicate.'
---

## When to use

When you need to manage algorithms, responsibilities, and communication between objects while keeping them loosely coupled.

## Key traits

- Focus on object collaboration and delegation
- Encapsulate behavior variations
- Define protocols for object interaction
- Enable runtime behavior changes

## Common Scenarios

| Pattern | When to Use |
| --- | --- |
| 🎯 Strategy | Selecting algorithms at runtime or swapping business logic implementations |
| 👁️ Observer | Event-driven systems where multiple objects need to react to state changes |
| 📋 Command | Encapsulating requests for undo/redo, queuing, or logging operations |
| 📝 Template Method | Defining algorithm skeleton while letting subclasses override specific steps |
| 🔄 State | Changing object behavior based on its internal state (state machines) |
| ⛓️ Chain of Responsibility | Processing requests through a chain where any handler can process or pass along |
| 🔁 Iterator | Accessing elements of a collection sequentially without exposing its structure |
| 🤝 Mediator | Centralizing complex communications and dependencies between objects |
| 🚶 Visitor | Adding new operations to existing object structures without modifying them |
| 💾 Memento | Capturing and restoring object state for undo/rollback functionality |
| 📖 Interpreter | Implementing domain-specific languages or parsing expression grammars |
