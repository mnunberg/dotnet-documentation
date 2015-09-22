##Using Prepared Statements##
When a N1QL statement is executed, the SDK sends the statement to the server as a string in the request body. The server will inspect the string and make a query "plan". This plan contains the names of indexes it will need to query. Generating the query plan creates additional computation overhead to the query request. To eliminate this overhead, statements can be prepared and the pre-generated plan reused.

To ensure that your queries reuse the prepared, the `AdHoc` flag should set to `false` on the `QueryRequest` object and the Couchbase .NET SDK will then handle the caching and use of the prepared statement after the initial query is sent to the server. 

    using (var cluster = new Cluster(ClientConfigUtil.GetConfiguration()))
    {
        using (var bucket = cluster.OpenBucket("travel-sample"))
        {
            var queryRequest = new QueryRequest()
                .Statement("SELECT * FROM `travel-sample` LIMIT $1")
                .AddPositionalParameter(10)
                .AdHoc(false);

            var result = bucket.Query<dynamic>(queryRequest);
            foreach (var row in result.Rows)
            {
                Console.WriteLine(row);
            }
        }
    }

From a user perspective the only thing that needs to be done to take advantage of prepared statements is set the `AdHoc` property to `false`. Note that by default, the `AdHoc` property is set to `true`: There is a one-time overhead at the client and network when a query is prepared, therefore only queries that are truly repeated (and are thus not ad-hoc) should modify the `AdHoc` flag to `false`.


