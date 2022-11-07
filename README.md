# LDES-ideas

Exploration of existing LDES implementations and ideas for the thesis.

## LDES in LDP

#### Specification

[Writing Linked Data Event Streams in LDP Basic Containers](https://woutslabbinck.github.io/LDESinLDP/index.html)

- 13 October 2022

#### Existing Implementation

[VersionAwareLDESinLDP](https://github.com/woutslabbinck/VersionAwareLDESinLDP)

- client to organize LDES in SOLID pod with each `tree:Node` as `ldp:container`
  - one-dimensional fragmentation using `tree:GreaterThanOrEqualToRelation`
  - `tree:node` for appending new members described by `ldp:inbox` under `ldes:EventStream`
  - append operation - if size of latest node > 100 members, create new node, update `ldp:inbox`
  - primitive member search based on provided time interval in implementation

#### Thesis Ideas

Develop client library to store `tree:Collection`s in LDP (based on [TREE Specification](https://treecg.github.io/specification/#Relation)). Each node must be dereferencable as HTTP resource, `tree:view` can be organized as some data structure -- the specification [mentions B+tree](<(https://woutslabbinck.github.io/LDESinLDP/index.html#b+-tree)>). Maybe K-D tree for more dimensions?

## LDES and TREE Client Libraries

#### Existing Implementation

[TREEcg/event-stream-client](https://github.com/TREEcg/event-stream-client)

- client library to 'subscribe' to LDES and receive notification if new member is added
- datetime pruning with `tree:LessThanRelation`
- features:
  - user simply provides new member callback to process new member
  - dereferencing member option (because the collection pages do not contain all information)
  - serialize / deserialize client state (as LDES spec states client needs to maintain state)
  - checking for updates by polling, maintain priority queue with time to live
    - page will not be re-added if the page-cache is set to immutable
- downsides
  - `tree:view` can't be pruned based on other parameters and `tree:Relation`, only common LDES use-case
  - GETs only accept `application/ld+json`

```
for (const [_, relation] of treeMetadata.metadata.treeMetadata.relations) {
                // Prune when the value of the relation is a datetime and less than what we need
                // To be enhanced when more TREE filtering capabilities are available
                if (this.fromTime && relation["@type"][0] === "https://w3id.org/tree#LessThanRelation"
                    && moment(relation.value[0]["@value"]).isValid()
                    && new Date(relation.value[0]["@value"]).getTime() <= this.fromTime.getTime()) {
                    // Prune - do nothing
                } else {
                    // Add node to book keeper with ttl 0 (as soon as possible)
                    for (const node of relation.node) {
                        // do not add when synchronization is disabled and node has already been processed
                        if (!this.disableSynchronization || (this.disableSynchronization && !this.processedURIs.has(node['@id']))) {
                            this.bookkeeper.addFragment(node['@id'], 0);
                        }
                    }
                }
            }
```

#### Thesis Ideas

Develop library implementing parametrized search of `tree:view` by using `tree:Relation` as hypermedia cursors for pruning of the search tree. Checking previous implementations, each project just implements search that it needs / supports. Library core could only implement search functionality applying search on each `tree:Node` so it can be easily reused by existing projects missing complex search -- e.g `TREEcg/client-stream-client` can be extended in code snippet above to do prunning over other collections based on specific parameters, using other `tree:Relation`s.

Use case: Publisher wants to make collection available using TREE specification. For searches of specific collection members, `tree:Collection` index is built (it can be only used as an index structure, each `tree:member` can be dereferenced).

Use cases:

- Spatial data (`tree:GeospatiallyContainsRelation`) (but [specs](https://en.wikipedia.org/wiki/DE-9IM) looks really complex)
- N-dim data (each dimension can be described by `tree:Relation`)
  - parametrized search over published data of W3C Data Cube Vocabulary?
  - LDES and other one-dim `tree:Collection`s
  - DCAT datasets with TREE

Output of search would be a collection of `tree:member` objects. This search functionality could be implemented as standalone module that can be extended by implementation specific requirements (LDES), getting notifications, etc... (features of TREEcg/client-stream-client).

##### Analysis

Is this difficult enough? To implement the core search functionality, for each `tree:Node` it needs to check all `tree:Relation` and by comparing with specified parameters decide if we want to follow ` tree:Relation`'s `tree:node` or not. (+ maybe do validation that provided parameters exist in`tree:member`or`tree:Relation`).

It can implement other more complex features but there is already `TREEcg/client-stream-client`, missing only parametrization and prunning.

## LDES in SOLID Calendar Application

For managing of notifications in SOLID calendar, [VersionAwareLDESinLDP](https://github.com/woutslabbinck/VersionAwareLDESinLDP) can be used? TODO: more detailed analysis how exactly this will be implemented.
