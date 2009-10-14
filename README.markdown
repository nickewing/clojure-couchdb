#clojure-couchdb

Simple clojure library to interface with [Apache CouchDB](http://couchdb.apache.org/).

Depends on clojure-contrib and [clojure-http-client](http://github.com/technomancy/clojure-http-client/) (hopefully soon a part of contrib).

---

##Database Functions:

###database-list
    user=> (database-list)
    ["foo" "bar" "some-db" "another-database"]
###database-info
    user=> (database-info "foo")
    {:instance_start_time "1242496720047538", :disk_size 4096, :compact_running false, :purge_seq 0, :update_seq 0, :doc_del_count 0, :doc_count 0, :db_name "foo"}
###database-create
    user=> (database-create "new-db")
    "new-db"
###database-delete
    user=> (database-delete "new-db")
    true
###database-compact
    user=> (database-compact "foo")
    true

##Document Functions:

###document-list
    user=> (document-list "some-db")
    ("doc1" "doc2" "another-doc" "yet-another-doc")
###document-get
    user=> (document-get "some-db" "doc1")
    {:bar [1 2 3], :_rev "1-2326402976", :_id "doc1"}
    user=> (document-get "some-db" "doc1" "1-2326402976")
    {:bar [1 2 3], :_rev "1-2326402976", :_id "doc1"}
###document-create
    user=> (document-create "some-db" {:foo 42})
    {:foo 42, :_id "5bc3673322f38a4998aca23976acd4c6", :_rev "1-1799037045"}
    user=> (document-create "some-db"  "my-doc" {:foo 42})
    {:foo 42, :_id "my-doc", :_rev "1-2310851567"}
###document-update
    user=> (let [doc (document-get "some-db" "my-doc")]
             (document-update "some-db" "my-doc" (assoc doc :bam true)))
    {:foo 42, :bam true, :_id "my-doc", :_rev "2-1148768398"}
###document-revisions
    user=> (document-revisions "some-db" "my-doc")
    {"2-1148768398" "available", "1-2310851567" "available"}
###document-delete
    user=> (document-delete "some-db" "my-doc")
    true

##Attachment Functions

###attachment-list
    user=> (attachment-list "some-db" "my-doc")
    {"my-attachment" {:length 28, :content_type "text/plain", :stub true}, "f" {:length 3, :content_type "cntnt/type", :stub true}
###attachment-create
    user=> (attachment-create "some-db" "my-doc" "new-attachment" "PAYLOAD" "text/plain")
    "new-attachment"
###attachment-get
    user=> (attachment-get "some-db" "my-doc" "new-attachment")
    {:body-seq ("PAYLOAD"), :content-type "text/plain"}
###attachment-delete
    user=> (attachment-delete "some-db" "my-doc" "new-attachment")
    true

##View Functions

For now, views should be created/edited outside of this API.

In the following examples consider a simple view:

   `"foobars": {
       "map": "function(doc) {  emit(doc.foobar, doc); }"
   }`

###view-get
    user=> ; create some docs
    user=> (document-create "some-db" "doc1" {:foobar 42})
    {:_rev "1-4227851621", :_id "doc1", :foobar 42}
    user=> (document-create "some-db" "doc2" {:foobar 23})
    {:_rev "1-1185099016", :_id "doc2", :foobar 23}
    user=> ; run views
	user=> (use clojure.contrib.pprint) ; for pprint
    user=> (pprint (view-get "some-db" "my-design-doc" "foobars"))
    {:rows
     [{:value {:foobar 23, :_rev "1-1185099016", :_id "doc2"},
       :key 23,
       :id "doc2"}
      {:value {:foobar 42, :_rev "1-4227851621", :_id "doc1"},
       :key 42,
       :id "doc1"}],
     :offset 0,
     :total_rows 2}
    nil
    user=> (pprint (view-get "some-db" "my-design-doc" "foobars" {:descending true}))
    {:rows
     [{:value {:foobar 42, :_rev "1-4227851621", :_id "doc1"},
       :key 42,
       :id "doc1"}
      {:value {:foobar 23, :_rev "1-1185099016", :_id "doc2"},
       :key 23,
       :id "doc2"}],
     :offset 0,
     :total_rows 2}
    nil
    user=> (pprint (view-get "some-db" "my-design-doc" "foobars" {:startkey 40}))
    {:rows
     [{:value {:foobar 42, :_rev "1-4227851621", :_id "doc1"},
       :key 42,
       :id "doc1"}],
     :offset 1,
     :total_rows 2}
    nil

Nested clojure terms are also allowed in view options (eg. as keys) -- they will be converted to json and then to proper url-encoded GET args.

