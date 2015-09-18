##Using Prepared Statements##
When a N1QL statement is executed, the SDK sends the statement to the server as a string in the request body. The server will inspect the string and make a query plan of which indexes it will need to query. Once it has done this the server will generate a query plan which will require some additional computation which adds overhead to the query request. To eliminate this overhead, statements can be prepared and the pre-generated plan reused.

To ensure that your queries reuse the prepared, the adHoc flag is set to false on the QueryRequest object and the Couchbase .NET SDK will then handle the caching and use of the prepared statement after the initial query is sent to the server. 

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

From a user perspective the only thing that needs to be done to take advantage of prepared statements is set the AdHoc property to false. Note that by default, the AdHoc property is set to true. Also, note that there is a finite number query plans that can be cached by the SDK, so do not toggle the Adhoc property back and forth between on and off or this cached, limit may be exceeded causing cache eviction.


