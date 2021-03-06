##Geospatial View Queries##
Geospatial Views are a new feature in Couchbase Server 4.0. With them it is possible to perform queries using two-dimensional geometries stored with fields in your documents. Like normal Views, Geospatial views can be queried either directly via the REST API or via an SDK (as is shown here). The Couchbase Server.NET SDK 2.2.0 provides a wrapper API around the REST API making it easy to compose and execute queries.

###Querying the "points" Spatial View###
In the beer-sample Sample data bucket, you can create a Map function that emits a `Point` type from the "Brewery" documents which contains the latitude and longitude field called `coordinates`:

    function (doc) {
	    if (doc.type == "brewery" && doc.geo.lon && doc.geo.lat) {
	        emit({ "type": "Point", "coordinates": [doc.geo.lon, doc.geo.lat]}, null);
    	}
    }

To query this view, similar to a regular View, we will use the `SpatialViewQuery` object which exposes the REST API's parameters as a fluent interface. Here is an example of using this class to construct a query which we execute against the bucket:

    using (var cluster = new Cluster(ClientConfigUtil.GetConfiguration()))
    {
        using (var bucket = cluster.OpenBucket("beer-sample"))
        {
            var query = new SpatialViewQuery().From("beer_ext_spatial", "points")
                 .Stale(StaleState.False)
                 .StartRange(-10.37109375, 33.578014746143985)
                 .EndRange(43.76953125, 71.9653876991313)
                 .ConnectionTimeout(60000)
                 .Limit(1)
                 .Skip(0);

            var result = bucket.Query<dynamic>(query);
            foreach (var viewRow in result.Rows)
            {
                Console.WriteLine(viewRow.Value);
            }
        }
    }

Here we are providing a start range between `-10.37109375` and `33.578014746143985` and an end range between `43.76953125` and `71.96533876991313` which will return all breweries for an area over Europe. 