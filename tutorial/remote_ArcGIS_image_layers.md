---
title: Remote ArcGIS Image Layers
layout: tutorial
---


### Remote ArcGIS Vector Layers

ESRI's ArcGIS platform has a huge number of datasets, as well as the ability to generate your own thru the ArcGIS application.  Here are a few examples of what we're going to build today. 

![Header pic](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/blob/gh-pages/images/tutorial/ArcGIS_Header.png)

You’ll need a sample project for this tutorial. Go back and start with the Hello Earth tutorial and work thru the [CartoDB Tutorial](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/blob/gh-pages/tutorial/remote_image_layer.md).  We’ll want the tiling logic from that tutorial.  If you'd rather just get started with those files, you can download them here;

[ViewController.m](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/tree/gh-pages/tutorial/code/ViewController_cartodb.m) Your main view controller.
[CartoDBLayer.h](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/tree/gh-pages/tutorial/code/CartoDBLayer.h) CartoDBLayer header.
[CartoDBLayer.m](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/tree/gh-pages/tutorial/code/CartoDBLayer.m) CartoDBLayer implementation.

### Hello Earth
OK to summarize, in this app we're are going to utilize remote datasets from ESRI's (that's the Environmental Systems Research Institute, I just finally learned) ArcGIS website.  ArcGIS is the premier GIS application out there, and it's used by everyone.  Check it out, join, whatever, but you'll have to do it on your own time.

In this app, we are going to load one of their great base maps a National Geographic globe found here.
And as a second act we are going to access one of their vector data sets showing New York City's flood zones found here.

As mentioned above, we're not going to get into much of the details of ArcGIS or how the Hello Earth vector tiling works, that's detailed elsewhere.  So, let's get setup.  I changed the CartoDB files to ArcGISLayer, and it's associated method to addVectors, just to be pedantic about it.  Run the app and you should get the CartoDB view of NYC's landlords or whatever.  If not, make it so.

![CartoDB pic](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/blob/gh-pages/images/tutorial/CartoDB_NYCBuildings.png)

### ArcGIS Base Map - National Geographic World Map
So first thing we're going to load up one of ArcGIS's base maps, the beautiful & revered [National Geographic World Map](http://services.arcgisonline.com/arcgis/rest/services/NatGeo_World_Map/MapServer).  All we have to do change the URL reference in the existing code & the new map should appear, as if by magic.  Find where that's done in viewDidLoad, and replace the URL string with this URL;

{% highlight bash %}
http://services.arcgisonline.com/arcgis/rest/services/NatGeo_World_Map/MapServer
{% endhighlight %}

{% highlight objc %}
// Portions Courtesy NASA/JPL­Caltech and U.S. Depart. of Agriculture, Farm Service Agency
MaplyRemoteTileSource *tileSource =  [[MaplyRemoteTileSource alloc]
initWithBaseURL:@"http://services.arcgisonline.com/arcgis/rest/services/                                 NatGeo_World_Map/MapServ/tile/{z}/{y}/{x}"
ext:@"png" minZoom:0 maxZoom:maxZoom];
{% endhighlight %}

you also need to adjust the maxZoom value (in this case - 17) and make sure the ext:file is png.  The /tile/{z}/{y}/{x} appended to the end of the URL is a specific requirement of ??Whatever??

Run the app, zoom way out, and you should get a great rendition of everyone's favorite Nat Geo globe.  Easy Peasey!

![NatGeo globe pic](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/blob/gh-pages/images/tutorial/NatGeoGlobe.png)

### Next up - Vector Layers
Vector layers are datasets that return polygons, attributes and other things.  The mechanics of how WhirlyGlobe handles this data is thoroughly detailed in the CartoDB tutorial, and you should review that there if you like.  Here today we are simply going to replace the CartoDB data with data from ArcGIS, specifically showing the various different flood zones in the NYC area.  We'll also make a few other changes to make the displayed data pop!  Here goes;

As discussed, we have created a CartoDBLayer object that conforms to the MaplyPagingDelegate.  This object then queries the remote data source for the data required by the tiles displayed with the method startFetchForTile:forLayer:.  This query is comprised of 2 portions, a URL for the remote server, and a SQL query that are joined together in the constructRequest method.  Let's start by changing the URL to -

{% highlight bash %}
http://services.arcgis.com/OfH668nDRN7tbJh0/ArcGIS/rest/services/NYCEvacZones2013/FeatureServer
{% endhighlight %}

in the CartoDBLayer constructRequest code.

{% highlight objc %}
- (NSURLRequest *)constructRequest:(MaplyBoundingBox)bbox {
// construct a query string
double toDeg = 180/M_PI;
NSString *query = [NSString stringWithFormat:search,bbox.ll.x*toDeg,bbox.ll.y*toDeg,bbox.ur.x*toDeg,bbox.ur.y*toDeg];
NSString *encodeQuery = [query stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
encodeQuery = [encodeQuery stringByReplacingOccurrencesOfString:@"&" withString:@"%26"];
NSString *fullUrl = [NSString stringWithFormat:@"https://pluto.cartodb.com/api/v2/sql?format=GeoJSON&q=%@",encodeQuery];.
NSURLRequest *urlReq = [NSURLRequest requestWithURL:[NSURL URLWithString:fullUrl]];

return urlReq;
}
{% endhighlight %}

And we'll also need to change the query in the ViewController's addBuildings method

{% highlight objc %}
- (void)addBuildings
{
NSString *search = @"WHERE=Zone>=1&f=pgeojson&outSR=4326";
// NSString *search = @"SELECT the_geom,address,ownername,numfloors FROM mn_mappluto_13v1 WHERE the_geom && ST_SetSRID(ST_MakeBox2D(ST_Point(%f, %f), ST_Point(%f, %f)), 4326) LIMIT 2000;";

CartoDBLayer *cartoLayer = [[CartoDBLayer alloc] initWithSearch:search];
cartoLayer.minZoom = 13;
cartoLayer.maxZoom = 15;
{% endhighlight %}

outSR is the ouptpu spacial reference, and pgeojson also defines the output format.
Run the project, and you should see the layers for the flood zones.  Not?  OK, lets adjust a few things to see what's going on;

Change the mni/maxZoom levels = 9 to 13
Modify the initial zoom level to 0.008
query for a single zone>=4
Also let's add in a NSLog statement to see if data is being returned

{% highlight objc %}
[NSURLConnection sendAsynchronousRequest:urlReq queue:opQueue completionHandler:^(NSURLResponse *response, NSData *data, NSError *connectionError)
{
NSLog(@"returned data length is %lu", (unsigned long)data.length);
{% endhighlight %}

Run the project again, and you should see some data displayed.  It's not exactly what we want, but at least we know we're receiving data, Yay!  Now lets clean up this bad boy.

Through the magic of files fiddling, we finally end up with -

![NYC flood zones pic](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/blob/gh-pages/images/tutorial/ArcGISProgression.png)








All we need to do is replace the existing MaplyRemoteTileSource URL with one supplied thru the GIBS website.  Here are two URLs that provide a base layer for your Whirly Globe.

{% highlight bash %}
http://map1.vis.earthdata.nasa.gov/wmts-webmerc/MODIS_Terra_CorrectedReflectance_TrueColor/default/2015-06-07/GoogleMapsCompatible_Level9/
{% endhighlight %}

{% highlight bash %}
http://map1.vis.earthdata.nasa.gov/wmts-webmerc/VIIRS_CityLights_2012/default/2015-07-01/GoogleMapsCompatible_Level8/
{% endhighlight %}

Open ViewController.m. Now let's find where to add the image layer URL.  Scroll down thru viewDidLoad until you find the following code

{% highlight objc %}
// MapQuest Open Aerial Tiles, Courtesy Of Mapquest
// Portions Courtesy NASA/JPL­Caltech and U.S. Depart. of Agriculture, Farm Service Agency
MaplyRemoteTileSource *tileSource =
[[MaplyRemoteTileSource alloc]
    initWithBaseURL:@"http://otile1.mqcdn.com/tiles/1.0.0/sat/"
    ext:@"png" minZoom:0 maxZoom:maxZoom];
{% endhighlight %}

Replace the initWithBaseURL property with one of the selections above.
Also change ext with "jpg"
And match the maxZoom with the level of the GoogleMapCompatible Level (8 or9)

Run the project, and you should get a stunning NASA earth globe (or map?)

### Adding an Overlay Layer

For our next feat let's add an overlay image to our globe.  Below are a couple of GIBS URLS for overlay images.

{% highlight bash %}
http://map1.vis.earthdata.nasa.gov/wmts-webmerc/Sea_Surface_Temp_Blended/default/2013-06-07/GoogleMapsCompatible_Level7/
{% endhighlight %}

{% highlight bash %}
http://map1.vis.earthdata.nasa.gov/wmts-webmerc/MODIS_Terra_Land_Surface_Temp_Day/default/2013-06-07/GoogleMapsCompatible_Level7/
{% endhighlight %}

{% highlight bash %}
http://map1.vis.earthdata.nasa.gov/wmts-webmerc/MODIS_Terra_Chlorophyll_A/default/2015-02-10/GoogleMapsCompatible_Level7/
{% endhighlight %}


Add this method

{% highlight objc %}
@interface ViewController : UIViewController 
                <WhirlyGlobeViewControllerDelegate,MaplyViewControllerDelegate>
{% endhighlight %}

Now just add the addOverlay method  (It's some derivation of this)

{% highlight objc %}
­// CPL is roughing out code
// Run through the overlays the user wants turned on
- (void)addOverlay:(NSDictionary *)baseSettings
{
// For network paging layers, where we'll store temp files
NSString *cacheDir = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)  objectAtIndex:0];
NSString *thisCacheDir = nil;

for (NSString *layerName in [baseSettings allKeys])
{
bool isOn = [baseSettings[layerName] boolValue];
MaplyViewControllerLayer *layer = ovlLayers[layerName];
// Need to create the layer
if (isOn && !layer)
{
if (![layerName compare:kMaplyTestOWM])
{
MaplyRemoteTileSource *tileSource = [[MaplyRemoteTileSource alloc] initWithBaseURL:@"http://map1.vis.earthdata.nasa.gov/wmts-webmerc/Sea_Surface_Temp_Blended/default/2013-06-07/GoogleMapsCompatible_Level7/"
ext:@"png" minZoom:1 maxZoom:7];

//             MaplyRemoteTileSource *tileSource = [[MaplyRemoteTileSource alloc] initWithBaseURL:@"http://tile.openweathermap.org/map/precipitation/" ext:@"png" minZoom:1 maxZoom:10];

tileSource.cacheDir = [NSString stringWithFormat:@"%@/openweathermap_precipitation/",cacheDir];
tileSource.tileInfo.cachedFileLifetime = 3 * 60 * 60; // invalidate OWM data after three hours
MaplyQuadImageTilesLayer *weatherLayer = [[MaplyQuadImageTilesLayer alloc] initWithCoordSystem:tileSource.coordSys tileSource:tileSource];
weatherLayer.coverPoles = false;
weatherLayer.flipY = true;         // chopped tiles fix??
layer = weatherLayer;
weatherLayer.handleEdges = false;
[baseViewC addLayer:weatherLayer];
ovlLayers[layerName] = layer;
} else if (!isOn && layer)
{
// Get rid of the layer
[baseViewC removeLayer:layer];
[ovlLayers removeObjectForKey:layerName];
}
}
{% endhighlight %}

Next, find this code in addOverlay;

{% highlight objc %}
MaplyRemoteTileSource *tileSource = [[MaplyRemoteTileSource alloc] initWithBaseURL:@"http://map1.vis.earthdata.nasa.gov/wmts-webmerc/Sea_Surface_Temp_Blended/default/2013-06-07/GoogleMapsCompatible_Level7/"
ext:@"png" minZoom:1 maxZoom:7];
{% endhighlight %}
 
Change the initWithBaseURL to any of the URLs listed above.
And don't forget to match the maxZoom level

just like in the first section replace the remoteTileSource URL with any one of the one listed above

Add don't forget to add this to the bottom of viewDidLoad

{% highlight bash %}
// adds the layer
[self addOverlay]
{% endhighlight %}


That's it! Build and run.  You should see some sweet NASA data! 

### Code Breakdown

That's all there is to it.  Here's a working [ViewController.m]({{ site.baseurl }}/tutorial/code/ViewController_post_vector_selection.m) if you need it.
