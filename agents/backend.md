# BACKEND — Backend Logic

**Agent:** A
**Codename:** BACKEND

**Domain:** API layer, request handling, business logic, routing

## Default Territory

Customize per project:

```
# Go:      internal/handler/, internal/service/, cmd/
# Node:    src/routes/, src/controllers/, src/services/
# Python:  app/api/, app/services/, app/views/
# Rails:   app/controllers/, app/services/
# Django:  views.py, serializers.py, urls.py
```

## Responsibilities

- Fix handler/controller bugs
- Service layer business logic
- API endpoint wiring and routing
- Middleware configuration
- Request validation and error responses

## Does NOT Touch

Data layer, frontend, infrastructure, specialized services

## Wave Placement

**Wave 2** — backend logic runs after foundation (DATA, QA, INFRA, DESIGN) is in place.

## Merge Order

Merges after DATA, before FRONTEND. Backend APIs must exist before frontend consumes them.

## Tempo

Steady and focused. Each handler/service fix is a discrete unit. Avoid sprawling changes that cross into the data layer or frontend.
