# ShortLink---Distributed-URL-Shortener
URL Shortener

A horizontally-scalable URL shortener built with Spring Boot, base62 encoding, and Snowflake ID generation. Sustains 12K req/s on a 3-node cluster with a Redis read-through cache achieving a 94% hit rate, dropping p99 redirect latency from 180ms to 11ms. Click analytics are streamed via Kafka with exactly-once delivery using the Outbox pattern.
Features

