# Pact Broker with Proxy

This repo is for reproducing issues with http proxies and proxy auth. It contains a docker-compose.yml file with a squid proxy container and a Pact Broker container.
It uses the OSS Pact Broker instead of PactFlow so that it can be run without requiring a licence file.

## Instructions for use

```
docker compose up
```


### Publish with pact-cli Docker container using Ruby 3

```
export PACT_BROKER_BASE_URL=http://pact-broker:9292
export http_proxy=http://test:test@127.0.0.1:8080

docker run --rm \
--network="host" \
 -w ${PWD} \
 -v ${PWD}:${PWD} \
 -e PACT_BROKER_BASE_URL \
 -e http_proxy \
  pactfoundation/pact-cli:latest \
  publish \
  ${PWD}/example-pact.json \
  --consumer-app-version fake-git-sha-for-demo-$(date +%s) \
  --verbose
```

This works.

```
opening connection to 127.0.0.1:8080...
opened
<- "GET http://pact-broker:9292/? HTTP/1.1\r\nAccept-Encoding: gzip;q=1.0,deflate;q=0.6,identity;q=0.3\r\nAccept: application/hal+json\r\nUser-Agent: Ruby\r\nProxy-Authorization: [redacted]\r\n"
-> "HTTP/1.1 200 OK\r\n"
-> "Vary: Accept\r\n"
-> "Content-Type: application/hal+json;charset=utf-8\r\n"
-> "Date: Wed, 26 Apr 2023 01:29:38 GMT\r\n"
-> "Server: Webmachine-Ruby/1.6.0 Rack/1.3\r\n"
-> "X-Pact-Broker-Version: 2.106.0\r\n"
-> "X-Content-Type-Options: nosniff\r\n"
-> "Content-Length: 4356\r\n"
-> "X-Cache: MISS from 44f92f9a9dc8\r\n"
-> "X-Cache-Lookup: MISS from 44f92f9a9dc8:3128\r\n"
-> "Via: 1.1 44f92f9a9dc8 (squid/5.7)\r\n"
-> "Connection: keep-alive\r\n"
-> "\r\n"
```


### Publish with pact-ruby-standalone container using Ruby 2.2

```
curl -fsSL https://raw.githubusercontent.com/pact-foundation/pact-ruby-standalone/master/install.sh | bash

export PACT_BROKER_BASE_URL=http://pact-broker:9292
export http_proxy=http://test:test@127.0.0.1:8080

pact/bin/pact-broker publish example-pact.json --consumer-app-version fake-git-sha-for-demo-$(date +%s) --verbose

```

This does not work.

```
opening connection to 127.0.0.1:8080...
opened
<- "GET http://pact-broker:9292/? HTTP/1.1\r\nAccept-Encoding: gzip;q=1.0,deflate;q=0.6,identity;q=0.3\r\nAccept: application/hal+json\r\nUser-Agent: Ruby\r\nHost: pact-broker:9292\r\n\r\n"
-> "HTTP/1.1 407 Proxy Authentication Required\r\n"
-> "Server: squid/5.7\r\n"
-> "Mime-Version: 1.0\r\n"
-> "Date: Wed, 26 Apr 2023 01:35:44 GMT\r\n"
-> "Content-Type: text/html;charset=utf-8\r\n"
-> "Content-Length: 3595\r\n"
-> "X-Squid-Error: ERR_CACHE_ACCESS_DENIED 0\r\n"
-> "Vary: Accept-Language\r\n"
-> "Content-Language: en\r\n"
-> "Proxy-Authenticate: Basic realm=\"my-proxy-name\"\r\n"
-> "X-Cache: MISS from 44f92f9a9dc8\r\n"
-> "X-Cache-Lookup: NONE from 44f92f9a9dc8:3128\r\n"
-> "Via: 1.1 44f92f9a9dc8 (squid/5.7)\r\n"
-> "Connection: keep-alive\r\n"
-> "\r\n"
```

