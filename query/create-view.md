
## Creating a View ##
Views can be created two ways:

1. Programmatically using the Bucket Management API or by
2. The Management Console

The Bucket Management API makes it possible to script or automate the creation of views and is useful for deployment scenarios. While the Management Console approach is best suited for developing on your local machine or when you have physical access to the remote cluster.

### Using the Bucket Management API###
The following code snippet uses the BucketManager class to insert a Design Document  called "dd_docs" containing a View called "all_docs" into the "default" bucket on Couchbase Server installed on localhost:

    using (var cluster = new Cluster())
    {
        using (var bucket = cluster.OpenBucket())
        {
            var manager = bucket.CreateManager("Administrator", "");
            var designDoc = "{\"views\":{\"all_docs\":{\"map\":\"function (doc, meta) { emit(meta.id, doc); }\"}}}";
            var result = manager.InsertDesignDocument("dd_docs", designDoc);
            Assert.IsTrue(result.Success);
        }
    }

In the example above, first we create a Cluster object and pass in no configuration which means localhost will be used for the bootstrap URI. Then we create a CouchbaseBucket object using CreateBucket, since we left the name empty, the "default" bucket will be used. Finally, we create a BucketManager and call the InsertDesignDocument method passing the name of the Design Document and the Design document itself (containing the View definition) into the method.

###Using the Console Manager###

Using a browser, open the Management Console and log-in:

![](https://raw.githubusercontent.com/couchbaselabs/dotnet-documentation/master/images/login-managment-console.JPG)

Navigate to the "Views" tab and select it, then select the correct bucket from the list and click "Create Developmental View".

![](https://raw.githubusercontent.com/couchbaselabs/dotnet-documentation/master/images/create-development-view.JPG)

Click save and then the "Edit" button to open the View editor. By default a View will be created for you which emits the meta id as the key and null for each row. 

![](https://raw.githubusercontent.com/couchbaselabs/dotnet-documentation/master/images/management-console-edit-view.JPG)

You can edit this map function to your use-case. Optionally you can add a reduce function as well. Notice that this design doc as a "dev" prefix; this indicates that the view has not been published yet and will only work on a subset of data within the cluster. In order to use the entire subset, you'll need to go back to the design doc editor and click "Publish".
