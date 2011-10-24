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
