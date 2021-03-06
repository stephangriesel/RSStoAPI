# RSS to API

RSS to API exposes an API for retrieving RSS feeds, and includes caching to increase speed.

**Key use case**: Access RSS feeds from your client-side JavaScript easily and quickly.

## Why this exists

Retrieving RSS feeds directly through JavaScript generally does not work due to cross-domain restrictions. 

The Google Feed API provides an easy way to retrieve feeds through JavaScript, but it's officially deprecated and will likely disappear sometime soon.

There are few alternatives to the Google Feed API. The Feedly API is great, but it doesn't work directly from JavaScript and it's rate-limited like any public API.

To retrieve RSS feeds through JavaScript, then, you're forced to host your own proxy. RSS to API makes this a one-click process, through the Deploy to Heroku button below. In no time, you'll be retrieving RSS through JavaScript. You'll benefit from caching, as well, so feeds pop up quickly for your users.

## Caching

RSS to API caches feeds as they're retrieved, using Redis. A Redis server is included when using the Deploy to Heroku button below.

The cache lasts for an hour by default. That is, an individual feed will be retrieved from its source only once per hour, despite the number of requests.

If a request comes in that cannot be served from cache, the app retrieves the feed and returns it.

**For caching to work most effectively, include a count (i.e. how many articles to retrieve) in every request.**

## Usage

Retrieve the contents of an RSS feed.

### Direct

`https://YOUR_DOMAIN/v1/feed?url=http://www.nytimes.com/services/xml/rss/nyt/HomePage.xml&count=8&key=YOUR_KEY`

### JQuery

[Demo](http://codepen.io/csm123/pen/KddKzW?editors=101)

```javascript
$.ajax(
  {
    url: "https://DOMAIN_FOR_THIS_APP/v1/feed?url=http://www.nytimes.com/services/xml/rss/nyt/HomePage.xml&count=8&key=YOUR_KEY"
  }
).done(
  function(data) {
    alert("Found " + data.items.length + " items");
  }
);
```

Parameter | Description
--------- | -----------
`url`    | The URL of the RSS feed. Alternatively, you can specify a `stream_id` as defined in Feedly's API reference. Either the `url` or `stream_id` is required.
`count`   | The number of entries to retrieve from the feed. Default is `DEFAULT_COUNT` or 10. Optional.
`key`     | If you set a key with the `API_KEY` configuration variable, this must match that key for the request to be processed.

### Results

```javascript
{
  "items":
    [
      {
        "title": "Title of article", 
        "link": "Link to article",
        "published": "Date that article was published"
      },
      {
        "title": "Title of article", 
        "link": "Link to article",
        "published": "Date that article was published"
      }
    ],
  "from_proxy_cache": true /* true if from serving this feed from cache; otherwise, false */
}
```

### Error results

```javascript
{
  "error": "Invalid key"
}
```

## Configuration variables

Only the REDIS_URL is required, and it's automatically filled if use the Deploy to Heroku button below. If you use the Deploy to Heroku button, you'll be prompted for these.

Config Variable | Description
--------------- | -----------
`ORIGINS`       | A comma-delimited list of domains that are allowed to submit cross-domain requests to the app.  No need for commas if you're specifying just one domain. If blank, no cross-origin requests will be accepted, unless `ALLOW_ALL_ORIGINS` is set to `true` <br/><br/>Examples:<br/>`http://mydomain.com/,http://someotherdomain.com/`<br/>`http://mydomain.com/`
`ALLOW_ALL_ORIGINS` | Set to `true` to allow any domain to submit cross-domain requests to this app. Not recommended.
`API_KEY`       | Set your own key that will be required to make a request to this API.
`DEFAULT_COUNT` | The number of entries returned for a given feed by default. This can be adjusted per request. If not specified here or in the request, the default is 10.
`NUMBER_OF_ITEMS_TO_CACHE_PER_FEED` | The number of enrties per feed to cache in Redis. If not specified, 25.
`CACHE_LIFE` | The amount of time (in seconds) after fetching a feed to serve requests from the cache. Think about how fresh the feed needs to be for your use case vs. the speed gained from caching. If blank, 3600 or one hour - meaning that feeds will be updated from the source a maximum of once per hour.
`REDIS_URL` | Automatically set if you deploy to Heroku below. Specifies the location of the Redis server used for caching.

## Setup

### Heroku (recommended)

Heroku deployment is easy and free using the button below. It includes the app itself and a Redis server.

The app will return an Internal Server Error for the first few minutes while Redis loads.

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy)

### Local
1. Extract into a directory and run `bundle`
2. Run `bundle exec rackup config.ru`

## Toolset

- Ruby
- Sinatra
- Redis
