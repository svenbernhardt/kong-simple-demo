# Kubernetes Resources

## Basic

Basic Infrastructure-Setup without any Kong-Custom-Resource-Definitions (CRDs).

1. Apply all resources on your running kubernetes instance, deployed with kong kubernetes ingress-controller

```bash
kubectl apply -f k8s-resources/basics/
```

2. Make request to new ingress

```bash
$ curl https://httpbin.example.com:8081/json

{
  "slideshow": {
    "author": "Yours Truly", 
    "date": "date of publication", 
    "slides": [
      {
        "title": "Wake up to WonderWidgets!", 
        "type": "all"
      }, 
      {
        "items": [
          "Why <em>WonderWidgets</em> are great", 
          "Who <em>buys</em> WonderWidgets"
        ], 
        "title": "Overview", 
        "type": "all"
      }
    ], 
    "title": "Sample Slide Show"
  }
}
```

## Consumer

1. Apply all resources on your running kubernetes instance, deployed with kong kubernetes ingress-controller

```bash
kubectl apply -f k8s-resources/consumer/
```

2. Make request without any api-key

```bash
$ curl https://httpbin.example.com:8081/get

401 - Unauthorized
{
  "message":"No API key found in request"
}
```

3. Make request with bob or alice

```bash
$ curl https://httpbin.example.com:8081/get -H apikey:bob-secret

{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Apikey": "bob-secret", 
    "Connection": "keep-alive", 
    "Host": "httpbin.example.com:8081", 
    "User-Agent": "curl/7.84.0", 
    "X-Consumer-Id": "a1d0181c-f84c-4933-a0cd-3dafa32bbfff", 
    "X-Consumer-Username": "bob", 
    "X-Credential-Identifier": "a4625e92-34d6-4b57-804e-88ff161a9aee", 
    "X-Forwarded-Host": "httpbin.example.com", 
    "X-Forwarded-Path": "/get"
  }, 
  "origin": "192.168.125.64", 
  "url": "https://httpbin.example.com/get"
}
```

4. Change Key-Auth-Plugin to hide credentials

File `k8s-resourcen/consumer/01-keyauth-plugin.yaml`
```bash
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: "key-auth"
  namespace: demo
config:                       # <---- Uncomment
  hide_credentials: true      # <---- Uncomment
plugin: key-auth
```

Then apply resource.

5. Request again with api-key

```bash
$ curl https://httpbin.example.com:8081/get -H apikey:bob-secret
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Connection": "keep-alive", 
    "Host": "httpbin.example.com:8081", 
    "User-Agent": "curl/7.84.0", 
    "X-Consumer-Id": "a1d0181c-f84c-4933-a0cd-3dafa32bbfff", 
    "X-Consumer-Username": "bob", 
    "X-Credential-Identifier": "a4625e92-34d6-4b57-804e-88ff161a9aee", 
    "X-Forwarded-Host": "httpbin.example.com", 
    "X-Forwarded-Path": "/get"
  }, 
  "origin": "192.168.125.64", 
  "url": "https://httpbin.example.com/get"
}
```

## Rate-Limiting

1. Apply all resources on your running kubernetes instance, deployed with kong kubernetes ingress-controller

```bash
kubectl apply -f k8s-resources/consumer/
```

2. Request with api-key

```bash
$ curl https://httpbin.example.com:8081/get -H apikey:bob-secret -v
< HTTP/2 200 
...
< x-ratelimit-limit-hour: 5
< ratelimit-reset: 1140
< ratelimit-remaining: 1
< ratelimit-limit: 5
< x-ratelimit-remaining-hour: 1
...

{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Connection": "keep-alive", 
    "Host": "httpbin.example.com:8081", 
    "User-Agent": "curl/7.84.0", 
    "X-Consumer-Id": "a1d0181c-f84c-4933-a0cd-3dafa32bbfff", 
    "X-Consumer-Username": "bob", 
    "X-Credential-Identifier": "a4625e92-34d6-4b57-804e-88ff161a9aee", 
    "X-Forwarded-Host": "httpbin.example.com", 
    "X-Forwarded-Path": "/get"
  }, 
  "origin": "192.168.125.64", 
  "url": "https://httpbin.example.com/get"
}
```

Request until ratelimit exceeds. Then do the same with alice api-key.

3. Change Rate-Limiting-Plugin to limit based on `ip`

File `k8s-resourcen/rate-limiting/01-ratelimiting-plugin.yaml`
```bash
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: "rate-limiting"
  namespace: demo
config:
  hour: 5
  limit_by: ip      # <---- Uncomment
plugin: key-auth
```

Then apply resource.

4. Check if it makes difference.