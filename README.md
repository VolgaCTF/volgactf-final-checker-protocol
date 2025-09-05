# VolgaCTF Final Checker Protocol

[VolgaCTF Final](https://github.com/VolgaCTF/volgactf-final) is an automatic checking system (ACS) for A/D CTF contests.

This document describes interaction between an ACS master and service checkers.

## Protocol
A service checker is a REST web service running on its own FQDN[:port]. For example, the FQDN might look like `awesome-service-checker.final.volgactf.ru`.

A service checker accepts requests from the ACS master and puts them into a queue. An HTTP request must be finished with a 202 Accepted code (without waiting for the queue to process the request). When the queue processes the request, it makes another HTTP request to the ACS master in order to acknowledge it about the result of processing.

Given that the ACS master server signs capsules with a private key, a public key must be made available to a service checker so that it can decode capsules.

### Authentication
See [HTTP Basic authentication](https://en.wikipedia.org/wiki/Basic_access_authentication).

### PUSH

**Master → Checker**

```
POST /push
Host: awesome-service-checker.final.volgactf.ru
Content-Type: application/json
Authorization: Basic dmFnc.....

{
  "metadata": {
    "round": 1,
    "service_name": "Awesome Service",
    "team_name": "Dream Team",
    "timestamp": "2019-05-05T12:24:13.063216+00:00"
  },
  "params": {
    "capsule": "VolgaCTF{eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJmbGFnIjoiNDkzMDIyZDFlODk2ZTczZGViZjMyYjdkNjc2NmU4NzQifQ.UZm19fi876xKDcYrjiK_rdvrZWybe5Qn9Wx-QpuR-8RKExr7ksTyAIVjZBkSGzo1RC_d6jQmSQKP83aX2Kb9Pw}",
    "endpoint": "10.0.1.3",
    "label": "jbhhNzsU"
  },
  "report_url": "http://master.final.volgactf.ru/api/checker/v2/report_push"
}
```

**Checker → Master**

```
POST /api/checker/v2/report_push
Host: master.final.volgactf.ru
Content-Type: application/json
Authorization: Basic dmFnc.....

{
  "flag": "493022d1e896e73debf32b7d6766e874",
  "label": "b12WSEBn",
  "message": "...",
  "status": 101,
  "open_data": "wyDXmZEL"
}
```

### PULL

**Master → Checker**

```
POST /pull
Host: awesome-service-checker.final.volgactf.ru
Content-Type: application/json
Authorization: Basic dmFnc.....

{
  "metadata": {
    "round": 1,
    "service_name": "Awesome Service",
    "team_name": "Dream Team",
    "timestamp": "2019-05-05T12:32:57.045429+00:00"
  },
  "params": {
    "capsule": "VolgaCTF{eyJ0eXAiOiJKV1QiLCJhbGciOiJFUzI1NiJ9.eyJmbGFnIjoiNDkzMDIyZDFlODk2ZTczZGViZjMyYjdkNjc2NmU4NzQifQ.UZm19fi876xKDcYrjiK_rdvrZWybe5Qn9Wx-QpuR-8RKExr7ksTyAIVjZBkSGzo1RC_d6jQmSQKP83aX2Kb9Pw}",
    "endpoint": "10.0.1.3",
    "label": "b12WSEBn",
    "request_id": 79
  },
  "report_url": "http://master.final.volgactf.ru/api/checker/v2/report_pull"
}
```

**Checker → Master**

```
POST /api/checker/v2/report_pull
Host: master.final.volgactf.ru
Content-Type: application/json
Authorization: Basic dmFnc.....

{
  "message": "...",
  "request_id": 79,
  "status": 101
}
```

### Status codes

| Code | Description |
| ---- | ----------- |
| `101` | `UP` |
| `102` | `CORRUPT` |
| `103` | `MUMBLE` |
| `104` | `DOWN` |


## License
MIT @ [VolgaCTF](https://github.com/VolgaCTF)
