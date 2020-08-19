# Kong Demo

*Note:*

* The following commands assume that Kong is running locally and that the Admin API is available through port 8001
* Furthermore it is assumed that decK, HTTPie and Docker Compose is installed locally

## Start the Kong demo environment

```bash
docker-compose -f docker-compose-kong-oss.yml up -d
```

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
