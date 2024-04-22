### Task-2

#### Resolve configuration issue 

- Updated `HASURA_GRAPHQL_ENABLE_CONSOLE: "true"`
- Updated graphql image `hasura/graphql-engine:v2.38.0`
- Updated the graphql internal port `"8111:8080"`
- Added Chinook dataset to Postgres volumes
- Updated the postgress database URL
- Added admin security password


```
version: "3.6"
services:
  redis:
    image: redis:latest
    restart: always
    ports:
      - "6379:6379"
  postgres:
    image: postgres:15
    restart: always
    ports:
      - "5432:5432"
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./chinook-database/ChinookDatabase/DataSources/Chinook_PostgreSql.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql.sql
      - ./chinook-database/ChinookDatabase/DataSources/Chinook_PostgreSql_AutoIncrementPKs.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql_AutoIncrementPKs.sql
      - ./chinook-database/ChinookDatabase/DataSources/Chinook_PostgreSql_SerialPKs.sql:/docker-entrypoint-initdb.d/Chinook_PostgreSql_SerialPKs.sql
    environment:
      POSTGRES_PASSWORD: postgrespassword
  
  graphql-engine:
    image: hasura/graphql-engine:v2.38.0
    ports:
      - "8111:8080"
    restart: always
    environment:
      HASURA_GRAPHQL_ADMIN_SECRET: "admin"
      HASURA_GRAPHQL_METADATA_DATABASE_URL: postgres://postgres:postgrespassword@postgres:5432/postgres
      HASURA_GRAPHQL_ENABLE_CONSOLE: "true"
      HASURA_GRAPHQL_DEV_MODE: "true"
      HASURA_GRAPHQL_REDIS_URL: "redis://redis:6379"
      HASURA_GRAPHQL_RATE_LIMIT_REDIS_URL: "redis://redis:6379"
      PG_DATABASE_URL: postgres://postgres:postgrespassword@postgres:5432/postgres

volumes:
  db_data:

```

```
docker compose up -d
```

login into the Postgres container and load datasets
```
docker exec -it <postgres-container-id> bash

psql -U postgres -f Chinook_PostgreSql.sql
psql -U postgres -f Chinook_PostgreSql_AutoIncrementPKs.sql
psql -U postgres -f Chinook_PostgreSql_SerialPKs.sql

```

### Graphql queries

Share your query, the headers used and the results of the following three queries:

Added genre relationship with track table and fix the query from `id` > `track_id`

```
query getTracks($genre: String, $limit: Int, $offset: Int) {
track(limit: $limit, offset: $offset, where: {genre: {name: {_eq: $genre}}})
{
name
track_id
}
}

```
![task2-1](https://github.com/learnwithabi/hasura/assets/66668461/f39e0947-bfa8-4573-8904-dbb3155898c1)


Fixed the query from `albums` to `album`

```
query getAlbumsAsArtist{
album {
title
}
}

```
![task2-2](https://github.com/learnwithabi/hasura/assets/66668461/88de49d8-28a2-4799-bada-32837d823f11)


Fixed the query from `tracks_aggregate` to `track_aggregate`

```
query trackValue {
track_aggregate {
aggregate {
sum {
unit_price
}
}
}
}

```
![task2-3](https://github.com/learnwithabi/hasura/assets/66668461/0fc4f691-b699-4594-9f23-51a006cecde4)












