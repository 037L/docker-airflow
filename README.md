# docker-airflow

Run Apache Airflow workflows in Docker without installing Airflow directly on your host. Use a single container for a SequentialExecutor setup, or the supplied Compose configurations for LocalExecutor and CeleryExecutor.

## Features

- Run Airflow with SequentialExecutor, LocalExecutor, or CeleryExecutor.
- Use PostgreSQL as the metadata backend and Redis as the Celery task queue.
- Install extra Airflow packages and Python dependencies through Docker build arguments.
- Mount custom DAGs and plugins into the Airflow container.
- Set Airflow configuration and connection URIs through environment variables.

## Installation

Install [Docker](https://docs.docker.com/get-docker/) and [Docker Compose](https://docs.docker.com/compose/install/). From the root of this repository, build a local image:

```sh
docker build --rm -t docker-airflow .
```

To include additional dependencies at build time:

```sh
docker build --rm --build-arg AIRFLOW_DEPS="datadog,dask" --build-arg PYTHON_DEPS="flask_oauthlib>=0.9" -t docker-airflow .
```

Choose dependencies compatible with the Airflow version in the image.

## Usage

Start the default SequentialExecutor setup:

```sh
mkdir -p dags plugins
docker run -d --name docker-airflow -p 8080:8080 \
  -v "$(pwd)/dags:/usr/local/airflow/dags" \
  -v "$(pwd)/plugins:/usr/local/airflow/plugins" \
  docker-airflow webserver
```

Open [localhost:8080](http://localhost:8080/). Place your DAG files in `dags/` and custom plugins in `plugins/`.

Example DAGs are disabled by default. Add `-e LOAD_EX=y` to the run command to enable them.

For another executor, first set the Airflow image references in the corresponding Compose file to `docker-airflow:latest`. Then run one of:

```sh
docker-compose -f docker-compose-LocalExecutor.yml up -d
docker-compose -f docker-compose-CeleryExecutor.yml up -d
```

Airflow settings accept environment variables in the form `AIRFLOW__<SECTION>__<KEY>`. Connections accept `AIRFLOW_CONN_<CONNECTION_ID>` with a connection URI as the value.

**Use the same Fernet key across containers** when storing encrypted connection passwords with LocalExecutor or CeleryExecutor. Generate a key with:

```sh
docker run --rm docker-airflow python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

Set the resulting value as `AIRFLOW__CORE__FERNET_KEY` for each Airflow container in your Compose configuration. Keep the key private and stable across restarts.

## License

Licensed under Apache-2.0. This project is based on [puckel/docker-airflow](https://github.com/puckel/docker-airflow). Retain the upstream copyright and license notices when redistributing derived code.

