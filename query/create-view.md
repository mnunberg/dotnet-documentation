
## Creating a View ##
Views can be created two ways:

1. Programmatically using the Bucket Management API or by
2. The Management Console

The Bucket Management API makes it possible to script or automate the creation of views and is useful for deployment scenarios, while the Management Console approach is best suited for development and learning purposes.

### Using the Bucket Management API###
The following code snippet uses the BucketManager class to insert a Design Document  called `dd_docs` containing a View called `all_docs` into the `default` bucket on Couchbase Server installed on localhost:

    using (var cluster = new Cluster())
    {
        using (var bucket = cluster.OpenBucket())
        {
            var manager = bucket.CreateManager("Administrator", "");
            var designDoc = "{\"views\":{\"all_docs\":{\"map\":\"function (doc, meta) { emit(meta.id, doc); }\"}}}";
            var result = await manager.InsertDesignDocumentAsync("dd_docs", designDoc);
            Console.Writeline(result.Success);
        }
    }

In the example above, first a Cluster object is created with no configuration which means `localhost` will be used for the bootstrap URI. Then a CouchbaseBucket object is opened using `OpenBucket()`, since we left the name empty, the `default` bucket will be used. Finally, a `BucketManager` object is created and `InsertDesignDocument()` method is called passing the name of the Design Document and the Design document itself (containing the View definition) into the method.

###Using the Management Console###

Using a browser, open the Management Console and log-in:

![](https://raw.githubusercontent.com/couchbaselabs/dotnet-documentation/master/images/login-managment-console.JPG)

Navigate to the _Views_ tab and select it, then select the correct bucket from the list and click _Create Developmental View_.

![](https://raw.githubusercontent.com/couchbaselabs/dotnet-documentation/master/images/create-development-view.JPG)

Click save and then the _Edit_ button to open the View editor. By default a View will be created for you which emits the `meta.id` (as the key) and `null` (as the value) for each row.

![](https://raw.githubusercontent.com/couchbaselabs/dotnet-documentation/master/images/management-console-edit-view.JPG)

You can edit this map function to your use-case. Optionally you can add a reduce function as well. Notice that this design doc as a `dev_` prefix; this indicates that the view has not been published yet and will only work on a subset of data within the cluster. In order to use the entire subset, you'll need to go back to the design doc editor and click _Publish_.
