---
title: "MongoDB Server Administration"
layout: post
date: 2018-02-20 20:21
image: /assets/images/markdown.jpg
headerImage: false
tag:
- markdown
- components
- extra
category: blog
author: nikhilraju
description: General information related to administeration of a Mongo Server
---

# MongoDB Server Administration
The intention of this document is to share a brief overview of relevant concepts and internal knowledge necessary to administer and debug server side issues

Similar to the process for application code changes, server configurations and changes should also be reviewed by one or more core contributor. 

## mongod process 

mongod is the primary daemon process for the MongoDB system. It handles data requests, manages data access, and performs background management operations.

Basic [upstart](http://upstart.ubuntu.com/) operations:
* check status:  `sudo service mongod status`
* start:         `sudo service mongod start`
* stop:			 `sudo service mongod stop`
* restart:       `sudo service mongod restart`

## Configuration
Location of ubuntu upstart config file:  `/etc/mongod.conf`

data : `/data/db`
mongod logs: `/log/mongod.log`  

## Useful mongo shell commands to manage Replica Set
Connect to the primary of the replica set and on the mongo shell type
* Check if the current instance is master:	`rs.isMaster()`
* Check current status: 		 			`rs.status()`
* Examine configuration: 		 			`rs.conf()` 
* Add a replica set member: 	 			`rs.add("host:port")`
* Remove a replica set member: 	 			`rs.remove("host:port")`
* Get the build version (for SOC2 questions) 		`db.runCommand( { buildInfo: 1 } )`	

[More commands](https://docs.mongodb.com/manual/reference/method/js-replication/)   

## Replication
Replication helps provide redundancy, improve data availability
Some important design decisions and tradeoffs: 
* Chain replication - enable or disable?
* Optional Arbiter, number of data bearing nodes
* [Consistency](https://docs.mongodb.com/manual/administration/production-notes/#data-consistency) requirements:
..*[WriteConcern](https://docs.mongodb.com/manual/reference/write-concern/) : `j`, `w`
..*[ReadConcern](https:/
/docs.mongodb.com/manual/reference/read-concern/) : `local` ,`majority`, `linearizable`
**NOTE** To use `majority`, `replication.enableMajorityReadConcern` should be true.

For more information on replication please see the official [documentation](https://docs.mongodb.com/manual/replication/)

## Managing Roles 
[Official Link](https://docs.mongodb.com/v3.2/tutorial/manage-users-and-roles/#overview)

## Authentication
Follow the documentation [here](https://docs.mongodb.com/manual/tutorial/enable-authentication/) to enable authentication in a replica set.

## Converting a Standalone to a Replica Set
Follow instructions [here](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/)

## Cloud Manager
[Cloud Manager](https://cloud.mongodb.com) is a service to manage, monitor, backup and configure alerts for a MongoDB deployment. 

To use this service, ask a Project Owner to add you as a user.

### Alerts

See official documentation [here](https://docs.cloudmanager.mongodb.com/tutorial/manage-alerts-and-events/)

### Backups

See official documentation [here](https://docs.cloudmanager.mongodb.com/tutorial/nav/backup-use/)

## Backup Agents
See official documentation [here](https://docs.cloudmanager.mongodb.com/tutorial/nav/backup-agent/)

## Monitoring Agents
See official documentation [here](https://docs.cloudmanager.mongodb.com/tutorial/nav/monitoring-agent/)

## MongoDB 3.4 features

* Starting with MongoDB 3.4, it is possible to enable authentication without downtime. See [this](https://docs.mongodb.com/manual/reference/configuration-options/#security.transitionToAuth) for more details.

* There are some new features like Views which could be useful to standardize SQL like joins across collections. But enabling these should be done with prior approval of #tech since some of these are [backwards incompaitible](https://docs.mongodb.com/manual/release-notes/3.4-compatibility/#backwards-incompatible-features)

## Provisioning a box
Ideally, this would be an automated step that very few people should care about.

In case you end up doing this step manually, some things to remember are:
* Security groups
* Storage: Make sure you allocate enough disk space and additional EBS volumes as needed.
* Make sure the volumes are [mounted](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html) otherwise the additional volumes won't be used.
* Storage: WiredTiger ( Mongo's storage engine ) highly recommends using [`XFS`](https://wiki.ubuntu.com/XFS) 
Read [this](https://scalegrid.io/blog/xfs-vs-ext4-comparing-mongodb-performance-on-aws-ec2/) to get a sense of the performane difference with ext4
