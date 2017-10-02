---
layout: post
title:  "Gateway vs load-balancer vs reverse-proxy"
date:   2017-09-28 23:55:00
Tags: [gateway, load-balancer, reverse-proxy]
Categories: [api]

---
<style>
    .table .tr {
        border: 1px solid gray;
    }
</style>

What is the difference between an api-gateway, a load balancer and a reverse proxy ? Here is my attempt to provide a synthetic synthetic answer. Note that the features below are not mutualy exlusive...


|               | could provide | talks to | improves | OSI layer |
|---------------|---------------|----------| ---------| ----------|
| Reverse-proxy | caching <br> compression <br> ssl | 1 server | security (1) <br>     flexibility (2) <br> performance | 4 or 7
| Load-balancer | load balancing <br> session-persistence <br> health check <br> circuit-breaker| many servers | scalability (3) <br> availability (4) | 4 or 7
| API-gateway | API composition  <br> authentication| many servers | payload (5) <br> roundtrips | 7



 * (1) by exposing only proxy to the internet
 * (2) by enabling re-configuration of infra
 * (3) by supporting more incoming requests
 * (4) by providing redundancy
 * (5) both in terms of network bandwith and DTO structure for app consumption

## Non-exhaustive list of products
- [Express-gateway (based on node + express)](http://www.express-gateway.io/)
- [HA proxy](http://www.haproxy.org/)
- [Kong (based on nginx)](https://getkong.org/)
- [Nginx](https://www.nginx.com/)

## [OSI Layers](https://en.wikipedia.org/wiki/OSI_model)

7. Application
6. Presentation
5. Session
4. Transport
3. Network
2. Data link
1. Physical
