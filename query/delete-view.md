##Deleting a View##

Similar to how Views are created, they can be deleted two ways:

1. Programmatically using the Bucket Management API or by
2. The Management Console

The Bucket Management API makes it possible to script or automate the deletion of views and is useful for deployment scenarios. The management console is a graphical interface and is simpler to use for one-off deletions.

### Using the Bucket Management API###
Using the `BucketManager` class, we can delete a View by either updating an existing Design Document and removing the View or by deleting the entire Design Document. Note that deleting the Design Document will remove all views associated with that Design Document.

    using (var bucket = cluster.OpenBucket())
    {
        var manager = bucket.CreateManager("Administrator", "");
        var designDoc = "{\"views\":{\"all_docs\":{\"map\":\"function (doc, meta) { emit(meta.id, doc); }\"}}}";
        var insertResult = await manager.InsertDesignDocumentAsync("dd_docs", designDoc).Result;

        if (insertResult.Success)
        {
            var designDocModified = "{\"views\": {}}";
            var updateResult = await manager.UpdateDesignDocumentAsync("dd_docs", designDocModified).Result;
            Console.WriteLine(updateResult.Success);
        }
    }

In the example above, first a Cluster object is created with no configuration which means `localhost` will be used for the bootstrap URI. Then a `CouchbaseBucket` object is opened using `OpenBucket()`, since we left the name empty, the `default` bucket will be used. Then a `BucketManager` object is created and `InsertDesignDocument()` method is called passing the name of the Design Document and the Design document itself (containing the View definition) into the method. Finally, the Design Document is updated by removing the View `all_docs` and then `UpdateDesignDocumentAsync()` is called with the Design Document name and updated Design Document.

<!---
I'm guessing it's more common for users to just want to nuke the entire ddoc rather than removing a single view from it
-->