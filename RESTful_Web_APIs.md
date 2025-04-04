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

There were other protocols to the World Wide Web such as the Gopher protocol, but this was not addressable and neither was FTP. FTP also had long lived, indefinite sessions, unlike HTTP which is very time limited. Eventually, all of the different Internet protocols were folded into HTTP GET.

Problems with most Web APIs
* They usually have human-readable documentation that explains how to construct the URLs for a different resource, which violates the correctedness and self-descriptive messages by not using, e.g. tagged hypermedia instead
* Websites have help docs but it is easier to click around instead, APIs tend to have a big menu of options rather than an interconnected web
* Integrating with a new API requires custom software or installing one-off libraries rather than just being interoperabnle and when they have self-descriptive representations
* When APIs change, the custom API clients break and have to be fixed but when a website undergoes a redesign, the site's users adapt, i.e. the website redesign is encapsulated in the self-descriptive HTML documents that are understandable from the previous ones.

The problem with web API design is to bridge the semantic gap between understanding a document's structure and understanding what it means, the semantic challenge.

## 2. A Simple API

If you have a URL starting with http:// or https:// you should start with an HTTP GET request. We know the URL to a resource and nothing else, so we need to find the options.

An example of this would be:

```bash
$ wget -S -O - http://www.youtypeitwepostit.com/api/
```

This uses the `Wget` command line tool with the `-S` option, which prints out the full HTTP response from the server, and the `-O` option, which prints the document rather than saving it, and in effect sends the following HTTP request:

```HTTP
GET /api. HTTP/1.1
Host: www.youtypeitwepostit.com
```

This is a request for a representation from the server, and as it doesn't change the data, it is *safe*.

The response we get to the GET request is in three sections:

1. The status / response code, such as:

```
HTTP/1.1 200 OK
```

2. The entity-body / the body, which is a document in some format that you have to understand

3. The response headers, a series of key-value pairs desribing the entity-body and the HTTP response in general, such as

```
ETag: "f60e0978bc9c458989815b18ddad6d75"
Last-Modified: Thu, 10 Jan 2013 01:45:22 GMT
Content-Type: application/vnd.collection+json
```

The most common versions of this are `text/html` and `image/jpeg` or similar, but we also get things like `vnd.collection+json`.

JSON is described in RFC 4627 and is a standard for representing siple data structures in plain text, using double quotes for strings, square brackets for lists and curly brackets for objects. The above is not a simple JSON document, which would be `Content-Type: application.json` not `Content-Type: application/vnd.collection+json`.

Collection+JSON is a standard for publishing a searchable list of resources on the web, and puts constraints on the JSON. These require the server to serve a JSON object, i.e. in `{}` rather than just any document. It also has to have a property called collection mapped to another object, which should have a property called items that maps to a list, which itself needs to be objects as in:

```
{"collection": {"items": [{}, {}, {}]}}
```

Collection+JSON is a way of serving lists that themselves describe HTTP resources. 

The string is defined as "the address used to retrieve a representation of the document". Each object in the `items` list has an `href` property, and the value is a string with a URL. Collection+JSON allows you to talk about resources and URLs, which JSON in general does not.

To use the API you need to use a `template` object to compose a valid `item` representation and then use HTTP POST to send that to the server for processing. 

The `template` property is the `template object`, and looks like:

```
{
  ...
  "template": {
    "data": [
      {"prompt": "Text of message", "name": "text", "value":""}
    ]
  }
}
```

Where we replace the empty string in `value` with the string to publish.

The full template filled out with the HTTP POST request looks like:

```
POST /api/ HTTP/1.1
Host: www.youtypeitwepostit.com
Content-Type: application/vnd.collection+json
{ "template":
  {
    "data": [
      {"prompt": "Text of the message", "name": "text", "value": "Squid!"}
    ]
  }
}
```

With this template being a valid Collection+JSON by itself.

This would lead to a server response like:

```
HTTP/1.1 201 Created
Location: http://www.youtypeitwepostit.com/api/47210977342911065
```

Where the 201 response code means that it is OK and that there is a new resource under the Location header.

HTTP POST is how a resource is created, and has the following specs from RFC 2616:
* Annotation of existing resources
* Posting a message
* Providing a block of data to a data handling process
* Extending a database through an append operation

