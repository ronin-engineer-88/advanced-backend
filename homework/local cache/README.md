# Local cache

## Setup database

- `docker-compose up -d`

## Explain the local cache

### Flow

PublicAirportController -> AirportService -> (Local cache)AirportRepository -> Postgres

### Code

- @Cacheable: Chỉ định rằng kết quả của phương thức findAll() sẽ được lưu vào bộ nhớ cache.
- cacheNames = "localCache": Xác định tên cache được sử dụng, ở đây là "localCache".
  - cacheManager = "localCacheManager": Chỉ định CacheManager sử dụng để quản lý bộ nhớ cache(Caffeine cache), Xem thêm cấu hình về LocalCacheManager trong class [LocalCacheConfig](../../src/main/java/dev/ronin_engineer/software_development/infrastructure/config/LocalCacheConfig.java).
- key = "#root.methodName":
    + Xác định key của cache là tên của phương thức (findAll).
    + #root.methodName sẽ tự động lấy tên phương thức đang được gọi, tức là "findAll".

```java

@Cacheable(key = "#root.methodName", cacheNames = "localCache", cacheManager = "localCacheManager")
public List<Airport> findAll() {
    return airportMapper.findAll();
}
```

### Local cache(Read aside)

- API `GET /api/public/airports` [airport-requests](./airport-requests.http)

1. Đầu tiên, dữ liệu được lấy từ Postgres và lưu vào local cache khi gọi API lần đầu tiên.

```
2025-02-02T20:25:07.993+07:00 TRACE 23581 --- [nio-8080-exec-1] o.s.cache.interceptor.CacheInterceptor   : Computed cache key 'findAll' for operation Builder[public java.util.List dev.ronin_engineer.software_development.infrastructure.repository.AirportRepositoryImpl.findAll()] caches=[localCache] | key='#root.methodName' | keyGenerator='' | cacheManager='localCacheManager' | cacheResolver='' | condition='' | unless='' | sync='false'
2025-02-02T20:25:07.993+07:00 TRACE 23581 --- [nio-8080-exec-1] o.s.cache.interceptor.CacheInterceptor   : No cache entry for key 'findAll' in cache(s) [localCache]
2025-02-02T20:25:07.993+07:00 TRACE 23581 --- [nio-8080-exec-1] o.s.cache.interceptor.CacheInterceptor   : Computed cache key 'findAll' for operation Builder[public java.util.List dev.ronin_engineer.software_development.infrastructure.repository.AirportRepositoryImpl.findAll()] caches=[localCache] | key='#root.methodName' | keyGenerator='' | cacheManager='localCacheManager' | cacheResolver='' | condition='' | unless='' | sync='false'
2025-02-02T20:25:08.000+07:00  INFO 23581 --- [nio-8080-exec-1] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2025-02-02T20:25:08.066+07:00  INFO 23581 --- [nio-8080-exec-1] com.zaxxer.hikari.pool.HikariPool        : HikariPool-1 - Added connection org.postgresql.jdbc.PgConnection@2a350f80
2025-02-02T20:25:08.067+07:00  INFO 23581 --- [nio-8080-exec-1] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2025-02-02T20:25:08.070+07:00 DEBUG 23581 --- [nio-8080-exec-1] d.r.s.i.m.airport.AirportMapper.findAll  : ==>  Preparing: SELECT airport_id, airport_name, status, version, created_at, created_by, updated_at, updated_by, data FROM airport
2025-02-02T20:25:08.076+07:00 DEBUG 23581 --- [nio-8080-exec-1] d.r.s.i.m.airport.AirportMapper.findAll  : ==> Parameters: 
2025-02-02T20:25:08.084+07:00 DEBUG 23581 --- [nio-8080-exec-1] d.r.s.i.m.airport.AirportMapper.findAll  : <==      Total: 3
```

2. Tiếp theo, dữ liệu được lấy từ local cache khi gọi API các lần tiếp theo.

```
2025-02-02T20:25:19.991+07:00 TRACE 23581 --- [nio-8080-exec-2] o.s.cache.interceptor.CacheInterceptor   : Computed cache key 'findAll' for operation Builder[public java.util.List dev.ronin_engineer.software_development.infrastructure.repository.AirportRepositoryImpl.findAll()] caches=[localCache] | key='#root.methodName' | keyGenerator='' | cacheManager='localCacheManager' | cacheResolver='' | condition='' | unless='' | sync='false'
2025-02-02T20:25:19.991+07:00 TRACE 23581 --- [nio-8080-exec-2] o.s.cache.interceptor.CacheInterceptor   : Cache entry for key 'findAll' found in cache 'localCache'
```

3. Response trả về.

```json
{
  "meta": {
    "code": "SUCCESS",
    "type": null,
    "message": null,
    "request_id": null,
    "service_id": "airlines-service",
    "extra_meta": null
  },
  "data": [
    {
      "airport_id": "JFK",
      "airport_name": "John F. Kennedy International Airport",
      "data": "{\"city\": \"New York\", \"country\": \"USA\"}",
      "status": 1,
      "version": 1,
      "created_at": "2022-12-31T17:00:00.000+00:00",
      "created_by": "admin",
      "updated_at": "2022-12-31T17:00:00.000+00:00",
      "updated_by": "admin"
    },
    {
      "airport_id": "LAX",
      "airport_name": "Los Angeles International Airport",
      "data": "{\"city\": \"Los Angeles\", \"country\": \"USA\"}",
      "status": 1,
      "version": 1,
      "created_at": "2022-12-31T17:00:00.000+00:00",
      "created_by": "admin",
      "updated_at": "2022-12-31T17:00:00.000+00:00",
      "updated_by": "admin"
    },
    {
      "airport_id": "ORD",
      "airport_name": "Hare International Airport",
      "data": "{\"city\": \"Chicago\", \"country\": \"USA\"}",
      "status": 1,
      "version": 1,
      "created_at": "2022-12-31T17:00:00.000+00:00",
      "created_by": "admin",
      "updated_at": "2022-12-31T17:00:00.000+00:00",
      "updated_by": "admin"
    }
  ]
}
```