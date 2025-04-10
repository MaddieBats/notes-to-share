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

The POST request looks like the HTTP responses with a `Content-Type` header and an entity-body

The safety constraint of HTTP GET meands that if you don't know what to do with a URL you can always GET it and look at the representation and nothing bad will happen. The use of JSON also means that you can find things such as types clearly.

You should use the Collection+JSON instead of just JSON to make things more interoperable.

This does not constrain the sorts of items that the collection should make up, and these are the application semantics that vary from one application to another.

## 3. Resources and Representations

REST is not a protocol, file format or development framework, but is instead a set of design constraints, called the Fielding constraints. 

Underlying the three web technologies of the URL, HTTP and HTML, are resources and representations. 

"A resource is anything that's important enough to be referenced as a thing in itself", i.e. anything you want to link to, asser about, retrieve or include. It is usually storable on a computer, and must have a URL. A client will only ever see the URLs and the representations.

A representation describes a resource state. Issuing a GET request should lead to a document that captures the current state of the resource, depending on what the client asks for. A representation can be any machine readable document containing any information about a resource. 

Resources are what get passed between clients and servers and a representational transfer is where the server sends a representation describing the state of a resource and the client sends a representation describing the state it would like the resource to have.

The two strategies for specifying what representation the client wants are:
* Content negotiation, where the client distinguishes between the representations based on the HTTP header
* Giving the resource a URL for each representation possible

When resources have multiple URLs, there should be a canonical URL. 

The HTTP standard has eight kinds of messages and the four most common are:
* GET : Get a representation of the resource
* DELETE: Destroy the resource
* POST: Create a new resource under this one
* PUT: Replace the state of this resource with the one described

And there are to methods commonly used for exploring an API:
* HEAD : Get the headers that would be sent along with the representation, but not the representation itself
* OPTIONS : Discover which HTTP methods the resource responds to.

There are also CONNECT and TRACE but these are for HTTP proxies.

There is also a ninth HTTP method in the supplement RFC 5789:
* Patch : Modify a part of the state f the resource based on its representation, where if a bit of the resource state is not mentioned in the given representation it is left, kind of like PUT but more fine-grained

There are also two extension HTTP methods:
* LINK: Connect a resource to another one
* UNLINK : Destroy the connection between two resources

These above are the protocol semantics of HTTP. 

GET is a safe HTTP method as it is just a request for information, and will not change the resource state other than incidental side effects. 

DELETE asks the server to remove a resource, as in:

```
DELETE /api/45ty HTTP/1.1
Host: ww.youtypeitwepostit.com
```

And you would get back a 204 if it has been deleted:

```
HTTP/1.1 204 No Content
```

You might also get 200 for "it's deleted" and 202 for "accepted, it is scheduled be deleted". If you then try to GET a DELETEd resource, you will get a 404 (not found) or 410 (gone). DELETE is not a safe method as it changes the resource, but it is idempotent, in that sending a request twice has the same effect on the resource state as sending it once. Every safe operation is also idempotent.

POST has two jobs:
* POST-to-append sends a POST request to crate a new resource underneath it, as in:

```
POST /api/ HTTP/1.1
Content-Type: application/vnd.collection+json
{
  "template" : {
    "data" : [
      {"name" : "text", "value" : "testing"}
    ]
  }
}
```

Which will hopefully give a 201 (created) response code, although you might get 202 (accepted). POST is neither safe not idempotent, in that it changes the resource and there is an effect from doing it more than once. 

* Overload POST, which is about creating a new resource

PUT requests a modification to the resource state. You take the representation from the GET request, modify it and send it back as the payload for the PUT request, as in:

```
PUT /api/q1w2e HTTP/1.1
Content-Type: application/vnd.collection+json

{
  "template" : {
    "data" : [
      {"name" : "text", "value" : "tasting"}
    ]
  }
}
```

The server is free to reject a PUT request if the entity-body doesn't make sense or for any other reason, and the response code is likely to be 200 (OK) or 204 (No Content). PUT is idempotent in that if you send the request multiple times there is no added effect. 

A PUT request can also create a new resource if the client knows the URL where the resource should live, but is stil idempotent.

PATCH is like PUT but it only works on a small "diff" representation of the total reprsentation. For example,

```
PATCH /my/data HTTP/1.1
Host: example.org

Content-Length: 326
Content-Type: application/json-patch+json
If-Match: "abc123"
[
  { "op": "test", "path": "/a/b/c", "value": "foo" },
  { "op": "remove", "path": "/a/b/c" },
  { "op": "add", "path": "/a/b/c", "value": [ "foo", "bar" ] },
  { "op": "replace", "path": "/a/b/c", "value": 42 },
  { "op": "move", "from": "/a/b/c", "path": "/a/b/d" },
  { "op": "copy", "from": "/a/b/d", "path": "/a/b/e" }
]
```

And you would get the same response codes as PATCH and DELETE. This is neither safe nor idempotent, as with POST. Note that this is a recent extension designed for web APIs. 

LINK and UNLINK manage the hypermedia links between resources, as in:

```
UNLINK /story HTTP/1.1
Host: www.example.com
Link: <http://www.example.com/~omjennyg>;rel="author"
```

These are idempotent but not safe. 

HEAD is a safe method that is a lightweight version of GET that only supplies the status code and the headers, saving bandwidth.

