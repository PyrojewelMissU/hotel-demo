# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-module Spring Boot hotel management system demonstrating Elasticsearch integration with event-driven data synchronization via RabbitMQ. The system consists of two independent microservices:

- **hotel-admin**: Administrative service for hotel data CRUD operations (port 8099)
- **hotel-search**: Search service providing Elasticsearch-powered hotel search capabilities (port 8089)

## Architecture

### Data Flow Pattern

The project implements an event-driven architecture where data modifications flow from MySQL → RabbitMQ → Elasticsearch:

1. **hotel-admin** manages hotel data in MySQL and publishes change events to RabbitMQ
2. **hotel-search** consumes these events and synchronizes data to Elasticsearch
3. Search queries are served directly from Elasticsearch without touching MySQL

### Message Queue Integration

RabbitMQ ensures data consistency between MySQL (hotel-admin) and Elasticsearch (hotel-search):

- **Topic Exchange**: `hotel.topic` ([MqConstants.java](hotel-admin/src/main/java/com/amazecode/hotel/constants/MqConstants.java:8))
- **Insert/Update Queue**: `hotel.insert.queue` with routing key `hotel.insert`
- **Delete Queue**: `hotel.delete.queue` with routing key `hotel.delete`

When hotel data is created, updated, or deleted in hotel-admin, only the hotel ID is sent via RabbitMQ. The hotel-search service receives the ID and queries MySQL to fetch full data for Elasticsearch indexing.

### Elasticsearch Configuration

The RestHighLevelClient is manually configured in [HotelSearchApplication.java](hotel-search/src/main/java/com/amazecode/hotel/HotelSearchApplication.java:20-24) connecting to `http://192.168.100.129:9200`.

### Data Model

- **Hotel**: MySQL entity for persistent storage
- **HotelDoc**: Elasticsearch document model with additional fields (distance, isAD flag, suggestion field for autocomplete)
- The `HotelDoc` constructor transforms `Hotel` to include derived fields like location (lat/lon concatenation) and suggestion array (brand + business areas)

## Build and Run Commands

### Build the entire project
```bash
mvn clean install
```

### Run hotel-admin service
```bash
cd hotel-admin
mvn spring-boot:run
```

### Run hotel-search service
```bash
cd hotel-search
mvn spring-boot:run
```

### Run tests for a specific module
```bash
# For hotel-search tests
cd hotel-search
mvn test

# Run a single test class
mvn test -Dtest=HotelIndexTest

# Run a specific test method
mvn test -Dtest=HotelIndexTest#createHotelIndex
```

### Package modules individually
```bash
mvn clean package -pl hotel-admin
mvn clean package -pl hotel-search
```

## Development Configuration

### Database Setup

Both services connect to MySQL database `itheima` on localhost:3306 (credentials: root/123456). Update [hotel-admin/src/main/resources/application.yaml](hotel-admin/src/main/resources/application.yaml:4-8) and [hotel-search/src/main/resources/application.yaml](hotel-search/src/main/resources/application.yaml:4-9) if your environment differs.

### RabbitMQ Setup

RabbitMQ is expected at 192.168.100.129:5672 (credentials: admin/admin123). Both services must connect to the same RabbitMQ instance for data synchronization to work.

### Elasticsearch Setup

Elasticsearch must be running at 192.168.100.129:9200 (version 7.17.6). Update the hardcoded URL in [HotelSearchApplication.java](hotel-search/src/main/java/com/amazecode/hotel/HotelSearchApplication.java:22) if needed.

## Key Implementation Details

### Search Service Features (hotel-search)

The [HotelService](hotel-search/src/main/java/com/amazecode/hotel/service/impl/HotelService.java) implements:

- **Complex search queries**: Multi-field search with filters (city, brand, star rating, price range)
- **Geo-distance sorting**: Results can be sorted by distance from user location
- **Function score queries**: Ads (isAD=true) are boosted by factor of 10 ([HotelService.java:244-257](hotel-search/src/main/java/com/amazecode/hotel/service/impl/HotelService.java#L244-L257))
- **Aggregations**: Dynamic filter options based on current search results
- **Autocomplete**: Completion suggester using the `suggestion` field in HotelDoc

### Event Listeners

The [HotelListener](hotel-search/src/main/java/com/amazecode/hotel/mq/HotelListener.java) processes RabbitMQ messages:
- Insert/update events trigger fetching hotel data from MySQL and indexing to Elasticsearch
- Delete events remove documents from Elasticsearch by ID

### MQ Configuration

MqConfig is only defined in hotel-search ([MqConfig.java](hotel-search/src/main/java/com/amazecode/hotel/config/MqConfig.java)) as it's the consumer. hotel-admin only publishes messages and doesn't need queue/binding definitions.

## Technology Stack

- Spring Boot 2.7.17
- Java 11
- MyBatis Plus 3.5.4 (pagination, CRUD operations)
- Elasticsearch 7.17.6 (RestHighLevelClient)
- RabbitMQ (AMQP)
- MySQL 8.0.33
- Hutool 5.8.12 (utility library for JSON handling)
- Lombok (boilerplate reduction)

## Important Notes

- The Elasticsearch index name is hardcoded as "hotel" throughout the codebase
- hotel-admin sends only hotel IDs via MQ (not full objects) to minimize message size
- Both services share the same MySQL database and can independently query hotel data
- The RestHighLevelClient is deprecated in newer ES versions; consider migrating to Java API Client for production use
