# Kong Demo Tutorial

*Note:*

* The following commands assume that Kong is running locally and that the Admin API is available through port 8001
* Furthermore it is assumed that decK, HTTPie and Docker Compose is installed locally

## Start the Kong demo environment

```bash
docker-compose -f docker-compose-kong-oss.yml up -d
```

## Configure and use Konga Admin UI for Kong OSS Gateway

When starting Docker Compose, Konga Admin UI (a Open Source Admin UI for Kong) is also deployed. Once started, Konga is reachable using the following URL: 

* http://localhost:1337

When accessing the Konga UI for the first time, you're required to create a Admin user that is used to access Konga UI afterwards. After logging in for the first time, you need to configure the connection to the Kong OSS API Gateway. As Kong Admin API URL use: http://kong:8001.

## Configure the API artifacts

### Create the API artifacts

* Create Kong Service

```bash
http POST :8001/services name=httpbin-svc protocol=https host=httpbin.org port:=443 path=/anything
```

* Create Kong Route

```bash
http POST :8001/services/httpbin-svc/routes name=httpbin-route paths:='["/httpbin-api/v1"]'
```

* Example GET call

```bash
http :8000/httpbin-api/v1/blablubb
```

### Add plugins to secure your API services

* Add Key-auth plugin

```bash
http :8001/services/httpbin-svc/plugins name=key-auth
```

* Add Consumer for API

```bash
http :8001/consumers username=httpbin-consumer
```

* Generate Key for newly added consumer

```bash
http POST :8001/consumers/httpbin-consumer/key-auth
```

* Call to secured service using generated API key

```bash
http :8000/httpbin-api/v1/blablubb apikey:<generated-api-key>
```

* Add rate limiting plugin

```bash
http :8001/services/httpbin-svc/plugins name=rate-limiting config:='{"hour":10}'
```

## Declarative Kong management with decK

* Dump your current Kong configuration

```bash
deck dump
```

Change something in your API definition

* Diff the local configuration against the current Kong configuration

```bash
deck diff
```

* Sync the local configuration to Kong

```bash
deck sync
```
