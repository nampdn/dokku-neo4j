# dokku neo4j [![Build Status](https://img.shields.io/travis/nampdn/dokku-neo4j.svg?branch=master "Build Status")](https://travis-ci.org/nampdn/dokku-neo4j) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Unofficial neo4j plugin for dokku. Currently defaults to installing [neo4j 4.0.0](https://hub.docker.com/_/neo4j/).

## requirements

- dokku 0.12.x+
- docker 1.8.x

## installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/nampdn/dokku-neo4j.git --name neo4j
```

## commands

```
neo4j:app-links <app>          List all neo4j service links for a given app
neo4j:backup <name> <bucket> (--use-iam) Create a backup of the neo4j service to an existing s3 bucket
neo4j:backup-auth <name> <aws_access_key_id> <aws_secret_access_key> (<aws_default_region>) (<aws_signature_version>) (<endpoint_url>) Sets up authentication for backups on the neo4j service
neo4j:backup-deauth <name>     Removes backup authentication for the neo4j service
neo4j:backup-schedule <name> <schedule> <bucket> Schedules a backup of the neo4j service
neo4j:backup-schedule-cat <name> Cat the contents of the configured backup cronfile for the service
neo4j:backup-set-encryption <name> <passphrase> Set a GPG passphrase for backups
neo4j:backup-unschedule <name> Unschedules the backup of the neo4j service
neo4j:backup-unset-encryption <name> Removes backup encryption for future backups of the neo4j service
neo4j:clone <name> <new-name>  Create container <new-name> then copy data from <name> into <new-name>
neo4j:connect <name>           Connect via telnet to a neo4j service
neo4j:connect-admin <name>     Connect via telnet to a neo4j service as admin user
neo4j:create <name>            Create a neo4j service with environment variables
neo4j:destroy <name>           Delete the service, delete the data and stop its container if there are no links left
neo4j:enter <name> [command]   Enter or run a command in a running neo4j service container
neo4j:exists <service>         Check if the neo4j service exists
neo4j:export <name> > <file>   Export a dump of the neo4j service database (as gzipped archive file)
neo4j:expose <name> [port]     Expose a neo4j service on custom port if provided (random port otherwise)
neo4j:import <name> < <file>   Import a dump into the neo4j service database
neo4j:info <name>              Print the connection information
neo4j:link <name> <app>        Link the neo4j service to the app
neo4j:linked <name> <app>      Check if the neo4j service is linked to an app
neo4j:list                     List all neo4j services
neo4j:logs <name> [-t]         Print the most recent log(s) for this service
neo4j:promote <name> <app>     Promote service <name> as NEO4J_URL in <app>
neo4j:restart <name>           Graceful shutdown and restart of the neo4j service container
neo4j:start <name>             Start a previously stopped neo4j service
neo4j:stop <name>              Stop a running neo4j service
neo4j:unexpose <name>          Unexpose a previously exposed neo4j service
neo4j:unlink <name> <app>      Unlink the neo4j service from the app
neo4j:upgrade <name>           Upgrade service <service> to the specified version
```

## usage

```shell
# create a neo4j service named lolipop
dokku neo4j:create lolipop

# you can also specify the image and image
# version to use for the service
# it *must* be compatible with the
# official neo4j image
export NEO4J_IMAGE="neo4j"
export NEO4J_IMAGE_VERSION="4.0.0"
dokku neo4j:create lolipop

# get connection information as follows
dokku neo4j:info lolipop

# you can also retrieve a specific piece of service info via flags
dokku neo4j:info lolipop --config-dir
dokku neo4j:info lolipop --data-dir
dokku neo4j:info lolipop --dsn
dokku neo4j:info lolipop --exposed-ports
dokku neo4j:info lolipop --id
dokku neo4j:info lolipop --internal-ip
dokku neo4j:info lolipop --links
dokku neo4j:info lolipop --service-root
dokku neo4j:info lolipop --status
dokku neo4j:info lolipop --version

# a bash prompt can be opened against a running service
# filesystem changes will not be saved to disk
dokku neo4j:enter lolipop

# you may also run a command directly against the service
# filesystem changes will not be saved to disk
dokku neo4j:enter lolipop ls -lah /

# a neo4j service can be linked to a
# container this will use native docker
# links via the docker-options plugin
# here we link it to our 'playground' app
# NOTE: this will restart your app
dokku neo4j:link lolipop playground

