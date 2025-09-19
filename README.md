# Description

- Dumps a remote Fly.io postgres database using the correct postgres Docker version.

- Retains the last 30 dump files.

- Intended to be run daily via a cron job or other scheduling tool.

[![Maintainability](https://qlty.sh/gh/scottohara/projects/fly-postgres-backup/maintainability.svg)](https://qlty.sh/gh/scottohara/projects/fly-postgres-backup)

# Background

Fly.io offers a way to run a Postgres cluster, but they go to great lengths to make it very clear that [this is not managed postgres](https://fly.io/docs/postgres/getting-started/what-you-should-know/).

(At the time of writing, [managed postgres](https://fly.io/docs/mpg/overview/) is in technical preview. This repository is only of use for the legacy, unmanaged postgres offering.)

A critical thing to note about the legacy unmanaged postgres is that Fly will take daily snapshots of the storage volumes attached to the postgres cluster and keep them for 5 days, but this is not the same as database dump/backup. Fly recommends users make their own backup plans for longer retention and off-site storage.

Dumping a postgres database is simple enough with the `pg_dump` utility, but one slight complicating factor is that if you are initiating the dump from a remote machine, the version of the `pg_dump` used must match the version of the Postgres cluster hosting the database being dumped.

To enable use across different clusters with different versions of Postgres, the script spins up the correct Postgres Docker image.

# Usage

The script expects a number of environment variables to be set (and will abort if any are missing):

```sh
FILE_PREFIX=any-prefix-you-like
FLY_ACCESS_TOKEN=access-token-goes-here
FLY_APP=name-of-the-fly-pg-app
PGVERSION=17
PGDATABASE=database-name
PGUSER=username
PGPASSWORD=password
```

Set the `PGVERSION` to match the version of your Fly postgres cluster.

The script first runs `fly proxy` to create a tunnel on local port `15432` to your postgres server running on remote port `5432`, and once the proxy is ready the script then runs `pg_dump`.

When the script completes successfully, a file named `[FILE_PREFIX]-YYYY-MM-DD-hh-mm-ss.dump` will be created in the same directory as the script.

A cleanup process removes old dump files, keeping at most the 30 most recent dumps.
