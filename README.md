## onyx-datomic

Onyx plugin providing read and write facilities for batch processing a Datomic database.

#### Installation

In your project file:

```clojure
[com.mdrogalis/onyx-datomic "0.5.2"]
```

In your peer boot-up namespace:

```clojure
(:require [onyx.plugin.datomic])
```

#### Catalog entries

##### partition-datoms
```clojure
{:onyx/name :partition-datoms
 :onyx/ident :datomic/partition-datoms
 :onyx/type :input
 :onyx/medium :datomic
 :onyx/consumption :sequential
 :onyx/bootstrap? true
 :datomic/uri db-uri
 :datomic/t t
 :datomic/partition :com.my.example/partition
 :datomic/datoms-per-segment n
 :onyx/batch-size batch-size
 :onyx/doc "Creates ranges over an :eavt index to parellelize loading datoms"}
```

##### read-datoms

```clojure
{:onyx/name :read-datoms
 :onyx/ident :datomic/read-datoms
 :onyx/fn :onyx.plugin.datomic/read-datoms
 :onyx/type :function
 :onyx/consumption :concurrent
 :datomic/uri db-uri
 :datomic/partition my.datomic.partition
 :datomic/t t
 :onyx/batch-size batch-size
 :onyx/doc "Reads and enqueues a range of the :eavt datom index"}
```

##### commit-tx

The first variant expects to be fed in a stream of new entity maps and will automatically assign tempid's for the partition given.

```clojure
{:onyx/name :out
 :onyx/ident :datomic/commit-tx
 :onyx/type :output
 :onyx/medium :datomic
 :onyx/consumption :concurrent
 :datomic/uri db-uri
 :datomic/partition my.datomic.partition
 :onyx/batch-size batch-size
 :onyx/doc "Transacts segments to storage"}
```

The `:onyx/medium :datomic-tx` variant expects a tx, almost as if it was ready for `(d/transact uri tx)`. This lets you perform retractions and arbitrary db functions. You will need to respond with a [{:tx (.array (fressian/write [...]))}] though. This is to prevent your tx data from getting munged in the transport between peers on Onyx. 

```clojure
{:onyx/name :out
 :onyx/ident :datomic/commit-tx
 :onyx/type :output
 :onyx/medium :datomic-tx
 :onyx/consumption :concurrent
 :datomic/uri db-uri
 :datomic/partition my.datomic.partition
 :onyx/batch-size batch-size
 :onyx/doc "Transacts segments to storage"}
```

A function of the following form should be used to transform your data to be ready for throwing at the datomic commit-tx:

```
(require '[clojure.data.fressian :as fressian])
(require '[datomic.api :as d])

(defn datomic-txfn [e]
  [{:tx (.array (fressian/write [[:db/add (d/tempid :db.part/user) :db/doc "Hello world"]]))}])
```

#### Attributes

|key                           | type      | description
|------------------------------|-----------|------------
|`:datomic/uri`                | `string`  | The URI of the datomic database to connect to
|`:datomic/t`                  | `integer` | The t-value of the database to read from
|`:datomic/partition`          | `keyword` | The partition of the database to read out of
|`:datomic/datoms-per-segment` | `integer` | The number of datoms to compress into a single segment

#### Contributing

Pull requests into the master branch are welcomed.

#### License

Copyright © 2014 Michael Drogalis

Distributed under the Eclipse Public License, the same as Clojure.
