# Analytics

Simple tool to track and analyze application events

## Description

In essence, this is client-server app. Client is sending event to server, depending on what actually happing in application. Server persists all data provides API to access it.

This repo contains both server and client (node.js) code. Clients for other platforms are welcome.

### Create client and post events

This is a small example of `analytics` usage:

```js
var analytics = require('analytics');

// create analytics client, by providing app id
var events = analytics('my-web-app');

// call function, with the name of event
events('application started');

// provide event id (optional, but suitable for fetching data)
events({id: 'app-start', event: 'application started'});

// provide additional payload (optional, but usefull for sophisticated analysis)
events('application stated', {id: 'app-start', environment: process.env.NODE_ENV});

// provide callback (optional)
events('application started', function (err) {
	console.log('event posted on server');
});
```

Scenarios for web (express.js) application:

```js
var express = require('express');
var analytics = require('analytics');
var app = express();

// create analytics client, by providing app id
var events = analytics('my-web-app');

var eventsMiddleware = function (event, data) {
	return function (req, res, next) {
		events(event, data, next);
	}
}

app.get('/login',
	eventsMiddleware('user login request'),
	renderLogin
);

app.post('/login',
	checkCredentials,
	eventsMiddleware('user logged on'),
	redirectToApp
);

app.get('/search', function(req, res) {
	var query = req.query['q'];
	search.run(query, function (err, results) {
		events('search executed', {query: query, time: results.timeTakes});
		res.json(results.data);
	});
})
```

### Querying for results

You can query server for data you application collected.

```js
var analytics = require('analytics');
var events = analytics('my-web-app');

// query all collected events
events.query(function (err, res, results) {
	console.log(results);
})

// query by event name
events.query('search executed', function (err, res, results) {
	console.log(results);
});

// query by event type
events.query({id: 'app-start'}, function (err, res, results) {
	console.log(results);
});
```

For convenience, you can request data by certain date.

```js
var analytics = require('analytics');
var events = analytics('my-web-app');

// query all collected events for today
events.query({date: 'today'}, function (err, results) {
	console.log(results);
});

// query all collected events for particular day
events.query({date: '2014-09-26'}, function (err, results) {
	console.log(results);
});

// query all collected events for particular, for event name
events.query('search executed', {date: '2014-09-26'}, function (err, results) {
	console.log(results);
});

// query all collected events for particular, for event type
events.query({id: 'app-start', date: '2014-09-26'}, function (err, results) {
	console.log(results);
});
```

## License

MIT