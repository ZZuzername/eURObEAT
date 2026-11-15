# Exercise P2: HTTP with curl -i (httpbin.org)
Run: Nov 15, 2026 09:44 AM ET

## 1. GET
```
curl -s -i https://httpbin.org/get
```
```
HTTP/2 200 
content-type: application/json
```

## 2. POST with JSON body
```
curl -s -i -X POST -H 'Content-Type: application/json' -d '{"name":"Avery"}' https://httpbin.org/post
```
```
HTTP/2 200 
content-length: 385
```

## 3. 404
```
curl -s -i https://httpbin.org/status/404
```
```
HTTP/2 404 
content-type: text/html; charset=utf-8
```

## 4. 401
```
curl -s -i https://httpbin.org/basic-auth/user/pass
```
```
HTTP/2 401 
www-authenticate: Basic realm="Fake Realm"
```

## Retry
I would not retry any of these as-is: the GET and POST succeeded, the 404 means the resource does not exist, and the 401 means credentials are missing, so repeating them changes nothing; I would retry only temporary failures like a 429, a 503, or a timeout, with a growing wait between tries.
