##Durability best practices##
Despite our best attempts, network failures, node outages and general maintenance are a part of distributed computing that cannot be completely removed. That being said, Couchbase provides ways of improving the likelihood that an operation or request will succeed. In this section, techniques for handling durability and availability will be discussed along with application patterns and techniques for handling these situations.

###Handling failover###
When a Couchbase cluster node is failed over, the node is removed from the cluster and replica data in the other nodes is made available for client requests. Typically for maintenance you will remove a node; failing over a node is for when a node is no longer functional.

From an application perspective, your major tools for handling fail over scenarios are by specifying durability constraints such as the number nodes the data must be persisted to and/or replicated to on writes and by allowing replicas to be checked during a read for data if it cannot be retrieved from the primary node. 

For example, if you have a three node cluster, you may wish to set the number of replicas stored to be two and then on your writes providing the PersistTo and ReplicateTo values as two and two respectively. This will ensure that in order for the write to succeed, the given durability must be achieved by replicating the data to at least two nodes and that the data is persisted on at least two nodes as well. This concept is known as "observe". 

###Enhanced and CAS Durability###
In Couchbase server 4.0 there are two ways of providing durability constraints: the older CAS based approach and the newer "enhanced durability" which uses a sequence number to ensure replication and persistence. From a developer perspective, thankfully, the "how" is abstracted away; you simply set property BucketConfiguration.UseEnhancedDurability to true. The default is "false" which means that CAS durability will be used.

Here is an example that uses enhanced durability (note that UseEnhancedDurability is true):

    var config = new ClientConfiguration
    {
        Servers = new List<Uri> { new Uri(ConfigurationManager.AppSettings["bootstrapUrl"]) },
        BucketConfigs = new Dictionary<string, BucketConfiguration>
        {
            {
                "default", new BucketConfiguration
                {
                    UseEnhancedDurability = true
                }
            }
        }
    };
    using (var cluster = new Cluster(config))
    {
        using (var bucket = cluster.OpenBucket())
        {
            bucket.Remove(key);
            var result = await bucket.InsertAsync(key, "foo", ReplicateTo.Two, PersistTo.Two);
            Assert.IsTrue(result.Success);
            Assert.AreEqual(Durability.Satisfied, result.Durability);
        }
    }

Had the defaults been used here, CAS and observe would have been used to enforce durability; once again this is all internal plumbing and the "how" is opaque to the developer.

Note that there is a trade-off here; we are ensuring durability at the cost of performance,since the application must wait for these constraints to be met. Depending upon the use-case, however this may be acceptable.

###Reading from Replicas###
For ensuring data can be read in the event of a failover, the Couchbase .NET SDK provides a method for performing replica reads: CouchbaseBucket.GetFromReplica. This method works as a sort of "fallback" if the request for the data on the primary fails. In order to use this method, replicas must be enabled for the cluster.

    using (var cluster = new Cluster(ClientConfigUtil.GetConfiguration()))
    {
        using (var bucket = cluster.OpenBucket())
        {
            var result = await bucket.GetAsync<string>("foo");

			//check result from primary
            if (result.Status != ResponseStatus.Success && result.Status != ResponseStatus.KeyNotFound)
            {
				//check the replicas
                result = await bucket.GetFromReplicaAsync<string>("foo");
            }
			if (result.Success)
            {
                //do something with result
            }
        }
    }

