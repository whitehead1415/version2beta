---
pagetitle: A quick note on MongoDB replica sets
longtitle: Once again, I didn't find an answer online, so I'm putting it out here.
tags: devops
author: Rob Martin
published: 2011-11-29 11:49
snippet: MongoDB is replying to a replica-set add command with "Need most members up to reconfigure, not ok". The problem is with my configuration.
---

MongoDB is replying to a replica-set add command with "Need most members up to reconfigure, not ok". The problem is with my configuration.

Here's the error:

	PRIMARY> rs.add("nosql1-private:27017")
	{
		"assertion" : "need most members up to reconfigure, not ok : nosql1-private:27017",
		"assertionCode" : 13144,
		"errmsg" : "db assertion failure",
		"ok" : 0
	}

The problem is in my configuration on the slave Mongod. Note that I'm using a configuration file, rather than command line options, which is how the tutorials are written. This fits in with my init file, which I can share if anyone wants an Ubuntu init for MongoDB 2.0.2.

	fork = true
	bind_ip = nosql1-private
	port = 27017 # default port
	logpath = /var/log/mongodb/mongod.log
	logappend = true
	cpu = true # log CPU activity periodically
	journal = true
	auth = true
	replSet = gedc

I disabled journal (shouldn't be needed on a slave, right?) and auth, which seems to actually be the problem. After, the rs.add() works fine.

	PRIMARY> rs.add("nosql1-private:27017")
	{ "ok" : 1 }

I'm not sure I don't have to go back and get auth reenabled, though, because now my status says that the second server is "still initializing" and "(not reachable/healthy)". But when I try to add authentication credentials, I get this error:

	uncaught exception: error { "$err" : "not master and slaveok=false", "code" : 13435 }

So, I did discover what I really needed was a keyFile directive[^key] in my mongod.conf. The file itself contains a base64 text key the servers will use for communicating.

	...
	keyFile = /etc/mongodb/key.txt
	...

[^key]: See the "Replica Set and Sharding Authentication" section on the [MongoDB doc page for Security and Authentication][key].

[key]: http://www.mongodb.org/display/DOCS/Security+and+Authentication#SecurityandAuthentication-ReplicaSetandShardingAuthentication "MongoDB docs for security and authentication, including key files for shards and replica sets."
