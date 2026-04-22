# no-sql spec notes

I have some changes I'd like you to make:

1. **Custom indexes** - Intead of passing custom index defintions to the DataStore constructor I would prefer a method like `ensureIndexes()` or `configureIndexes()`, or maybe just `configure()`. The reason is that the application may need to dynamically add or remove indexes while the database is in use, without restarting the application. I forgot to mention that use case earlier.
2. **Abstract StorageEngine Interface create/update** - Instead separate create() and update() methods, use the same method and signature defined for put(doc, options) on the DataStore Public API.
3. **Abstract StorageEngine Interface QueryOptions** - Instead gte, lte, gt, lt, for queries, use greaterThanOrEqualTo, lessThanOrEqualTo, greaterThan, lessThan. Also add beginsWith and startKey/endKey (startKey and endKey should both be inclusive).
4. **DataStore Public API remove()** - Rename the remove() method to delete(). Delete is more common and intuitive. We don't want any surprises in our naming.
