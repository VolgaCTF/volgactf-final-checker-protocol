# Themis Finals Checker Protocol (TFCP)
Themis Finals is a contest checking system for CTF contests (attack-defence). This document describes proposals for the new protocol of interacting with service checkers.

## Prerequisites for change
The current protocol is relying on [Beanstalk](http://kr.github.io/beanstalkd/) queue. Despite being quite useful, it has limited range of client libraries which support all features. That is why the new protocol should rely on more well-known and well-supported basis.

## Proposal for change
A service checker is a REST web service. To facilitate the process of developing a service, service developers should be supplied with libraries that help transform their *push* and *pull* methods into a REST service.

A service checker is running on its own FQDN[:port]. For example, the FQDN might look like `service.checker.finals.themis-project.com`.

A service checker accepts requests from master server and puts them in a queue (HTTP request should be finished with 202 Accepted code without waiting for the queue to process the request). When the queue processes the request, it makes another request to master server in order to acknowledge it about the result of processing.

### PUSH
```
POST /push
Host: service.checker.finals.themis-project.com
Content-Type: application/json

{
	"request": {
		"id": 783,
		"host": "10.0.1.3",
		"port": 8080,
		"flag": "c86bc292d3da0e01c15946f17238f2b3=",
		"adjunct": "SEVMTCBZRUFICg=="
	},
	"metadata": {
		"team_name": "Team #1",
		"service_name": "Service #1",
		"round_number": 34,
		"timestamp": 1467034357.0
	},
	"response": {
		"url": "http://master.finals.themis-project.com/api/checker/respond-push"
	}
}
```

### PULL
```
POST /pull
Host: service.checker.finals.themis-project.com
Content-Type: application/json

{
	"request": {
		"id": 784,
		"host": "10.0.1.3",
		"port": 8080,
		"flag": "c86bc292d3da0e01c15946f17238f2b3=",
		"adjunct": "SEVMTCBZRUFICg=="
	},
	"metadata": {
		"team_name": "Team #1",
		"service_name": "Service #1",
		"round_number": 34,
		"timestamp": 1467039822.0
	},
	"response": {
		"url": "http://master.finals.themis-project.com/api/checker/respond-pull"
	}
}
```