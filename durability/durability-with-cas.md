#Using Check and Set (CAS)#
CAS is acronym for "Check and Set" and is useful for ensuring that a mutation of a document by one user or thread does not override another near simultaneous mutation by another user or thread. The CAS value is returned by the server with the result when you perform a read on a document using Get or when you perform a mutation on a document using Insert, Upsert, Replace or Remove. 

##Example: Enforcing optimistic concurrency with CAS##
In certain situations you may want to ensure that you are not overwriting another users changes when you perform a mutation operation on a document. For this you have two choices: optimistic concurrency or pessimistic concurrency (locking). Optimistic concurrency tends to provide better performance than pessimistic concurrency, because no actual locks are help on the data.The pattern for optimistic concurrency is simple:

1. Perform a Get on a document to retrieve it
2. Edit the document as desired
3. Perform a mutation on the document and pass the CAS as an argument
4. If the mutation fails and the key exists, get the latest revision and apply mutation and then submit it again
5. Repeat until success

Here is an example:

    static async Task<bool> UpdatePostWithCasAsync(Post modified)
    {
        var bucket = ClusterHelper.GetBucket("default");
        const int maxAttempts = 10;
        var attempts = 0;

        do
        {
            //get the original document - if it doesn't exist fail
            var result = await bucket.GetAsync<Post>(modified.PostId);
            if (result.Status == ResponseStatus.KeyNotFound)
            {
                return false;
            }

            //update the original documents fields
            var original = result.Value;
            original.Content = modified.Content;
            original.Author = modified.Author;
            original.Views = original.Views++;

            //perform the mutation passing in the CAS value
            result = await bucket.UpsertAsync(modified.PostId, modified, result.Cas);
            if (result.Success)
            {
                return true;
            }

            //failed so try again up until maxAttempts
        } while (attempts++ < maxAttempts);

        //we failed within maxAttemps
        return false;
    }

If the document doesn't exist, the GetAsync call will return false. If it exists, then it's fields will be updated with this latest values passed in the method call. Then an UpsertAsync will be attempted and if it succeeds, true will be returned. If not the attempts counter will be incremented and the entire sequence will be tried again. Not that logic here can vary quite a bit depending upon your use-case, but the pattern itself will pretty much be the same: get the latest, make changes, try to update, if it fails try again until successful or until some arbitrary limit is reached. 
