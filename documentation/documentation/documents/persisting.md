<!--Title:Persisting Documents and Unit Of Work-->
<!--Url:persisting-->

At this point, the `IDocumentSession` is the sole [unit of work](http://martinfowler.com/eaaCatalog/unitOfWork.html) for transactional updates -- but that may change later as Marten outgrows its origin as a replacement for RavenDb. As <[linkto:documentation/documents;title=shown before]>, document sessions come in three flavors (lightweight, identity map tracking, and identity map + dirty checking), but there are only two modes of change tracking:

1. Lightweight and the standard "identity map" sessions require users to do all the change tracking manually and tell the `IDocumentSession` 
   what documents have changed
1. The "dirty checking" session tries to determine which documents loaded from that `IDocumentSession` has any changes when `IDocumentSession.SaveChanges()` is called


## Manual Change Tracking

The first step is to create a new `DocumentSession` with the `IDocumentStore.LightweightSession()` (or `IDocumentStore.OpenSession()`):

<[sample:lightweight_document_session_uow]>

Do note that Marten's .Net API makes no distinctions between inserts and updates. The Postgresql functions generated by Marten to update the document storage tables perform "upserts" for you. Anytime a document is registered through `IDocumentSession.Store(document)`, Marten runs the "auto-assignment" policy for the id type of that document. See <[linkto:documentation/documents/document_ids]> for more information on document id's.


## Automatic Dirty Tracking Session

In the case an `IDocumentSession` opened with the dirty checking enabled, the session will try to detect changes to any of the documents loaded by that
session. The dirty checking is done by keeping the original JSON fetched from Postgresql and using Newtonsoft.Json to do a node by node comparison of the 
JSON representation of the document at the time that `IDocumentSession` is called. 

<[sample:tracking_document_session_uow]>

Do be aware that the automated dirty checking comes with some mechanical cost in memory and runtime performance. 


## SaveChanges() Optimization

The call to `IDocumentSession.SaveChanges()` tries to batch all the queued updates and deletes into a single ADO.Net call to Postgresql. Our testing has
shown that this technique is much faster than issuing one ADO.Net call at a time.


## Bulk Inserts

See also <[linkto:documentation/documents/bulk_insert]> for an alternative for inserting a large number of one kind of document at a time. Also see the section on <[linkto:documentation/documents/identitymap]>.