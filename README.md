# Run Container

```shell
docker-compose up -d
```

# Exec SQL

## Option 1

```shell
docker-compose exec postgres bash

psql -U postgres wakarimi

\i /tmp/test.sql
```


## Option 2

```shell
docker-compose exec postgres bash

psql -U postgres wakarimi < /tmp/test.sql
```

## Option 3

```shell
docker-compose exec postgres bash -c 'psql -U postgres wakarimi -c "\i /tmp/test.sql"'
```

## Option 4

```shell
docker-compose exec postgres bash -c 'psql -U postgres wakarimi < /tmp/test.sql'
```

## Option 5

- use [Postico](https://eggerapps.at/postico/)

## Option6

- use [SQL Tools(VS Code)](https://marketplace.visualstudio.com/items?itemName=mtxr.sqltools-driver-pg)
