---
layout: page
title: About me
subtitle: 
---

I'm a software engineer working at [New Relic](https://newrelic.com). I'm Irish, but I'm based in the sunny Barcelona. I work on a mostly a bunch of ingest services with a handful of API services, the vast majority of which are on the JVM. Most code I write these days is in Kotlin, still with some in Java and the occasional Node.js application.

We use Kafka for our ingest. We have many standard kafka consumers and numerous kafka streams applications, all written in Kotlin. Our biggest consumer consumes over 1 million messages a second, so we've had some fun learning how to scale consumption! Everything we consume ends up either cached in a redis store or an elasicsearch cluster. 

On the elastic side, we have had some fun special cases trying to cater for create a custom search experience for each user, even with a shared data store for all accounts which has been a lot of fun

If you have interest in collaborating with me on either of the two technologies, don't hesitate to contact! 

Thanks for reading!

Cian 




