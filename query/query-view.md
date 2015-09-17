##Querying a View##

After creating and published a View, the View is queried using the View REST API. Since most developers like to use an SDK, all of the Couchbase clients expose a API which wraps the underlying View REST API making it easy to build and execute queries and map the results of those queries to POCO or dynamic objects.

###The QueryRequest Class###
To build the View query, there is a special ViewQuery class which exposes a fluent-type interface which allows you to build a query request for a specific View on a given Design Document. The ViewQuery contains methods mapping to all of View REST API fields, for example Asc, Dsc, Skip and Limit are supported as well as many others like querying by key range.

    using (var cluster = new Cluster())
    {
         using (var bucket = cluster.OpenBucket())
         {
             var query = new ViewQuery()
                .From("dd_docs", "all_docs")
                .Stale(StaleState.Ok)
                .Limit(10)
                .Skip(1)
                .Asc();

            var results = bucket.Query<dynamic>(query);
            if (results.Success)
            {
                foreach (var result in results.Rows)
                {
                    Console.WriteLine(result.Value);
                }
            }
        }
    }


In the snippet above, a Cluster object is created and a CouchbaseBucket object is opened, then a ViewQuery is created that will query the "all_docs" View on the "dd_docs" Design Document. The StaleState is specified as StaleState.OK, which means the any new documents may not have been indexed yet. There are two other options for StaleState: False and UpdateAfter. False will force an index update before returning the data and UpdateAfter will run the indexer after the View has been accessed.
