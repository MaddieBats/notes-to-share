# Production Ready GraphQL

*Building well designed, performant and secure GraphQL APIs at scale*

*Marc-Andre Giroux*

## Introduction

Prior to GraphQL becoming popular, the architecture that people were using was generally Enpoint-based APIs which revolved around HTTP endpoints, such as a JSON API over HTTP. The endpoints are simple to implement and are very good at working with one use case, are chacheable, discoverable,a dn simple to use. 

These are less good if you have to serve multiple use cases. People started trying to make one-size-fits-all APIs which tried to do too much at once. One solution was to have a different endpoint for each variation. There were also aproaches that had one endpoint per use case but had a query parameter for each different type, and some tried to just use partial and full queries. Other approaches included letting clients select what they wanted back from the server or writing the query language in the query parameter. There was then a movement to make a fully customisable API that was actually good for developers. 

SoundCloud tried to solve this by having a different server for each sort of user (called Backends for Frontends).

Facebook announced the GraphQL release in 2015
