# Kong Demo Tutorial

*Note:*

* The following commands assume that Kong is running locally and that the Admin API is available through port 8001
* Furthermore it is assumed that decK, HTTPie and Docker Compose is installed locally

## Start the Kong demo environment

```bash
docker-compose -f docker-compose-kong-ee-free-mode.yml up -d
```

## Tutorial: Access Kong Manager

When starting Docker Compose, Kong is started in Enterprise free mode. In addition to Kong Open Source, it also allows to use Kong Manager (no further Kong EE features are available). Once started, Kong Manager is reachable using the following URL:

* http://localhost:8002/manager

With that your able to explore and manage Kong configuration using Kong Manager.

## Tutorial: Configure API artifacts

### Create the API artifacts

* Create Kong Service

```bash
http POST :8001/services name=httpbin-svc url=https://httpbin.org:443/anything
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

* Add Consumers for API

```bash
http :8001/consumers username=bob
```

```bash
http :8001/consumers username=alice
```

* Generate Key for newly added consumers

```bash
http POST :8001/consumers/bob/key-auth
```

```bash
http POST :8001/consumers/alice/key-auth
```

* Call to secured service using generated API key

```bash
http :8000/httpbin-api/v1/blablubb apikey:<generated-api-key>
```

* Patch key-auth plugin to hide API key from upstream service

```bash
http PATCH :8001/services/httpbin-svc/plugins/<plugin-id> 'config[hide_credentials]:=true' 
```

* Add rate limiting plugin

```bash
http :8001/services/httpbin-svc/plugins name=rate-limiting config:='{"hour":5}'
```

* Test to call API with Bobs API key for 6 times and see the last request failing with HTTP response code 429. Afterwards try with Alices API key
* Rate limit plugin works on cosnumers by default to aggregate the number of calls; this behaviour can be changed in the Plugin configuration (config.limit_by)

## Tutorial: Declarative Kong management with decK

* Dump your current Kong configuration

```bash
deck dump
```

Change something in your API definition

* Diff the local configuration against the current Kong configuration

```bash
deck diff
```

Review the changes you made to Kong's configuration.

* Sync the local configuration to Kong

```bash
deck sync
```

After syncing the API changes your local dump and your Gateway configuration are the same. This can be verified by again executing the _sync_ command.