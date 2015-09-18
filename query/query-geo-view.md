##Geo Spatial View Queries##
Geo Spatial Views are a new feature in Couchbase Server 4.0. With them it is possible to perform queries using two-dimensional geometries stored with fields in your documents. To perform Geo Spatial queries, the View REST API is used or more likely an SDK. Similar to how regular Views are queried, the Couchbase Server.NET SDK 2.2.0 provides a wrapper API around the REST API making it easy to compose and execute queries. Note that to use Geo spatial queries, you documents must follow the GeoJSON specification.

###Querying the "points" Spatial View###
In the beer-sample Sample data bucket, you can create a Map function that emits a "Point" type document from the "Brewery" documents and contains the latitude and longitude field called "coordinates":

    function (doc) {
	    if (doc.type == "brewery" && doc.geo.lon && doc.geo.lat) {
	        emit({ "type": "Point", "coordinates": [doc.geo.lon, doc.geo.lat]}, null);
    	}
    }

To query this view, similar to a regular View, we will use query request object which exposes the REST API's parameters as a fluent interface called SpatialViewQuery. Here is an example of using this class to construct a query which we execute against the bucket:

    using (var cluster = new Cluster(ClientConfigUtil.GetConfiguration()))
    {
        using (var bucket = cluster.OpenBucket("beer-sample"))
        {
            var query = new SpatialViewQuery().From("beer", "points")
                 .Bucket("beer-sample")
                 .Stale(StaleState.False)
                 .BBox(-10.37109375, 33.578014746143985, 43.76953125, 71.9653876991313)
                 .ConnectionTimeout(60000)
                 .Limit(10)
                 .Skip(0);

            var result = bucket.Query<dynamic>(query);
            foreach (var viewRow in result.Rows)
            {
                Console.WriteLine(viewRow.Value);
            }
        }
    }

In this case, instead of specifying a range query, we are using a "bounded-box" which is an area defined by two latitudes. This example includes an area over Europe. 