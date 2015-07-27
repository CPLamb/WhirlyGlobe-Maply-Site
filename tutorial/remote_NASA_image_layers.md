---
title: Remote NASA Image Layers
layout: tutorial
---


### 3-2-1 Launching NASA EarthData Global Imagery

NASA provides a wealth of image layers for free ''Steve adds the intro''.  These layers are available through their [GIBS (Global Imagery Browse Services) website](https://earthdata.nasa.gov/about/science-system-description/eosdis-components/global-imagery-browse-services-gibs).

You'll need to have done the [Remote Image Layer](remote_image_layer.html) tutorial.  Open your HelloEarth project and get ready.

![Xcode ViewController.m](https://github.com/CPLamb/WhirlyGlobe-Maply-Site/blob/gh-pages/images/tutorial/NASA_GIBS_header.png)

If you haven't got one here is a suitable [ViewController.m]({{ site.baseurl }}/tutorial/code/ViewController_vector_selection.m) file to start with.  This version has a remote image layer already configured and it makes a nice starting point.

Let's start by explaining whatever????

### NASA GIBS base layer tile sources

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
