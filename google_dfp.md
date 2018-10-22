# Migration of AdTech to Google DFP (now known as google admanager)

This migration was carried out in two parts. First was code changes in May 2018 followed by deployment in late August 2018. 

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

## Google AdManager Script

Below is the script returned by the call above for older sites. For the react sites the script is very similar but is setup to request multiple ads instead of just one.

```JavaScript
var sizeMapping = googletag.sizeMapping()
  .addSize([1410, 120], [[970, 90], [728, 90]])
  .addSize([992, 120], [728, 90])
  .addSize([0, 0], [320, 50])
  .build();
googletag.cmd.push(function() {
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
    .addService(googletag.pubads());
  googletag.pubads().collapseEmptyDivs();
  googletag.enableServices();
  googletag.display('4516038');
});
```

### Size Mapping