OPTIONS is a discovery mechanism for HTTP which contains the HTTP `Allow` header laying out th eHTTP methods that a resource supports, as in:

```
OPTIONS /api/a1s2d3 HTTP/1.1
Host: www.youtypeitwepostit.com

200 OK
Allow: GET PUT DELETE HEAD OPTIONS
```

POST can also convey any kind of change to a resource. This is often used in an HTML form because an HTML form can't trigget a PUT request. The data-handling process that it is providing a block of data for is so broad that it does not have any protocol semantics, and this is called overloaded POST. As these don't have protocol semantics, you have to provide reliable application semantics. 

RESTful systems are made up of independent compoents that are created by a variety of users and communicated by passing documents back and forth over HTTP. HTTP protocol semantics are mostly defined by HTTP methods but there is redundancy in these (where, e.g. PUT can substitute for PATCH). 

If you want an API entirely described by HTML documents then you need the protocol semantics to just use GET and POST.

## 4. Hypermedia

URLs identify resources and a client makes an HTTP request to those URLs. The server sends representations in response and the client builds picture of the resource state, with the client making a PUT/POST/PATCH request to send the repreentation back to the server to modify the resource state. The client knows what requests it can make and what the entity-body should look like by using hypermedia.

Hypermedia connects resoures to each other and describes their capabilities in a machine-readable way. Hypermedia is strategy that i sthen implemented by different technologies, and have the goal of making the server tell the client what HTTP requests the client might want to make. 

HTML is a hypermedia format, and `<a>` is a hypermeida control tag, which is a description of an HTTP request the browser might want to make in the future.

Within the HTML there is a series of possible requests that you can make as a client, such as with the `<img>` tag:
* The `<a>` tag describes a GET request for a specific URL if the user triggers the control
* The `<img>` tag desribed a get request for a specific URL which happens automatically in the background
* The `<form>` tag with `method="POST"` describes a POST request to a specific URL with a custom entity-body constructed by the client if the user triggers the control
* The `<form>` tag with `method="GET"` describes a GET request to a custom URL constructed by the client if the user triggers the control

There are some further hypermedia controls but they all fall under the Fielding definition that "Hypermedia is defined by the presence of application control information embedded within, or as a layer above, the presentation of information". 

This application control information is what distinguishes an HTML document from a book, in that it allows for connections. 

URLs created using the HTML tag are of a certain format, but they don't have to be. We use URI templates and RFC 6570 to explain how we can turn a string into a set of URLs, so `http://www.youtypeitwepostit.com/search/{search}` can be turned into a series of different URLs with `{search}` switched out. 

There is a direct translation of the URI templates into valid URLs. For instance if `hello := "Hello, World!"`, then the URI template `http://www.example.org/greeting?g={+hello}` will expand to the URL `http://www.example.org/greeting?g=Hello%20World!`.

A URI Template is shorter and more flexible than an HTML GET form but the achieve a similar thing in allowing you to describe an infinite number of URLs with a short string. A URI Template has to be embedded in a hypermedia format, and it means that we do not need custom formats.

A URL is a short string used to define a resource, like a URI, every URI is a URL and they are described in RFC 3986. However, URIs do not have to have a representation, so a URL is an identifier that can be dereferenced. For instance, whilst an `http:` URI or an `ftp:` URI can have protocols associated with them for obtaining a representation of a resource, something like `urn:isbn:9781449358063` is the URI for the abstract concept of the edition of this book, which obviously doesn't have a protocol.

Without a representation there cannot be a representational state transfer, it can't be self-descriptive (as it can't send messages). Generally, when your API refers to a resource, it should use a URL with `http` or `https` and the URL should work by showing a useful representation in response to a GET request.

#### The Link Header

RFC 5988 defined the `Link` header extension to HTTP which lets you put hypermedia controls into entity-bodies such as JSON objects and binary image files. 

For instance, if we had a story split into chapters, we could use the link in the header to point to the second chapter:

```
HTTP/1.1 200 OK
Content-Type: text/plain
Link: <http://www.example.com/story/part2>;rel="next"

It was a dark and story night (continued in part 2).
```

This is roughly similar to an HTML `<a>` tag. We can use the LINK and UNLINK extension methods with the `Link` header.

#### What Hypermedia Is For

Hypermedia controls have three jobs:
* Telling the client how to construct an HTTP request, which HTTP method to use, what URL to use, what HTTP headers and/or entity-body to send
* To make promises about the HTTP response with the status code, HTTP headers and/or the data the server is likely to respond with
* To suggest how the client should integrate responses in the workflow

HTML GET forms and URI Templates both tell the client how to contruct a URL to be used in an HTTP GET request.

An HTTP request has four parts:
* A method
* A target URL
* The HTTP headers
* The entity-body

The `<a>` tag specifies the target URL and HTTP method to use, with the target URL in the `href` attribute, and the HTML specification of the tag becoming a GET request when the end user clicks a link. Notably, a URI template only specifies the target URI rather than anything about the HTTP request itself, it doesn't tell you whether to make a GET request or POST request etc. This means that you have to combine the URI Template with another hypermedia technology.

Unlike the `<a>` tag, the `<img>` tag supplies a promise to the client that the server will also send some form of image representation in response to the GET. We can also use a simple XML hypermedia control from the Atom Publishing Protocol, as with `<link rel="edit" href="http://example.org/posts/1/"/>` and the `rel="edit"` means that the resource supports both PUT and DELETE as well as a GET.
