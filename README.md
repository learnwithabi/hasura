### Task 1

#### Prerequisites

```
curl https://raw.githubusercontent.com/hasura/graphql-engine/stable/install-manifests/docker-compose/docker-compose.yaml -o docker-compose.yml
```

Refer the chinook database [compose file](https://github.com/lerocha/chinook-database/blob/master/docker-compose.yml) and updated our Postgres volume to copy the datasets
```
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

#### GraphQl Queries

**1. How many artists are in the database?**
```
query MyQuery {
  artist_aggregate {
    aggregate {
      count
    }
  }
}
```
![1](https://github.com/learnwithabi/hasura/assets/66668461/630bdc28-618b-4274-ac9a-0b6a11268cde)


**2. List the first track of every album by every artist in ascending order.**

We can't get the result directly. We have to set a relationship with the album, track and artist

```
query MyQuery {
  album(order_by: { artist_id: asc }) {
    artist_id
    album_id
    title
    tracks(limit: 1, order_by: { name: asc }) {
      track_id
      name
    }
    artist {
      name
    }
  }
}

```
![2](https://github.com/learnwithabi/hasura/assets/66668461/8c166fae-22a7-4802-8dea-ad719437beb6)

**3. Get all albums for artist id = 5 without specifying a where clause.**

We can't get the result directly. We have to set a relationship with the album, track and artist

```
query MyQuery {
  artist(where: {artist_id: {_eq: 5}}) {
    artist_id
    name
    albums {
      album_id
      title
    }
  }
}

```

![3](https://github.com/learnwithabi/hasura/assets/66668461/ede21094-a6d6-4f52-98cd-ea5d93bb7b29)

**4. Using a GraphQL mutation, add your favorite artist and one of their albums that isn’t in the dataset.**

```
mutation {
  insert_artist_one(object: {
    artist_id: 277
    name: "test_artist1"
  }) {
    artist_id
  }
}

```
![4](https://github.com/learnwithabi/hasura/assets/66668461/5fb30fb8-21b6-4f66-b07f-94ece4ceb6be)

**5. How did you identify which ID to include in the mutation?**

```
query MyQuery {
  artist_aggregate {
    aggregate {
      count
    }
  }
}
```

![5](https://github.com/learnwithabi/hasura/assets/66668461/36ec64d8-8e76-4394-8e38-5372a2214925)


#### SQL Queries

**1. Return the artist with the most number of albums**

```
SELECT artist_id, COUNT(*) AS album_count
FROM album
GROUP BY artist_id
ORDER BY album_count DESC
LIMIT 1;

```
![sql1](https://github.com/learnwithabi/hasura/assets/66668461/e7705233-8f7b-46cd-ad72-ff538767ba70)


**2. Return the top three genres found in the dataset in descending order**

```
SELECT genre.name AS genre_name, COUNT(*) AS genre_count
FROM genre
INNER JOIN track ON genre.genre_id = track.genre_id
GROUP BY genre.genre_id
ORDER BY COUNT(*) DESC
LIMIT 3;

```
![sql2](https://github.com/learnwithabi/hasura/assets/66668461/e0aeac5d-3d25-4b32-9655-11c34ba24374)


**3. Return the number of tracks and average run time for each media type**
```
SELECT
    mt.name AS media_type,
    COUNT(t.track_id) AS num_tracks,
    AVG(t.milliseconds) AS avg_run_time
FROM
    track t
JOIN
    media_type mt ON t.media_type_id = mt.media_type_id
GROUP BY
    mt.name;

```
![sql3](https://github.com/learnwithabi/hasura/assets/66668461/98740be5-192a-4147-a5ad-d28d37ce54b9)








