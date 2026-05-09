# ShortLink---Distributed-URL-Shortener
URL Shortener

A horizontally-scalable URL shortener built with Spring Boot, base62 encoding, and Snowflake ID generation. Sustains 12K req/s on a 3-node cluster with a Redis read-through cache achieving a 94% hit rate, dropping p99 redirect latency from 180ms to 11ms. Click analytics are streamed via Kafka with exactly-once delivery using the Outbox pattern.
Features

POST /shorten — create a short URL with optional custom alias and TTL
GET /{code} — 302 redirect with sub-12ms p99 latency
Snowflake ID generation + base62 encoder for collision-free short codes
Redis read-through cache with cache-aside on writes
Kafka producer → consumer analytics pipeline (click events → PostgreSQL)
Outbox pattern for transactional event publishing
k6 load test scripts included (smoke / load / stress profiles)
GitHub Actions CI: build → test → dockerize

Tech Stack
LayerTechnologyLanguageJava 21FrameworkSpring Boot 3Primary DBPostgreSQL 16CacheRedis 7MessagingApache Kafka 3.6ID GenerationSnowflake (custom)ContainerizationDocker + Docker ComposeCI/CDGitHub ActionsLoad Testingk6
Architecture
graph LR
    Client -->|POST /shorten| API
    Client -->|GET /{code}| API
    API -->|cache miss| Postgres
    API -->|cache hit/write| Redis
    API -->|click event| Kafka
    Kafka --> AnalyticsConsumer
    AnalyticsConsumer --> Postgres
Quick Start
Prerequisites: Java 21+, Docker, Docker Compose
bash# 1. Clone
git clone https://github.com/nidhisharma0160/shortlink-distributed.git
cd shortlink-distributed

# 2. Start infrastructure
docker compose up -d postgres redis kafka zookeeper

# 3. Run the application
./mvnw spring-boot:run

# 4. Shorten a URL
curl -X POST http://localhost:8080/shorten \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com", "alias": "demo", "ttlDays": 7}'

# 5. Follow the redirect
curl -L http://localhost:8080/demo

# 6. Run load test
k6 run tests/load/load-test.js
Environment variables (copy .env.example → .env):
POSTGRES_URL=jdbc:postgresql://localhost:5432/shortlink
REDIS_HOST=localhost
KAFKA_BOOTSTRAP_SERVERS=localhost:9092
API Reference
MethodEndpointDescriptionPOST/shortenCreate a short URLGET/{code}Redirect to original URLGET/api/stats/{code}Click analytics for a codeDELETE/api/{code}Delete a short URL
Running Tests
bash# Unit + integration tests (uses Testcontainers)
./mvnw test

# Load test (requires running app)
k6 run tests/load/load-test.js
