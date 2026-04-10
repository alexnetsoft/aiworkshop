# AI Workshop - OpenEdge ABL Project

This project demonstrates the ABL Business Entity Architecture pattern for data access and business logic in OpenEdge applications.

## Architecture

The project uses a modern Object-Oriented ABL architecture with:

- **EntityFactory**: Singleton factory for creating entity instances
- **BusinessEntity**: Base class providing common data access functionality
- **Entity Classes**: Domain-specific business logic (CustomerEntity, ItemEntity)
- **Dataset/Temp-table Pattern**: For data transfer between UI and business layer

## Key Components

- `src/business/EntityFactory.cls` - Factory for entity instances
- `src/business/CustomerEntity.cls` - Customer business entity
- `src/business/ItemEntity.cls` - Item business entity
- `src/CustomerWin.w` - Customer management UI
- `src/ItemWin.w` - Item management UI
- `doc/business-entity-pattern.md` - Architecture documentation

## Database

The project uses the sports2000 database schema.

## Getting Started

See `doc/business-entity-pattern.md` for detailed architecture documentation and refactoring guidelines.