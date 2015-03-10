Development Environment
====================

The development environment uses [Docker](http://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/). This makes it super easy to get up and running with a configuration that would closely mirror production. A [Vagrant](http://www.vagrantup.com/) config file is included for operating systems that cannot yet run Docker natively.

With Docker/Compose/Vagrant installed, use the following steps to launch for the first time:

* `vagrant up` - from within the project root to build the Ubuntu VM with Docker and Fig automatically installed. If you're already using Linux, you can skip this step and just install Docker and Fig on your system.
* `docker-compose up` to start the web app. This will download and provision two containers: one running PostgreSQL and one running the Flask app. This will take a while, but once it completes subsequent launches will be much faster. (NOTE: if you are using the Vagrant VM that was provisioned in the first step, change into the `/vagrant` directory before running `docker-compose up`.)
* When `docker-compose up` completes, the app should be accessible at [http://127.0.0.1:5000](http://127.0.0.1:5000). (NOTE: if running commands within the Vagrant VM that was provisioned in the first step, the app can be found at: [http://192.168.13.81:5000](http://192.168.13.81:5000))


Environment Variables
====================

There are just a couple of configurations managed as environment variables. In the development environment, these are injected by Docker Compose and managed in the `docker-compose.yml` file.

* `DATABASE_URL` - This is the connection URL for the PostgreSQL database. It is not used in the **development environment**.
* `DEBUG` - This toggle debug mode for the app to True/False.
* `SECRET_KEY` - This is a secret string that you make up. It is used to encrypt and verify the authentication token on routes that require authentication.


Project Organization
====================

* Application-wide settings are stored in `config.py` at the root of the repository. These items are accessible on the `config` dictionary property of the `app` object. Example: `debug = app.config['DEBUG']`
* The directory `/app` contains the API application
* URL mapping is managed in `/app/routes.py`
* Functionality is organized in packages. Example: `/app/users` or `/app/utils`.
* Tests are contained in each package. Example: `app/users/tests.py`


Running Tests
====================

Tests are ran with [nose](https://nose.readthedocs.org/en/latest/) from inside the `docker-compose` web container:

```
$ docker-compose run web nosetests -v
```


Database Migrations
====================

Migrations for the provided models are part of the seed project. To generate new migrations use `Flask-Migrate`:

    $ docker-compose run web python run.py db migrate
    $ docker-compose run web python run.py db upgrade


API Routes
====================

This API uses token-based authentication. A token is obtained by registering a new user (`/api/v1/user`) or authenticating an existing user (`/api/v1/authenticate`). Once the client has the token, it must be included in the `Authorization` header of all requests.


### Register a new user

**POST:**
```
/api/v1/user
```

**Body:**
```json
{
    "email": "something@email.com",
    "password": "123456"
}
```

**Response:**
```json
{
    "id": 2,
    "token": "eyJhbGciOiJIUzI1NiIsImV4cCI6MTQxMDk2ODA5NCwiaWF0IjoxNDA5NzU4NDk0fQ.eyJpc19hZG1pbiI6ZmFsc2UsImlkIjoyLCJlbWFpbCI6InRlc3QyQHRlc3QuY29tIn0.goBHisCajafl4a93jfal0sD5pdjeYd5se_a9sEkHs"
}
```

**Status Codes:**
* `201` if successful
* `400` if incorrect data provided
* `409` if email is in use


### Get the authenticated user

**GET:**
```
/api/v1/user
```

**Response:**
```json
{
    "id": 2,
    "email": "test2@test.com",
}
```

**Status Codes:**
* `200` if successful
* `401` if not authenticated


### Authenticate a user

**POST:**
```
/api/v1/authenticate
```

**Body:**
```json
{
    "email": "something@email.com",
    "password": "123456"
}
```

**Response:**
```json
{
    "id": 2,
    "token": "eyJhbGciOiJIUzI1NiIsImV4cCI6MTQxMDk2ODA5NCwiaWF0IjoxNDA5NzU4NDk0fQ.eyJpc19hZG1pbiI6ZmFsc2UsImlkIjoyLCJlbWFpbCI6InRlc3QyQHRlc3QuY29tIn0.goBHisCajafl4a93jfal0sD5pdjeYd5se_a9sEkHs"
}
```

**Status Codes:**
* `200` if successful
* `401` if invalid credentials
