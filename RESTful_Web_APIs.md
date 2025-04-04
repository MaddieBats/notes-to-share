# RESTful Web APIs

## 0. Introduction

Why you need to have an API that is able to change:
* The more commercially sucessful an API is, the longer it will last, which means that the problem domain will need to change too
* There may be constant additions of data elements and business rules
* Clients might need to chnage the workflow dependent on circumstances
* If the API is open to non-devs then you need to have a design that fits all of these end users

Two problems that need to be solved:
* Each API takes completely different designs rather than building on other APIs - "there's something about APIs that makes everyone want to design their own from scratch"
* You need to understand the actual REST principles when designing the APIs, including an understanding of hypermedia that allows linkages, and until that happens REST will remain a buzzword

Types of standards used:
* Fiat standards are behaviours and are a description of the general way that people do things. The core assumptions of a standard, that other people should do the same, is misisng, and this refers to things like the "Twitter API" or the "Facebook API". A large amount of time and effort is locked into these fiat standards that can't be reused. In general, a fiat standard should be a gloss over other standards.
* Personal standards are one person's opinion on what a good design looks like. They are often less formal and might later be adopted into more open standard.
* Corporate standards are created by a consortium of companies trying to solve a problem that is collective. These are often better defined and use formal language, but do not have more inherent force than a personal standard. Industry standards often start out as corporate standards.
* Open standards are designed by committee or open comment and is recognised by a standards body, which gives it more force. These come with an agreement that you can implement them without patent infringement. The most important is the IETF, which manages Requests for Comments (RFCs), which are created through a Standard Track. RFCs stard as an internet draft which are meant to expose problems with the specification, and last up to six months before being approved as an RFC or updated with a new draft. Each RFC or internet draft is given a code name (e.g. HTTP/1.1 is RFC 2616)

## 1. Surfing the Web

Model of the web as a platforom for distributed computing and is broadly based on: a URL naming convention, the HTTP protocol, and the HTML document format. 

Statelessness gets at the fact that a server doesn't care about what state the client is in.

HTTP has eight methods that a client can apply to a resource, including GET, HEAD, POST, PUT, and DELETE, as well as the extension method PATCH,, which i used in web APIs. This is not like a progamming language where you can define a bunch of your own methods.

Each state of a page, in REST terms, is referred to as an application state and each transition from one application to another corresponds to a link or form.

The resource state is the overall state that the site is in and is the server-side state, compared with the application state, which is client side (although the server can manipulate this by sending representations like HTML documents. The resource state is server side but the client can manipulate it by sending the server a representation.

Connectedmess is the propert of websites to explain how to get to adjoining pages. This can be described by the principle of "hypermedia as the engine of application state". 

There were other protocols to the World Wide Web such as the Gopher protocol, but this was not addressable and neither was FTP
