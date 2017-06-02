# [feeder.co](https://feeder.co/) REST API

[feeder.co](https://feeder.co) supports a REST API using an OAuth2 flow. To be able to authenticate you need a client ID and a client secret that you can get by e-mailing us at [support@feeder.co](mailto:support@feeder.co). Be sure to include what you wish to do with it and a bit of info about who you are.

## Conventions

### JSON

Our JSON structure is a namespaced key with underscored object property names. For example:

```json
{
  "feed": {
    "id": 123,
    "title": "feeder.co blog",
    "path": "http://blog.feeder.co/rss/index.xml",
    "num_posts": 30,
    "... etc": "... etc"
  }
}
```

Property names can be quite inconsistent because of legacy reasons. Sorry about that.

### Errors

When creating or updating objects validation errors might occur. They are passed inside the object they referr to:

```json
{
  "feed": {
    "title": "feeder.co blog",
    "errors": {
      "base": ["An error"],
      "title": ["Specific error"]
    }
  }
}
```

### Paths

The root path to access the REST API is:

`https://feeder.co/1/`

Paths follow a fairly standard REST convention. As our backend is built on [Ruby on Rails](http://rubyonrails.org) so if you are familiar with that it should be fairly straightforward.

- `GET /1/feeds` to fetch a user's feeds
- `POST /1/feeds` to create a feed
- `GET /1/folders` to fetch a users folders
- `PUT /1/folders/:id` to update a folder
- `DELETE /1/folders/:id` to delete a folder
- ... etc

## Authentication

To authenticate you need to open a web view asking the user to log in and give your app permissions to act on their behalf. Our Oauth2 flow uses standardized tools and should integrate into existing libraries.

All authentication requests go to the path:

`https://feeder.co/1/oauth2`

To login a user you need to: open a browser with the URL:

```
https://feeder.co/1/oauth2/authorize?
  client_id=$CLIENT_ID&
  redirect_uri=$REDIRECT_URI&
  response_type=code
```

The `$REDIRECT_URI` has to match with a URL you passed to us when setting up the application.

<img width="628" alt="screenshot 2017-06-02 14 12 45" src="https://cloud.githubusercontent.com/assets/21059/26725179/994383f4-479d-11e7-8348-2715cf7e93e5.png">

When the user accepts the browser will be redirected to your `$REDIRECT_URI` with a `?code=` query string parameter.

**This `code` is what you save locally.**

This code is used to retrieve access tokens that authorize API requests on the user's behalf.

To get an access token your code must run a `POST` request:

```
POST https://feeder.co/1/oauth2/token?
  grant_type=authorization_code&
  code=$CODE_HERE&
  redirect_uri=$REDIRECT_URI&
  client_id=$CLIENT_ID
```

- `$CODE_HERE` is the `?code=` from the previous step
- `$REDIRECT_URI` must exactly match the URI from the previous step
- `$CLIENT_ID` your client ID.

This will return a JSON response that looks like this:

```json
{
  "access_token": "tokentokentokentokentokentoken...",
  "token_type": "bearer",
  "expires_in": 7199,
  "created_at": 1496406467
}
```

To call the API you simply pass in an `Authentication`-header with the request:

```
GET /1/feeds HTTP/1.1
Host: feeder.co
Authentication: Bearer tokentokentokentokentokentoken...
```

**It is highly recommended to use an existing OAuth2 libray for your platform.**

## Feeds

```
GET      /1/feeds
POST     /1/feeds
GET      /1/feeds/:id
PUT      /1/feeds/:id
DELETE   /1/feeds/:id
POST     /1/feeds/mark-all-as-read
POST     /1/feeds/:feed_id/mark-as-read
```

## Folders

```
GET      /1/folders
POST     /1/folders
GET      /1/folders/:id
PATCH    /1/folders/:id
PUT      /1/folders/:id
DELETE   /1/folders/:id
```

## Posts

```
GET      /1/posts
POST     /1/posts
GET      /1/posts/:id
PATCH    /1/posts/:id
PUT      /1/posts/:id
```

### Starred posts

```
GET      /1/posts/starred
```

### Unread posts

```
GET      /1/posts/unread
```

## Unread counts

**Do not attempt to count unreads yourself**. This is error prone and buggy. **Do not** fetch all feeds and posts and store in a local database and run a `SELECT COUNT(*) FROM posts WHERE is_read = false`. It is inefficiant and almost guaranteed to not sync correctly across devices.

Use the API below. It is highly recommended to consider "unread counts" as a separate model in a local object store.

```
GET      /1/feeds/unread
```

## Pagination

Most `GET` endpoints that return an array support `limit` and `offset` query parameter that can be used for pagination.
