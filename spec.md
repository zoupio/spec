# zoup specification

![zoup](assets/zoup.s.png)

* version: `0`
* author: [yetzt](https://github.com/yetzt)

## preface

this is a specification for implementing decentralized and federated tumblelogs reminiscent of what soup.io was before it's disappearance. it outlines a minimum set of requirements for interoperable implementations. this specification is in the public domain and anyone is free to implement it. 

at this point in time this spec does not claim to be complete or usable. it serves as a starting point to which contributions and discussions are welcome.

## basics

anyone can operate their own zoup instance. any instance keeps a stream of posts. posts can be published by the user or aggregated from the feeds of different instances. any instance will make its user-published posts publicly available via a feed. following someone is equivalent to importing their feed. posts from imported feeds can be reposted into the feed of user-published posts. 

## protocol

any zoup instance **must** be accessible via https.

## urls

a zoup instances has a base url in the format `https://domain[/path]`, all api end points are relative to this base url. 

for a base url of `https://zoup.example.org/somepath/` an endpoint of `/feed.json` results in an url of `https://zoup.example.org/somepath/feed.json`.

## user interface

a zoup instance **should** provide a html web interface for humans. 

## meta tags

the [feed](#feed) should be referenced in the instances web interfaces meta tags.

``` html
<link rel="alternate home" type="application/feed+json" href="/feed.json" title="This Zoup's JSON Feed" />
```

## feed

a zoup instance **must** provide a [json feed](https://jsonfeed.org/version/1.1) of the instances user-published posts at `/feed.json`. this should delivered with the `application/feed+json` mime type.

the feed **should** be paginated using the `next_url` field. how so is up to the implementation.

* `version` **must** contain `1.1`
* `title` **must** contain the title of the zoup instance as configured
* `description` **should** contain the description of the zoup instance if configured
* `home_page_url` **must** contain the instances base url
* `feed_url` **must** contain the feeds url `/feed.json`
* `next_url` **must** contain the next batches feed url [as specified](#feed) if there are more entries available.
* `icon` **should** contain a url to the zoup instances profile image with a dimension of `512×512`
* `favicon` **shhould** contain a url to the zoup instances profile image with a dimension of `64×64`
* `authors` **must** be an array containing an object with the property `name` containing the username of the zoup instance.
* `items` **must** contain posts [as specified](#posts)

### example

``` javascript
{
	"version": "https://jsonfeed.org/version/1.1",
	"title": "Example's zoup",
	"description": "An exampe zoup",
	"home_page_url": "https://zoup.example.org/",
	"feed_url": "https://zoup.example.org/feed.json",
	"next_url": "https://zoup.example.org/feed.json?before=<last_date_published>",
	"icon": "https://zoup.example.org/asset/icon-512.png",
	"favicon": "https://zoup.example.org/asset/icon-64.png",
	"authors": [{ "name": "example user" }],
	"items": [
		...posts
	]
}
```

## posts

posts follow the [items definition of the json feed spec](https://jsonfeed.org/version/1.1#items-a-name-items-a).
data specific to zoup will be prefixed by `_` in accordance with the [json feed spec](https://jsonfeed.org/version/1.1#extensions-a-name-extensions-a)

### id

every post **must** have one id, which
* **must** consist only of [unreserved characters](https://datatracker.ietf.org/doc/html/rfc3986#section-2.3)
* **must** be unique within a zoup instance
* **must** be between 1 and 255 characters long, but **should** be reasonably short

### url

the url of a post **must** contain the id. 
the url suffixed by `.json` **must** deliver the json of the post.

### content_html

posts **must** contain the html content of the post in `content_html`. this **must not** contain any executable code, style information or relative urls.

this **must** be [sanitized](#sanitizing) before display.

### content_text

posts **may** contain a text representation of `content_html`.

### tags

`tags` are optional, but if present **must** consist only of [unreserved characters](https://datatracker.ietf.org/doc/html/rfc3986#section-2.3)

### title

posts **may** contain a `title`. 

### authors

posts `must` contain one author object, which 

### attachments

media uploaded by the user **should** be referenced in `attachments`. 

### external_url

posts **should** contain an `external_url` if the post is about one specific url.

### _zoup

is an object of zoup specific extensions to the json feed spec. 

### _zoup.from

if the post was reposted from another zoup instance, `_zoup.from` **must** contain `url`, `name` and `avatar` (if present) of the *original* post.

### _zoup.via

if the post was reposted from another zoup instance, `_zoup.via` **must** contain `url`, `name` and `avatar` (if present) of the *reposted* post.

### _zoup.reposts[]

if there are reposts of this post from other instances (see [ping](#ping)), `_zoup.reposts[]` **may** contain `url`, `name` and `avatar` (if present) of the *repost*.

### _zoup.reaction

if this post is a reaction to another post, `_zoup.reaction` **must** contain `url`, `name` and `avatar` (if present) of the post *reacted to*.

### _zoup.reactions[]

if there are reactions to this post from other instances (see [ping](#ping)), `_zoup.reactions[]` **may** contain `url`, `name` and `avatar` (if present) of the post *reacting*.

### example

``` javascript
{
	"id": "<id>",
	"date_published": "<date>",
	"date_modified": "<date>",
	"url": "https://zoup.example.org/post/<id>",
	"title": "my first post",
	"tags": ["nsfw"], 
	"content_html": "...",
	"external_url": "https://example.com/example.html",
	"attachments": [{
		"url":
		"mime_type":
	}],
	"authors": [{
		"name": "username",
		"url": "https://example.org/".
		"avatar": "https://example.org/assets/avatar.png",
	}],
	"_zoup": {
		"from": {
			"url": "https://another-instance.example.org/post/<id>",
			"name": "another-username",
			"avatar": "https://another-instance.example.org/assets/avatar.png"
		},
		"via": {
			"url": "https://another-reposter.example.org/post/<id>",
			"name": "another-reposter",
			"avatar": "https://another-reposter.example.org/assets/avatar.png"
		},
		"reposts": [{
			"url": "https://reposter.example.org/post/<id>",
			"name": "reposter",
			"avatar": "https://reposter.example.org/assets/avatar.png"
		}],
		"reaction": {
			"url": "https://more-zoups.example.org/post/<id>",
			"name": "more-zoups",
			"avatar": "https://more-zoups.example.org/assets/avatar.png"
		},
		"reactions": [{
			"url": "https://different-zoup.example.org/post/<id>",
			"name": "different-zoup",
			"avatar": "https://different-zoup.example.org/assets/avatar.png"
		},{
			"url": "https://another-zoup.example.org/post/<id>",
			"name": "another-zoup",
			"avatar": "https://another-zoup.example.org/assets/avatar.png"
		}],
	}
}
```

## reposts

reposts from an external feed to the instances feed:

* to avoid collisions, `id` must be unique within the feed
* `_zoup.from` must be populated if not present
* `_zoup.via` must be populated
* the `url` must point to an address within the instance
* `date_published` and `date_modified` must be set to current values
* no other changes

## streams

zoup instances **may** provide websocket streams for their feed. 

* `wss://domain[/pathname]/stream.json`

every message within the websocket stream **must** be the json representation of a post.
if websockets streams are available this **should** be announced in the corresponding feed

```
// ...
"hubs": [{
	"type": "websocket",
	"url": "wss://domain[/pathname]/stream.json"
}], 
// ...
```

## cors

all public api endpoints used via webbrowser by other instances **should** allow cross-origin-requests by setting the appropriate cors headers. this may be limited to the origins of zoup instances followed or explicitly allowed

## ping

pings let zoup instances know that on another instance something has happened in relation to them. since this bears the potential for spam, these should be filtered and can be discarded

all parameters **must** be url-encoded.

`/ping/follow?url=<feed-url.json>`

another zoup instance has started following this zoup instance.

`/ping/repost?url=<post-url>`

another zoup instance has reposted a post from this zoup instance.

`/ping/reaction?url=<post-url>`

another zoup instance has published a reaction to a post from this zoup instance.

## intents

intents are endpoints that let authenticated users do something to their instance. they are meant to be found by other instances via [discovery](#discovery). they are not apis and require interaction by an authenticated user. if they are accessd by an unauthenticated user, they should provide means of authentication. 

all parameters **must** be url-encoded.

### follow

`/intent/follow?url=<feed-url.json>`

follow a zoup instances feed

### repost

`/intent/repost?url=<post-url>`

repost a post

### react

`/intent/react?url=<post-url>`

react to a post

## discovery

the zoup instances web interface [resgisters a protocol handler](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/registerProtocolHandler) with the scheme `web+zoup` for authorized users. third party instances can then use this to discover the zoup instances URL by loading a resource with this protocol in an iframe*. the loaded url then uses [postMessage()](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) to transmit the URL to the third party website.

\* *using custom protocol schemes do not work with fetch. even in an iframe they might break in the future.*

the zoup instance **should** provide and endpoint to handle custom scheme requests.

*all parameters must be url-encoded*

`/discover?url=<custom-url>`

custom urls can be of the following formats

* `web+zoup://follow?url=<feed.json>` should redirect to the follow [intent](#intents).
* `web+zoup://repost?url=<post-url>` should redirect to the repost [intent](#intents).
* `web+zoup://react?url=<post-url>` should redirect to the react [intent](#intents).
* `web+zoup://discover?url=<zoup-url>` should provide the zoup instances base url via `postMessage`

### example

1\. zoup-a registers a protocol handler beforehand to make it discoverable

``` javascript
navigator.registerProtocolHandler("web+zoup", "https://zoup-a.example.org/discover?url=%s");
```

2\. if not known, zoup-b uses the custom protocol scheme to discover the users zoup instance url

```
<iframe id="discover" src="web+zoup://discover?url=https%3A%2F%2Fzoup-b.example.org%2F"></iframe>
```

*depending on the brosers privacy settings the user might need to confirm this*

The URL is then handled by the browser and transformed into

```
https://zoup-a.example.org/discover?url=
	web%2Bzoup%3A%2F%2Fdiscover%3Furl%3Dhttps%253A%252F%252Fzoup-b.example.org%252F
```

3\. zoup-a decodes this input and sends a `postMessasge()` to zoup-b in the parent window

``` javascript
// (this should better be done by the backend)
const origin = new URL(
	new URL(location).searchParams.get("url")
).searchParams.get("origin"); 

window.parent.postMessage(JSON.stringify({
	url: "https://zoup-a.example.org/", 
}), (origin || "*"));
```

4\. zoup-b retrieves the message and now knows the users zoup

``` javascript
window.addEventListener("message", function(message){
	if (document.getElementById('discover').contentWindow === event.source) {
		const zoup_url = JSON.parse(message.data).url;
		document.cookie = 'zoup_url='
			+encodeURIComponent(zoup_url)
			+';path=/;domain=zoup-b.example.org;'
			+'max-age=31536000;secure;sameseite=strict';
	}
}, false);
```
*all code is example code, and should implemented in a sane way*

### when to use discovery

discovery should only happen when the user performs an action that requires interaction with their zoup, e.g. follow, repost, react; If no message is received from the iframe in a reasonable amount of time, the user should be asked to provide their zoup url manually. 

### fallback

if discovery fails, the user should be asked for their zoup url.

### different approaches

in the future discovery might be provided by a browser extension in addition.

## sanitizing

html imported from external sources must undergo sanitization. the recommended allowlist for html tags and attributes is

`<a href name data-src data-width data-height>`, `<abbr title>`, `<b>`, `<bdi>`, `<bdo>`, `<blockquote cite>`, `<br>`, `<caption>`, `<cite>`, `<code>`, `<col>`, `<colgroup>`, `<data>`, `<dd>`, `<dfn title>`, `<div>`, `<dl>`, `<dt>`, `<em>`, `<figcaption>`, `<figure>`, `<h1>`, `<h2>`, `<h3>`, `<h4>`, `<h5>`, `<h6>`, `<hr>`, `<i>`, `<img src alt title width height>`, `<iframe src width height allow>` `<kbd>`, `<li>`, `<mark>`, `<ol>`, `<p>`, `<pre>`, `<q cite>`, `<rb>`, `<rp>`, `<rt>`, `<rtc>`, `<ruby>`, `<s>`, `<samp>`, `<small>`, `<span>`, `<strong>`, `<sub>`, `<sup>`, `<table>`, `<tbody>`, `<td>`, `<tfoot>`, `<th>`, `<thead>`, `<time datetime>`, `<tr>`, `<u>`, `<ul>`, `<var>`, `<wbr>`, `<audio controls>`, `<video controls width height>`, `<source src type>`

`href`, `src`, `data-src` and `cite` attributes need to be filtered for `javascript://` pseudo schemes.

`<iframe>` tags should be rendered with sane attributes in the browser `referrerpolicy="no-referrer" sandbox loading="lazy" allow="fullscreen"`.
