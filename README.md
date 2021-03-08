[![Build Status](https://dev.azure.com/ianpurton0711/onchain.io/_apis/build/status/contor-systems.contor?branchName=master)](https://dev.azure.com/ianpurton0711/onchain.io/_build/latest?definitionId=18&branchName=master) ![Docker Pulls](https://img.shields.io/docker/pulls/contorsystems/contor?style=plastic)

# Quickly add user registration and logon to any application

Contor is a proxy you put in front of your application. It then intercepts requests and will show a logon or registration page to the user. Contor uses your database and requires minimal configuration.

## Quickstart 

Cut and paste the following into a docker-compose.yml

```yaml
version: '3.4'
services:

  db:
    image: postgres:alpine
    environment:
      POSTGRES_PASSWORD: testpassword
      POSTGRES_USER: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  nginx:
    image: nginx:stable-alpine

  contor:
    image: ianpurton/barn-door
    environment:
        SECRET_KEY: 190a5bf4b3cbb6c0991967ab1c48ab30790af876720f1835cbbf3820f4f5d949
        DATABASE_URL: postgresql://postgres:testpassword@db:5432
        FORWARD_URL: nginx
        FORWARD_PORT: 80
        REDIRECT_URL: '/'
    ports:
      - "9090:9090"
    depends_on:
      db:
        condition: service_healthy
```

Bring up the services

```console
docker-compose up
```

We need to add a user table to our database. Run the psql command line from docker-compose

```console
docker-compose run db psql postgres://postgres:testpassword@localhost:5432
```

Once you have the psql command prompt you can cut and paste the following code to create a users table.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY, 
    email VARCHAR NOT NULL UNIQUE, 
    hashed_password VARCHAR NOT NULL, 
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

Go to localhost:9090 in your browser and register. You should be taken to the nginx default page.

Run the following command in the psql shell to see yourself in the database.

```sql
SELECT * FROM users;
```