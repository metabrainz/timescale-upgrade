# ListenBrainz Timescale Upgrade
Holds various files and procedure to upgrade ListenBrainz Timescale installation

# Steps

- Update the old and new postgresql versions in Dockerfile.pg.upgrade
- Clone, checkout the repo locally and cd into it
- Build the docker image
  ```shell
  docker build -t meb-ts-upgrade -f Dockerfile.pg.upgrade .
  ```
- Create new volume for timescale data
  ```shell
  docker volume create --driver local listenbrainz-timescale-data-13
  ```
- Run a container from the previously built image with the old and new volumes mounted
  ```shell
  docker run -it -v listenbrainz_timescaledb:/var/lib/postgresql/old -v listenbrainz-timescale-data-13:/var/lib/postgresql/new -u postgres meb-ts-upgrade bash
  ```
- Init a new database cluster
  ```shell
  $PGBINNEW/initdb \
    --encoding utf8 \
    --locale en_US.UTF8 \
    --username postgres \
    --pgdata $PGDATANEW
  ```
- Run pg_upgrade with checks  
  ```shell
  $PGBINNEW/pg_upgrade \
    --old-bindir=$PGBINOLD \
    --new-bindir=$PGBINNEW \
    --old-datadir=$PGDATAOLD \
    --new-datadir=$PGDATANEW \
    --jobs=6 \
    --check \
    --username=postgres \
    --verbose
  ```
