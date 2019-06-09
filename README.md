# Nuxt, Rails 5, and Docker boilerplate

This project contains all you need to start developing web apps using
Vue.js/Nuxt.js on the front end, and Rails 5 in API mode as the back end.

We'll use docker to run our rails server, nuxt server, and postgres database,
so this dev environment can be used in any OS. If you mess up or something goes
wrong, feel free to delete your containers and try again - that's what they're
there for.

## Prerequisites

* [Git](https://git-scm.com/downloads)
* [Docker](https://www.docker.com/products/docker-desktop)

## Getting Started


Download the contents of this repo (except for this README) to a directory on
your computer that you want to contain your new application.

For example: `~/Projects/myapp/`

### Create database volume

Run

```bash
docker volume create --name=pgdata
```

Docker containers are ephemeral. You can destroy and recreate them as often as you
want. This volume will allow our DB data to survive such purges.

There is another way to do this: instead of creating a Docker volume, you can
use the `volumes` attribute under the `db` service, and mount a local
directory (./tmp/db):

```yaml
db:
  image: postgres
  volumes:
    - ./tmp/db:/var/lib/postgresql/data
```

You could then delete the top level `volumes` attribute:

```yaml
volumes:
  pgdata:
    external: true
```

However, due to file ownership issues, this approach won't work on Windows and
you'll need to stick with using a Docker volume.

### Create rails application

Run:
```bash
docker-compose run --no-deps rails rails new ./back-end --force --database=postgresql --skip-test --api
```

This will run the `rails new` command on our `rails` service defined in docker-compose.yml.

Flag explanations:
* **--no-deps** - Tells `docker-compose run` not to start any of the services in `depends_on`.
* **--force** - Tells rails to overwrite existing files, such as Gemfile.
* **--database=postgresql** - Tells Rails to default our db config to use postgres.
* **--skip-test** - We're going to install Rspec, so we don't need the unit test framework that comes with rails.
* **--api** - Configures Rails for api mode.

If everything went well you should have a directory full of Rails boilerplate.

### Create nuxt application

Run:
```bash
docker-compose run nuxt npx create-nuxt-app ./front-end
```

This will ask you a few questions, which you should answer based on your
preferences. See https://nuxtjs.org/guide/installation for more details.

#### Configure Nuxt to work with Docker

Change `dev` script in package.json to `HOST=0.0.0.0 nuxt` so that it is
available on the host machine

#### Configure Nuxt webpack watcher

Something about Windows prevents hot reloading from working without this configuration. Add this to your `nuxt.config.js` file:

```
watchers: {
  webpack: {
    poll: true
  }
},
```

### Configure database

Next open `./back-end/config/database.yml`.

It's already configured for postgres, but we need some additional configuration to get it to work with our `db`
container.

Change the `&default` config to match the following:
```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  # For details on connection pooling, see Rails configuration guide
  # http://guides.rubyonrails.org/configuring.html#database-pooling
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  host: db
  username: postgres
  password:
```

This uses the default username and password (which is empty) for the postgres image. It also changes `host` to point
to `db`, the name of our service.

### Re-build rails & nuxt images

Run
```bash
docker-compose build
```

Since we have a new gemfile and yarn file with our image dependencies
defined, we need to rebuild our images to install them. Any time you change
your gemfile or yarn file you'll need to rebuild.

### Start the application

Run
```bash
docker-compose up
```

This should start our rails, nuxt, and database containers, as well as run the
`command` config in docker-compose.yml, which will start our nuxt and rails
servers.

### Create database

In another terminal, run:

```bash
docker ps
```

You should see all three defined services running.

Run:

```bash
docker-compose run rails rake db:create
```

This will build our development and test databases.

### Visit application

In your browser, go to `http://localhost:3000`. You should see the nuxt
intro page.

In your browser, go to `http://localhost:8080`. You should see the rails
intro page.

### Install rspec

Follow these directions to install Rspec: https://github.com/rspec/rspec-rails#installation

Remember that you'll need to `docker-compose build` after adding the gem to your Gemfile, and you'll need to use
`docker-compose run rails` to run the `rails generate` command.

## Build Your Application

You're now ready to take your application in whatever direction you choose. 
