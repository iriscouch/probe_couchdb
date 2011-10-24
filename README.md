# Automated, comprehensive, event-driven CouchDB probing

Probe CouchDB is a Javascript library which digs into every corner of a CouchDB server and fire events when it find interesting things: users, configs, databases, design documents, etc.

Probe CouchDB is available as an NPM module.

    $ npm install probe_couchdb

You can also install it globally (`npm install -g`) to get a simple `probe_couchdb` command-line tool.

## Is it any good?

Yes.

## Usage

Probe CouchDB is an event-emitting object. Give it a URL and tell it to start.

```javascript
var probe_couchdb = require("probe_couchdb");

var url = "https://admin:secret@example.iriscouch.com";
var couch = new probe_couchdb.CouchDB(url);

couch.start();
```

Next, handle any events you are interested in.

```javascript
couch.on('db', function(db) {
  console.log('Found a database: ' + db.url);
  db.on('metadata', function(data) {
    console.log(db.name + ' has ' + data.doc_count + ' docs, using ' + (data.disk_size/1024) + 'KB on disk');
  })
})
```

<a name="api"></a>
## API Overview

This is the object hierarchy: **CouchDB** &rarr; **Database** &rarr; **Design Document**

* A *CouchDB* emits events describing the main server, including zero or more `db` events, containing a *Database* object.
* A *Database* emits events describing the database, including zero or more `ddoc` events, containing a *design document* object.
* A *Design Document* emits events describing the design document.

### Common Events

All events pass one parameter to your callback unless otherwise noted.

* `error` | An Error object indicating a problem. Databases re-emit all design document errors, and CouchDBs re-emit all database errors.
* `end` | Indicates that this object's work is done. *Zero callback arguments*

### Common properties

* `log` | A log4js logger. Databases inherit the log from CouchDBs, design documents inherit the log from databases.
* `url` | The url to this resource (either a couch, a database, or a design doc)

### Common methods

All objects have a `.request()` method which wraps a [request](https://github.com/mikeal/request) call.

## CouchDB Probes

You create these using the API.

### Events

* `couchdb` | The server "Welcome" message (server is confirmed)
* `session` | The session with this server (`/_session` response). Check `.userCtx` to see your login and roles.
* `config` | The server configuration (`/_config` response). If you are not the admin, this will be `null`.
* `users` | A list of all user documents (usually in the `_users` database). This will *always* include an anonymous user document: `{"name":null, "roles":[]}`
* `db` | A *Database* probe. If you care about that database, subscribe to its events!

These events are used internally and less useful:

* `dbs` | An array of databases on this server
* `end_dbs` | Indicates that all databases have been processed

### Properties

* `only_dbs` | *Either* an array, to probe only specific databases, *or* a `function(db_name)` which returns whether to probe that database.
* `max_users` | Emit an error if the server has more users than this number.

### Methods

* `start()` | Start probing the server
* `anonymous_user()` | Helper function to produce an anonymous userCtx: `{"name":null, "roles":[]}`
