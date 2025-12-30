---
title: "🏗️ Creational Patterns"
weight: 1
summary: 'Control how objects are instantiated.'
---

## When to use

When you need flexibility in object creation - deciding which class to instantiate, managing complex construction, or controlling instance creation.

## Key traits

- Decouple object creation from usage
- Hide instantiation logic
- Provide alternatives to direct constructors
- Useful when object creation is complex or resource-intensive

## Common Scenarios

| Pattern | When to Use |
| --- | --- |
| 🔒 Singleton | Shared resources like configuration, logging, or database connections |
| 🏭 Factory Method | Creating objects where the exact type is determined by subclasses |
| 🧱 Builder | Constructing complex objects with many optional parameters step-by-step |
| 🏢 Abstract Factory | Creating families of related objects without specifying concrete classes |
| 🐑 Prototype | Cloning existing objects when creation is expensive or complex |
