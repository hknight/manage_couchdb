# Manage CouchDB: Common management and maintenance

Manage CouchDB is a Couch app ("design document") to do all the common stuff we all need.

It has identical implementations in **JavaScript** (for clarity and portability) and **Erlang** (for performance). Both implementations pass the same test suite.

Just drop it in your database (the id is `_design/couchdb` for JavaScript, and `_design/ecouchdb` for Erlang) to get several features.

## A view of all conflicts

Hit `/_view/conflicts` to see all conflicts in the database.

* Use `?group_level=1` to get a conflict count for each document
* Use `?reduce=false` to get one row per `[id,revision]` pair.
* Use `?reduce=false&include_docs=true` to get every conflicted revision in one big batch.

## Some useful rewrites

Manage CouchDB includes some simple rewrites if you want to build from it.

* `/_rewrite/_couchdb` hits the root CouchDB API
* `/_rewrite/_db` hits the current database
* `/_rewrite/_ddoc` hits the manage_couchdb design document itself

## An easy validation function

To make a database read-only, edit this design document (e.g. `_design/couchdb`), and set `.access.read_only` = **true**.

When a database is read-only, normal users *may not* modify any documents. Admins *may* make changes, but a note is placed in the CouchDB log file.

## Refilter: a powerful regular-expression filter function

The filter function, `refilter`, is great for filtered replication or `_changes` feeds. Just provide:

* The **key** in the document
* A **regex** it should match

Refilter will *pass* documents that match.

### Refilter example

Suppose your documents have a *created_at* value, with a timestamp.

    { "_id": "example_doc"
    , "created_at": "2011-06-19T00:42:20.079Z"
    , "value": 23
    }

To get *documents from June this year and last*, drop this document in the `_replicator` database.

    { "_id": "docs created in June"
    , "source": "the_big_database"
    , "target": "june"
    , "filter": "ecouchdb/refilter"
    , "query_params": { "field": "created_at"
                      , "regex": "^(2011|2012)-06"
                      }
    }

## Test suite

Manage CouchDB uses [node-tap][tap]. Start a CouchDB in Admin Party at `http://localhost:5984` and run the suite via the starter script. The script will do several things:

    $ ./test.sh
    # Looking for CouchDB
    {"couchdb":"Welcome","version":"1.2.0"}

    # Configuring CouchDB
    "{couch_native_process, start_link, []}"
    {"ok":true}
    {"ok":true}

    # Pushing JavaScript app
    Reading dependency tree...
    loading js
    preprocessor traditional-couchapp/load
    loading js/packages/traditional-couchapp
    Build complete: 44ms
    Uploading...
    OK: http://localhost:5984/test_manage_couchdb/_design/couchdb/_rewrite/

    # Pushing Erlang app
    Reading dependency tree...
    loading erlang
    preprocessor traditional-couchapp/load
    loading erlang/packages/traditional-couchapp
    Build complete: 41ms
    Uploading...
    OK: http://localhost:5984/test_manage_couchdb/_design/ecouchdb

    # Running TAP test suite
    ok tap/conflicts.js ................................. 101/101
    ok tap/lib.js ........................................... 3/3
    ok tap/refilter.js ...................................... 6/6
    total ............................................... 110/110

    ok

## License

Manage CouchDB is released under the Apache license, version 2.

[tap]: https://github.com/isaacs/node-tap
