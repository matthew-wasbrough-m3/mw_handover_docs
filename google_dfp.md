# Migration of AdTech to Google DFP (now known as google admanager)  <!-- omit in toc --> 

This migration was carried out in two parts. First was code changes in May 2018 followed by deployment in late August 2018. 

## Contents <!-- omit in toc --> 
- [Apps Affected](#apps-affected)
    - [Github](#github)
    - [Gitlab](#gitlab)
    - [External](#external)
    - [Service](#service)
- [Implementation](#implementation)
  - [Older Sites](#older-sites)
  - [React Sites](#react-sites)
- [Google AdManager Script](#google-admanager-script)
  - [Define Ad Position](#define-ad-position)
  - [Targeting](#targeting)
  - [Size Mapping](#size-mapping)
  - [Extra Settings](#extra-settings)

## Apps Affected

The front end apps affected as part of this migration were:

#### Github
* [m3.intl.dnuk-home-controller](https://github.com/m3europe/m3.intl.dnuk-homepage-controller/)
* [m3.intl.offduty-controller](https://github.com/m3europe/m3.intl.offduty-controller/)
* [m3.intl.megadodo-controller](https://github.com/m3europe/m3.intl.megadodo-controller/)
* [m3.intl.webmail](https://github.com/m3europe/m3.intl.webmail/)
* [m3.intl.forum.web.ie](https://github.com/m3europe/m3.intl.forum.web.ie/)

#### Gitlab
* [Dnuk.Education](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.education)
* [Dnuk.Library](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.library)
* [Dnuk.ECMEQuizHome](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.ECMEQuizHome)
* [Dnuk.LearningRecord](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.LearningRecord)
* [Dnuk.UpdateYourProfile](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.UpdateYourProfile)
* [Dnuk.Wordpress](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.Wordpress)
* [Dnuk.JournalNews](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.JournalNews)

#### External
* Volcanic

#### Service
Also modified was the .NET Advertising Service API:
* [Dnuk.API.Advertising](http://gitlab.internal.doctors.net.uk/Dnuk/Dnuk.API.Advertising)

([Top](#contents))

## Implementation

### Older Sites

The older parts of the site (all the gitlab apps, webmail, and forum.web.ie) all make a preliminary request to Dnuk.API.Advertising. This is an api service that translates the ad id from the page into a placement that google knows about. The following code shows an example from the forum.web.ie project:

```html
<div id="4516038" class="leaderBoard pure-g pure-u-1-1">
  <script language="javascript" type="text/javascript" src="{{jsapiPath}}/jsapi/latest/advertising/advertising.svc/gd/m3.forum/4516038"></script>
</div>
```

In this case, because the snippet is from a template, it is possible to replace the `{{jsapiPath}}` with `https://www.doctors.net.uk` or `https://fftest.doctors.net.uk` as required. The id at the end of the script src url must match the id of the containing div as this indicates to the ad service where to put the ad on the page. The gd is to denote the Google DFP endpoint as the original endpoints were left in place. The `m3.forum` is the syndication key that is linked to the referrer url. 

The script returned from `Dnuk.API.Advertising` is partially dependant on the logged in user. 

### React Sites

The react sites implement this script directly in the code. This allows a single request to be made to Google AdManager for all ads on the page instead of single requests for each ad placement on the page. To see the implementation on the m3.intl.dnuk-homepage-controller project it is between lines 219 and 294 [here](https://github.com/m3europe/m3.intl.dnuk-homepage-controller/blob/21fc1827f3d6a520e9bd796602f01408d6952971/src/app/containers/home-container/index.jsx#L219).

([Top](#contents))

## Google AdManager Script

Below is the script returned by the call above for older sites. For the react sites the script is very similar but is setup to request multiple ads instead of just one.

```JavaScript
// define the size mapping for later use
var sizeMapping = googletag.sizeMapping()
  .addSize([1410, 120], [[970, 90], [728, 90]])
  .addSize([992, 120], [728, 90])
  .addSize([0, 0], [320, 50])
  .build();

// define the function to be passed to google
googletag.cmd.push(function() {

  // define an individual ad position
  googletag
    .defineSlot(
      '/21705339479/DNUK/forum-topleaderboard',
      [[320, 50], [728, 90], [970, 90]],
      '4516038'
    )
    .setTargeting('spec', [148])
    .setTargeting('sen', [10])
    .setTargeting('mg', [12373,24667,24950])
    .setTargeting('type', [2])
    .defineSizeMapping(sizeMapping)
    .addService(googletag.pubads()
  );

  // add extra general parameters
  googletag.pubads().collapseEmptyDivs();

  // enable the google services
  googletag.enableServices();

  // actually request the ad
  googletag.display('4516038');
});
```

### Define Ad Position

The ad position is defined as a function in the googletag.cmd array. The function can take as many ad units as you need. The homepage passes all four ad units at once. The script from the .Net ad service returns a single ad position per script as it allows individual ad units to be called only when required.

The ad position definition can be as simple as `googletag.defineSlot()`. The parameters this function takes define how google understand the ad position relative to its system and where on the page to put the requested ad unit. In this example the first parameter `/21705339479/DNUK/forum-topleaderboard` is made up of three parts that define the ad position in AdManager, the `21705339479` is the account number. In AdManager for ease of working with an integration environment and a live environment ad units are defined in a hierarchy. In this example the `DNUK` is for the live site, the alternative would be `DNUK-Test`. The final part is the ad unit name as defined in ad manager in this case `forum-topleaderboard`, note there are two ad units with this name, one defined under `DNUK` and one under `DNUK-Test`. The second parameter is the size of the ad units that can be accepted in this placement. Each placement is defined as a two part array of `[width, height]`, when multiple placement sizes can be accepted, as in this case then you have an array of these placement arrays. The final parameter is the `id` of the div on the page where the ad unit should be placed. In this case it is `4516038` which is a throw back to the AdTech implementation. You can see the div id being the same [above](#older-sites)

### Targeting

The main reason for calling to the .Net ad service for both the older and react sites is to get the targeting information. The request to the .Net ad service for the react apps returns just the targeting information so the react site can implement the targeting as it needs. The older sites have the targeting built into the script. The targeting is defined using `.setTargeting()` which takes two parameters. The first parameter is the string name of the targeting key as defined in Google AdManager, the second parameter is an array of values that have been previously defined in Google AdManager, these values can be either strings or numbers. 

### Size Mapping

```JavaScript
var sizeMapping = googletag.sizeMapping()
  .addSize([1410, 120], [[970, 90], [728, 90]])
  .addSize([992, 120], [728, 90])
  .addSize([0, 0], [320, 50])
  .build();
```

The first part of the script sets up the size mappings for later use. Each size has two components, the first is the minimum size for the mapping to apply and the second is the size of the ads to show for the given size of display. The first `.addSize` takes the queries against a viewport of greater than 1410px wide and 120px high. On a page that meets these conditions an ad of size 970px x 90px or an ad of 728px x 90px will be displayed. If either the height or the width are smaller than the given values then the next line will be evaluated: if the view port has a width greater than 992px and a height greater that 120px then an ad of 728px x 90px will be displayed. If bother of these queries fail the fall back at width greater than 0px and height greater than 0px is to display an ad of 320px x 50px.

To add the size mapping to the ad unit request the variable defined above can be passed directly to the `.defineSizeMapping()` function.

### Extra Settings

```JavaScript
  // add extra general parameters
  googletag.pubads().collapseEmptyDivs();

  // enable the google services
  googletag.enableServices();

  // actually request the ad
  googletag.display('4516038');
```

There are extra settings that can be added to the googletag object. The options are fairly self explanatory. They are:
* `collapseEmptyDivs()` - makes sure that if no ad is served then the div is collapsed so it does not leave empty gaps on the page. 
* `enableServices()` - enables all google services for the ads units to work.
* `display()` - this is the main request for an ad to be served and the passed parameter is a string which is the id of the div the ad should be served into.

