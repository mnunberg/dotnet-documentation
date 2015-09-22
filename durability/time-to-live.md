##Enforcing Document Lifetimes with TTL##
In some situations you may want to limit the lifetime of a document so that is automatically evicted after some set duration. TTL or "time to live" is supported by both Memcached and Couchbase buckets and allows you to control the lifespan of a document. 

For example, perhaps you have a scenario where the most recent documents are read quite often and older docs, less often. To keep your memory footprint small, you want to expire older documents if there not accessed after some arbitrary time-span. In this case you could use TTL to provide a sliding expiration where if the document is accessed within a set amount of time, the expiry will be updated. If it's not read within the duration, the document will be evicted from the cache.

###Setting document lifetimes###
Here is an example setting an expiry on a document for 2 days:

    ClusterHelper.Initialize(new ClientConfiguration
    {
        Servers = new List<Uri>
        {
            new Uri("http://localhost:8091/")
        }
    });

	//get the bucket
	var bucket = ClusterHelper.GetBucket("default");

	//set the timespan to 2 days
    var result = await bucket.UpsertAsync("foo", "bar", new TimeSpan(2, 0, 0, 0));
    if (result.Success)
    {
        //upsert was successful
    }

    ClusterHelper.Close();

You can also do this using a Document object:

    var result = await bucket.UpsertAsync(new Document<string>
    {
        Id = "foo",
        Content = "bar",
        Expiry = (uint) new TimeSpan(2, 0, 0, 0).TotalMilliseconds
    });

    if (result.Success)
    {
        //upsert was successful
    }

In either case, the result is the same; after 2 days the document will be evicted from memory and from disk if the operation is done on a persistent Couchbase bucket. Note that the Document API uses a uint representing the milliseconds that is the lifetime of the document.

###Using Touch and Touch and Get###
You may have a use-case where you want the expiry to be updated when the document is read or you may want to "touch" a document to update the TTL values, there are two means of doing this: Touch and TouchAndGet. Here is an example using TouchAndGet:

    var bucket = ClusterHelper.GetBucket("default");
    var result = bucket.Upsert("foo", "bar", new TimeSpan(0, 0, 0, 3));

    if (result.Success)
    {
        Thread.Sleep(1000);

        result = bucket.GetAndTouch<string>("foo", new TimeSpan(0, 0, 0, 3));
        Console.WriteLine(result.Value);
    }

Touch works in a similar manner, the only difference is that the value is not returned so it's a slightly quicker operation.




