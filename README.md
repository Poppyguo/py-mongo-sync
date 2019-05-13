# py-mongo-sync

An oplog based data sync tool for MongoDB written in Python.
It's used to sync data from a replica set to another standalone/replica-set/sharded-cluster.

## Features

- initial sync and oplog based incremental sync
- sync the specified databases
- sync the specified collections
- concurrent oplog replay


## Requirements

- gevent
- mmh3
- pymongo 3.0 or later

    > Use pymongo 3.5.1 if your source MongoDB is version 2.4.  
    > pymongo 3.6 adds support for MongoDB 3.6, drops support for CPython 3.3 (PyPy3 is still supported), and drops support for MongoDB versions older than 2.6.  
    > Refer to [https://api.mongodb.com/python/3.6.0/changelog.html](https://api.mongodb.com/python/3.6.0/changelog.html).

- toml

## Notice

- source is a replica set of MongoDB with version 2.4/3.0/3.2/3.4
- ignore following databases
    - admin
    - local
- ignore system collections
    - system.\*
- create users for destination manually if necessary
- authenticate with superuser if source authentication is enabled
- sync sharded-cluster by running a sync process for each shard, first of all, guarantee that balancer is off
- not support geospatial index


## Usage 

Command options has functional limitations.
It's strongly recommended that use config file.

### sync

```bash
usage: sync.py [-h] [-f [CONFIG]] [--src [SRC]] [--src-authdb [SRC_AUTHDB]]
               [--src-username [SRC_USERNAME]] [--src-password [SRC_PASSWORD]]
               [--dst [DST]] [--dst-authdb [DST_AUTHDB]]
               [--dst-username [DST_USERNAME]] [--dst-password [DST_PASSWORD]]
               [--start-optime [START_OPTIME]]
               [--optime-logfile [OPTIME_LOGFILE]] [--logfile [LOGFILE]]

Sync data from a replica-set to another MongoDB/Elasticsearch.

optional arguments:
  -h, --help            show this help message and exit
  -f [CONFIG], --config [CONFIG]
                        configuration file, note that command options will
                        override items in config file
  --src [SRC]           source should be hostportstr of a replica-set member
  --src-authdb [SRC_AUTHDB]
                        src authentication database, default is 'admin'
  --src-username [SRC_USERNAME]
                        src username
  --src-password [SRC_PASSWORD]
                        src password
  --dst [DST]           destination should be hostportstr of a mongos or
                        mongod instance
  --dst-authdb [DST_AUTHDB]
                        dst authentication database, default is 'admin', for
                        MongoDB
  --dst-username [DST_USERNAME]
                        dst username, for MongoDB
  --dst-password [DST_PASSWORD]
                        dst password, for MongoDB
  --start-optime [START_OPTIME]
                        timestamp in second, indicates oplog based increment
                        sync
  --optime-logfile [OPTIME_LOGFILE]
                        optime log file path, use this as start optime if
                        without '--start-optime'
  --logfile [LOGFILE]   log file path

```


## Configurations

Use [TOML](https://github.com/toml-lang/toml) as configuration file format.

### src
Source config items.

- src.hosts - hostportstr of a member of replica set
- src.authdb - authentiction database
- src.username - username
- src.password - password

### dst
Destination config items.

- MongoDB refer to [mongo_conf.toml](example/mongo_conf.toml)
    - dst.mongo.hosts
    - dst.mongo.authdb
    - dst.mongo.username
    - dst.mongo.password

- Elasticsearch refer to [es_conf.toml](example/es_conf.toml)
    - dst.type
    - dst.hosts

### sync
Custom sync options.

`sync.dbs` specfies the databases to sync.
`sync.dbs.colls` specifies the collections to sync.

- sync.dbs - databases to sync, sync all databases if not specify
    - sync.dbs.db - source database name
    - sync.dbs.rename_db - destination database name, stay the same if not specify
    - sync.dbs.colls - collectons to sync, sync all collections if not specify

`coll` in `sync.dbs.colls` element specifies the collection to sync.
`fileds` in `sync.dbs.colls` element specifies the fields of current collection to sync.

### log
- log.filepath - log file path, write to stdout if empty or not set


## TODO List

- [ ] command options tuning
- [ ] config file format tuning
