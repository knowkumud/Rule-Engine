# SprintSight — Sprint Velocity Forecaster

Answers the question every PM asks: **"When will this epic be done?"**

Runs three forecast models in parallel — average velocity, weighted recent, and Monte Carlo simulation — and returns confidence intervals, not just a single date.

## Architecture

```
┌───────────────────────────────────────────┐
│     forecaster-api (Spring Boot REST)     │
│   Controllers │ DTOs │ Swagger │ Store    │
├───────────────────────────────────────────┤
│     forecaster-core (Pure Java — 0 deps)  │
│  Engine │ Strategies │ Pipeline │ Models  │
└───────────────────────────────────────────┘
```

## Java Concepts Demonstrated

| Concept | Where |
|---------|-------|
| **Strategy pattern** | `ForecastStrategy` → 3 interchangeable algorithms |
| **Template Method** | `ForecastPipeline` → abstract class with `final` template |
| **Builder pattern** | `SimulationConfig`, `Epic`, `ForecastEngine` |
| **Enums with behavior** | `EpicStatus` with `canTransitionTo()`, `validateTransition()` |
| **Generics** | `DataSeries<T extends Number>` — works with Integer, Double, Long |
| **Records** | `SprintSnapshot`, `ForecastResult`, `ForecastComparison` |
| **CompletableFuture** | Parallel execution of 3 strategies simultaneously |
| **ThreadLocalRandom** | Thread-safe RNG for Monte Carlo (not `Math.random()`) |
| **Immutability** | Unmodifiable collections, defensive copies, no setters |
| **ConcurrentHashMap** | Thread-safe in-memory store |
| **Multi-module Maven** | Core (0 Spring deps) + API (Spring Boot wrapper) |

## Quick Start

```bash
cd forecaster-api
mvn spring-boot:run
# Swagger UI: http://localhost:8080/swagger-ui.html
```

Sample data is auto-seeded on startup (6 sprints, 2 epics).

## API Examples

### Compare all forecast models
```bash
curl -X POST http://localhost:8080/api/v1/forecast \
  -H "Content-Type: application/json" \
  -d '{"epicId": "EPIC-001"}'
```

### Single strategy (Monte Carlo)
```bash
curl -X POST http://localhost:8080/api/v1/forecast \
  -d '{"epicId": "EPIC-001", "strategy": "MONTE_CARLO", "iterations": 10000}'
```

### With outlier removal
```bash
curl -X POST http://localhost:8080/api/v1/forecast \
  -d '{"epicId": "EPIC-001", "removeOutliers": true}'
```

### Velocity statistics
```bash
curl http://localhost:8080/api/v1/velocity/stats
```

### Add sprint data
```bash
curl -X POST http://localhost:8080/api/v1/sprints \
  -d '{"sprintName":"Sprint 7","startDate":"2025-03-31","endDate":"2025-04-11","committedPoints":22,"completedPoints":20,"carryOverPoints":2}'
```

### List strategies
```bash
curl http://localhost:8080/api/v1/forecast/strategies
```

## Running Tests

```bash
mvn test    # 25 tests — strategies, pipeline, engine, immutability
```
