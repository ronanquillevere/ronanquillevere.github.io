---
layout: post
title:  "Coding an expess gateway policy plugin"
date:   2017-09-12 23:55:00
Tags: [express, gateway, api, nodejs, node, micro-services]
Categories: [api]

---

Hello everyone,

I was looking at a efficient way to build an API gateway for my job. So first if you have no idea of what I am talking about here are some good articles about it and micro-services architectures.

- [API gateway pattern](http://microservices.io/patterns/apigateway.html)
- [Building Microservices: Using an API Gateway](https://www.nginx.com/blog/building-microservices-using-an-api-gateway/)
- [Embracing the Differences : Inside the Netflix API Redesign](https://medium.com/netflix-techblog/embracing-the-differences-inside-the-netflix-api-redesign-15fd8b3dc49d)
- [Martin Fowler serverless](https://martinfowler.com/articles/serverless.html)

I know that [nginx](https://www.nginx.com/solutions/api-gateway/) is probably an excellent option but the licensing does not really suit us and we were looking for an open-source and free option. I also need to have a look at [kong](https://github.com/Mashape/kong).

I discovered [express-gateway](http://www.express-gateway.io/) and I gave it a try.

# express-gateway

You can find all the details inside the [github repo](https://github.com/ronanquillevere/express-gateway-plugin-merge-example).
I managed pretty easily to code my own specific policy inside the gateway that does the following :

- Call http://api.chucknorris.io/jokes/random using superagent
- Extract the value field of the response body to get the quote (warning some of them can be offensive, I am not responsible for this API)
- Call http://numbersapi.com/{number} passing the length of the first quote as the number path param
- Extract the text field of the response body
- Return javascript object with the 2 quotes to caller, does not call next(), with following format ```{chuckQuote : quote1, numberQuote: quote2}```

![flow](/img/merge-example-policy.png)

Even if the gateway is still very young I think it is an intesting project to follow !