# the following environment variables will be set automatically by docker (not
# on the app itself, so they won’t be listed when calling dokku config)
#
#   DOKKU_NEO4J_LOLIPOP_NAME=/lolipop/DATABASE
#   DOKKU_NEO4J_LOLIPOP_PORT=tcp://172.17.0.1:27017
#   DOKKU_NEO4J_LOLIPOP_PORT_27017_TCP=tcp://172.17.0.1:27017
#   DOKKU_NEO4J_LOLIPOP_PORT_27017_TCP_PROTO=tcp
#   DOKKU_NEO4J_LOLIPOP_PORT_27017_TCP_PORT=27017
#   DOKKU_NEO4J_LOLIPOP_PORT_27017_TCP_ADDR=172.17.0.1
#
# and the following will be set on the linked application by default
#
#   NEO4J_URL=neo4jdb://lolipop:SOME_PASSWORD@dokku-neo4j-lolipop:27017/lolipop
#
# NOTE: the host exposed here only works internally in docker containers. If
# you want your container to be reachable from outside, you should use `expose`.

# another service can be linked to your app
dokku neo4j:link other_service playground

# since DATABASE_URL is already in use, another environment variable will be
# generated automatically
#
#   DOKKU_NEO4J_BLUE_URL=neo4jdb://other_service:ANOTHER_PASSWORD@dokku-neo4j-other-service:27017/other_service

# you can then promote the new service to be the primary one
# NOTE: this will restart your app
dokku neo4j:promote other_service playground

# this will replace NEO4J_URL with the url from other_service and generate
# another environment variable to hold the previous value if necessary.
# you could end up with the following for example:
#
#   NEO4J_URL=neo4jdb://other_service:ANOTHER_PASSWORD@dokku-neo4j-other-service:27017/other_service
#   DOKKU_NEO4J_BLUE_URL=neo4jdb://other_service:ANOTHER_PASSWORD@dokku-neo4j-other-service:27017/other_service
#   DOKKU_NEO4J_SILVER_URL=neo4jdb://lolipop:SOME_PASSWORD@dokku-neo4j-lolipop:27017/lolipop

# you can also unlink a neo4j service
# NOTE: this will restart your app and unset related environment variables
dokku neo4j:unlink lolipop playground

# you can tail logs for a particular service
dokku neo4j:logs lolipop
dokku neo4j:logs lolipop -t # to tail

# you can dump the database
dokku neo4j:export lolipop > lolipop.dump

# you can import a dump
dokku neo4j:import lolipop < database.dump

# you can clone an existing database to a new one
dokku neo4j:clone lolipop new_database

# finally, you can destroy the container
dokku neo4j:destroy lolipop
```

## Backups

Datastore backups are supported via AWS S3 and S3 compatible services like [minio](https://github.com/minio/minio).

You may skip the `backup-auth` step if your dokku install is running within EC2
and has access to the bucket via an IAM profile. In that case, use the `--use-iam`
option with the `backup` command.

Backups can be performed using the backup commands:

```
# setup s3 backup authentication
dokku neo4j:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY

# remove s3 authentication
dokku neo4j:backup-deauth lolipop

# backup the `lolipop` service to the `BUCKET_NAME` bucket on AWS
dokku neo4j:backup lolipop BUCKET_NAME

# schedule a backup
# CRON_SCHEDULE is a crontab expression, eg. "0 3 * * *" for each day at 3am
dokku neo4j:backup-schedule lolipop CRON_SCHEDULE BUCKET_NAME

# cat the contents of the configured backup cronfile for the service
dokku neo4j:backup-schedule-cat lolipop

# remove the scheduled backup from cron
dokku neo4j:backup-unschedule lolipop
```

Backup auth can also be set up for different regions, signature versions and endpoints (e.g. for minio):

```
# setup s3 backup authentication with different region
dokku neo4j:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION

# setup s3 backup authentication with different signature version and endpoint
dokku neo4j:backup-auth lolipop AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_REGION AWS_SIGNATURE_VERSION ENDPOINT_URL

# more specific example for minio auth
dokku neo4j:backup-auth lolipop MINIO_ACCESS_KEY_ID MINIO_SECRET_ACCESS_KEY us-east-1 s3v4 https://YOURMINIOSERVICE
```

## Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `NEO4J_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.
