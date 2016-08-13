# Themis Finals Checker Protocol (TFCP)
Themis Finals is a contest checking system for CTF contests (attack-defence). This document describes proposals for the new protocol of interacting with service checkers.

## Prerequisites for change
The current protocol is relying on [Beanstalk](http://kr.github.io/beanstalkd/) queue. Despite being quite useful, it has limited range of client libraries which support all features. That is why the new protocol should rely on more well-known and well-supported basis.

## Proposal for change
A service checker is a REST web service. To facilitate the process of developing a service, service developers should be supplied with libraries that help transform their *push* and *pull* methods into a REST service.

A service checker is running on its own FQDN[:port]. For example, the FQDN might look like `service.checker.finals.themis-project.com`.

A service checker accepts requests from master server and puts them in a queue (HTTP request should be finished with 202 Accepted code without waiting for the queue to process the request). When the queue processes the request, it makes another request to master server in order to acknowledge it about the result of processing.

### Authentication
Master server makes request to checker providing authentication token. In response to master server requests, checker makes request to master server providing another authentication token. Both token are generated using the following scheme:

1. Generate so-called **nonce** - random byte string. The size of byte array is specified using `THEMIS_FINALS_NONCE_SIZE` environment variable (by default 16);  
2. Decode Base64 urlsafe encoded (RFC 4648) **secret key** to a byte array. In first case, it should be contained in `THEMIS_FINALS_MASTER_KEY` environment variable, `THEMIS_FINALS_CHECKER_KEY` in the second case;  
3. Combine both **nonce** and **secret key** byte arrays and calculate SHA256 **hash digest** (byte array);  
4. Combine **nonce** and **hash digest** byte arrays and encode it with Base64 urlsafe (RFC4648) method;  
5. Pass generated token in HTTP Header specified in `THEMIS_FINALS_AUTH_TOKEN_HEADER` (by default `X-Themis-Finals-Auth-Token`).

### PUSH
```
POST /push
Host: service.checker.finals.themis-project.com
Content-Type: application/json
X-Themis-Finals-Auth-Token: 7BD3gvcN6fMc5S7xVPFxUP-2lUldx7C_RHOId_cvLzb7v2JbJS60Fgix4eRlqWMb

{
  "params": {
    "endpoint": "10.0.1.3",
    "flag": "c86bc292d3da0e01c15946f17238f2b3=",
    "adjunct": "SEVMTCBZRUFICg=="
  },
  "metadata": {
    "team_name": "Team #1",
    "service_name": "Service #1",
    "round": 34,
    "timestamp": 1467034357.0
  },
  "report_url": "http://master.finals.themis-project.com/api/checker/v1/report_push"
}
```

### PULL
```
POST /pull
Host: service.checker.finals.themis-project.com
Content-Type: application/json
X-Themis-Finals-Auth-Token: D6JYT29r8mTbNOGkTSGwfDLL0MOIMelxSGZ8q2rFon3MvcjMOHf3tof7gc3kIfK5

{
  "params": {
    "request_id": 784,
    "endpoint": "10.0.1.3",
    "flag": "c86bc292d3da0e01c15946f17238f2b3=",
    "adjunct": "SEVMTCBZRUFICg=="
  },
  "metadata": {
    "team_name": "Team #1",
    "service_name": "Service #1",
    "round": 34,
    "timestamp": 1467039822.0
  },
  "report_url": "http://master.finals.themis-project.com/api/checker/v1/report_pull"
}
```

### Web server process(es)
Web server can be launched in several processes. Process monitoring system (e.g. Supervisor) should provide the web server start script with the following environment variables:
- `HOST`
- `PORT`
- `INSTANCE`
- `THEMIS_FINALS_NONCE_SIZE`
- `THEMIS_FINALS_AUTH_TOKEN_HEADER`
- `THEMIS_FINALS_MASTER_KEY`

`INSTANCE` number will be used to calculate the port number to listen to. For example, the first process should be launched with these environment variables: `HOST=127.0.0.1 PORT=3000 INSTANCE=0`, and the second one with these: `HOST=127.0.0.1 PORT=3000 INSTANCE=1`. According to these settings, the processes will listen to ports 3000 and 3001 respectively.

### Queue process(es)
Queue process can be launched in several processes. Process monitoring system (e.g. Supervisor) should provide the queue start script with the following environment variables:
- `INSTANCE`
- `THEMIS_FINALS_NONCE_SIZE`
- `THEMIS_FINALS_AUTH_TOKEN_HEADER`
- `THEMIS_FINALS_CHECKER_KEY`

### Other requirements
To have similar interface, it is suggested to put all scripts in `script` folder. The following script names are reserved and should be used only for certain purposes:

1. `bootstrap` - project setup;  
2. `console` - CLI;  
3. `queue` - queue launch script;  
4. `server` - web server launch script.

Among those reserved, only `queue` and `server` are required. The presence/absence of other scripts depends on the author of the checker.

## License
MIT @ [Alexander Pyatkin](https://github.com/aspyatkin)
