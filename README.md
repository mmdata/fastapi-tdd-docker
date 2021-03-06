# About this application
It is entirely based on the course https://testdriven.io/courses/tdd-fastapi/getting-started/ with changelog 1.2.0

![Continuous Integration and Delivery](https://github.com/mmdata/fastapi-tdd-docker/workflows/Continuous%20Integration%20and%20Delivery/badge.svg?branch=master)

## Running the Docker container
It might happen that we have an error saying that there is no space to build the docker image. In this case do:
- `docker system prune`


First we have to make the file project/entrypoint.sh executable:
- `chmod +x project/entrypoint.sh`
Then we build:
- `docker-compose build`
Run the container (erase the d to not be in detached mode):
- `docker-compose up -d`  
check:
- `http://localhost:5004/ping`

To do the build and run in one commands:
- `docker-compose up -d --build`

To bring down the container and volume:
- `docker-compose down -v`

IMPORTANT: if the relation elation "textsummary" is not created we can ssh the container and do it manually:
- `docker exec -it <container_ID> bash`
- `python3 app/db.py`

or directly:
- `docker-compose exec web python3 app/db.py`

To test that the database is initialiased and we can actually create a new summary:
- `http --json POST http://localhost:8004/summaries/ url=http://testdriven.io`

## Checking the logs
- `docker-compose logs web`

## Access Database with psql
- `docker-compose exec web-db psql -U postgres`
Then:
- `\c web_dev`
- `\dt`
- `\q`

## Running the tests
With the container up and running execute:
- `docker-compose exec web python -m pytest -v`
To run the tests with coverage:
- `docker-compose exec web python -m pytest --cov="."`
Run tests with coverage and generate html report:
- `docker-compose exec web python -m pytest --cov="." --cov-report html`

## Running the app without Docker
- `uvicorn app.main:app --port 5000`
To enable autoreload:
- `uvicorn app.main:app --port 5000 --reload`
Or:
- `python3 app/main.py`

## Running the deployed app
For the docs:
- `https://<app-name>.herokuapp.com/docs`

## Heroku

Log in to Heroku Container Registry:
- `heroku container:login`
Provision a new Postgres database with the hobby-dev plan:
- `heroku addons:create heroku-postgresql:hobby-dev --app <app-name>`
Build production image:
- `docker build -f project/Dockerfile.prod -t registry.heroku.com/<app-name>/web ./project`
To test locally (using database on heroku):
- `docker run --name fastapi-tdd -e PORT=8765 -e DATABASE_URL=sqlite://sqlite.db -p 5003:8765 registry.heroku.com/<app-name>/web:latest`
navigate to http://localhost:5003/ping/
Push the image to the registry:
- `docker push registry.heroku.com/<app-name>/web:latest`
Release the image:
- `heroku container:release web --app <app-name>`
Apply database migration:
- `heroku run python app/db.py --app <app-name>`

Create a new summary:
- `http --json POST https://<app-name>.herokuapp.com/summaries/ url=https://testdriven.io`

# Formatting and sorting
Check what changes will be made:
- `docker-compose exec web black . --check`
Format files
- `docker-compose exec web black .`
To check what the ordering will do:
- `docker-compose exec web isort . --check-only`
Apply the changes:
- `docker-compose exec web isort .`

# To Do
- Create cli.sh
- add proper logging
- add prometheus endpoints