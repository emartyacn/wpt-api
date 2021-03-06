# Mozilla wpt-api Documentation
(a.k.a **Using WebPageTest... _Mozilla Edition_**)

## Introduction
## Project Goals:
* Accurately measure, track, and report on key Firefox-performance metrics, over time
* Leverage, and be an integral part of, the [WebPageTest](https://www.webpagetest.org/) ecosystem

## Metrics & Metadata    

**Currently tracked metrics:**  

| metric | a.k.a. | units | type | source |  
| :--: | :-------: | :---: | :--: | :--: |  
| ```bytesInDoc``` | total bytes | num | count | Mult. sources[0] |  
| ```domContentFlushed``` | dcf | ms | duration | ```timeToDOMContentFlushed - fetchStart``` in <br>[wptagent/internal/js/page_data.js](https://github.com/WPO-Foundation/wptagent/pull/230/files) |    
| [```fetchStart```](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceTiming/fetchStart) | same | ms, sec | duration | gecko |
| ```pageLoadTime``` | ```pageload``` in [Raptor](https://searchfox.org/mozilla-central/rev/6c784c93cfbd5119ed07773a170b59fbce1377ea/testing/raptor/webext/raptor/measure.js#287-308) | ms, sec | duration | ```loadEventStart - fetchStart``` in <br> [cloudops-deployment/projects/wpt/puppet/modules/wpt/templates/settings/custom_metrics/pageLoadTime.js.epp](https://github.com/mozilla-services/cloudops-deployment/blob/5de5b9c90353b186e5ed1f0cb1b8f5e2296a988b/projects/wpt/puppet/modules/wpt/templates/settings/custom_metrics/pageLoadTime.js.epp) |  gecko |
| ```timeToDOMContentFlushed``` | TTDCF | ms, ms | duration | gecko |
| ```timeToFirstByte``` | TTFB | ms, sec | duration | Mult. sources[0] |
| ```timeToContentfulPaint``` | FCP | ms, sec | duration |(1)```user_pref("dom.performance.time_to_contentful_paint.enabled", true)``` in <br> [wptagent/internal/support/Firefox/profile.prefs.js](https://github.com/WPO-Foundation/wptagent/pull/214/files#diff-463e288bcde710e9a9ef8a46d490aac1R58) <br> and <br> (2)```addTime("timeToContentfulPaint");``` in <br> [wptagent/internal/js/page_data.js](https://github.com/WPO-Foundation/wptagent/pull/214/files#diff-69b0882d86377063fd0514c0dc978308R22)| gecko |
| ```timeToFirstNonBlankPaint``` | FNBP | ms, sec | duration |```user_pref("dom.performance.time.to_non_blank_paint", true)``` in <br> [wptagent/internal/support/Firefox/profile/prefs.js](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/Firefox/profile/prefs.js#L60) | gecko |  
| ```timeToFirstInteractive``` | TTI | ms, sec | duration | gecko |  
| [```visualComplete```](https://www.webpagetest.org/forums/showthread.php?tid=12846) | visually complete | ms, sec | duration | WebPageTest (video) |  
| [ ```SpeedIndex``` ](https://sites.google.com/a/webpagetest.org/docs/using-webpagetest/metrics/speed-index) | Speed Index | ... | score | WebPageTest (video) |   
| ```requestsFull``` |  total requests | num | count | Mult. sources[0]|  

[0] Multiple sources: (1) gecko (2) wptagent (3) another I'm forgetting?  The key takeaway is that because WPT can't get a trustable set of network events from Firefox, it does log-culling/HAR-generating "magic" (less magic is good!)  See https://github.com/WPO-Foundation/webpagetest/issues/1199

**Test Metadata:**  

| name | type | value(s) | source |
| :--: | :--: | :------: | :-----: |
| ```browser_version``` | _string_  |  "66.0a1" | ```self.marionette.session.capabilities``` in [wptagent/internal/firefox.py](https://github.com/WPO-Foundation/wptagent/blob/84018f548a2dea78dfca0ca64c19386adc6e2bca/internal/firefox.py#L128-L129) |  
| ```browser_name``` | _string_ | "Firefox Nightly" | ... |
### Manually Testing Metrics in Firefox
1. Enable/modify any (missing) prefs/pref-overrides
2. Open Tools -> Web Developer -> Web Console
3. Load a URL
4. After the page has "finished" loading, type something like: ```window.performance.timing.timeToFirstInteractive``` (**DO** take note of the other implemented/available metric options available as you autocomplete.)
5. Hit return/enter
6. You should see a value similar to ```1542438605479``` (time-stamped offset, in milliseconds)

PRO-TIPs: you can and *should* input ```window.performance.timing``` and/or ```performance.getEntriesByType("navigation")``` into the console, for the full data.

```window.performance.timing```:

![twitch-performance-timing](https://user-images.githubusercontent.com/387249/51446908-1a804c80-1ccd-11e9-9c33-52cadca49c30.png)


```performance.getEntriesByType("navigation")```:
![nav-timing](https://user-images.githubusercontent.com/387249/51446966-bf9b2500-1ccd-11e9-817d-b681728489d2.png)

### Adding Metrics to WebPageTest with Firefox
1. Manually test the metric + prefs; most metrics can be found in the following Firefox DOM WebIDL:  [```mozilla-central/dom/webidl/PerformanceTiming.webidl```](https://hg.mozilla.org/mozilla-central/file/tip/dom/webidl/PerformanceTiming.webidl)
2. If needed, add/modify Firefox's `prefs.js`, via a PR to [```wptagent/internal/support/Firefox/profile/prefs.js```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/Firefox/profile/prefs.js)
 * Example: https://github.com/WPO-Foundation/wptagent/pull/181/files#diff-69b0882d86377063fd0514c0dc978308
3. Additionally, we might need to have the metric (if not available via standard APIs) emitted in WebPageTest, in [```wptagent/internal/js/page_data.js```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/js/page_data.js#L27).
 * Example: https://github.com/WPO-Foundation/wptagent/pull/230

## Mini WebPageTest Compendium
* Batch/bulk-test (Python API/lib)
* Firefox WebExtension
* HAR files
  * ```send``` and ```wait``` events/timings are lumped into a shared ```wait``` metric
    *  https://github.com/WPO-Foundation/webpagetest/blob/0da83ac3f7e7407c96feaff46af1cfa65c461d6a/www/har/HttpArchiveGenerator.php#L379-L410
* logging
* log-parsing/recreation
  * devtools_parser.py
  * firefox_log_parser.py
* mobile-device testing (Android)
  * public instance:
    ```$ webpagetest test https://faraday.basschouten.com/mozilla/executionorder/externaltimeout0.html --server http://webpagetest.org -k [redacted_API_key] --location "Dulles_MotoG4:Moto G4 - Firefox" --connectivity 3GFast --runs 3 --first```
    
You'll see something like the following JSON returned:

```
{
  "statusCode": 200,
  "statusText": "Ok",
  "data": {
    "testId": "190123_FA_dcbc2acb71e08a640f81999f9d2360dd",
    "ownerKey": "5f84816b2aaf6c795cb4dad85e2a472e004f58c2",
    "jsonUrl": "http://webpagetest.org/jsonResult.php?test=190123_FA_dcbc2acb71e08a640f81999f9d2360dd",
    "xmlUrl": "http://webpagetest.org/xmlResult/190123_FA_dcbc2acb71e08a640f81999f9d2360dd/",
    "userUrl": "http://webpagetest.org/result/190123_FA_dcbc2acb71e08a640f81999f9d2360dd/",
    "summaryCSV": "http://webpagetest.org/result/190123_FA_dcbc2acb71e08a640f81999f9d2360dd/page_data.csv",
    "detailCSV": "http://webpagetest.org/result/190123_FA_dcbc2acb71e08a640f81999f9d2360dd/requests.csv"
  }
```

To view the full test results in the Web UI, just use that ```userURL```, which is http://webpagetest.org/result/190123_FA_dcbc2acb71e08a640f81999f9d2360dd/, in this case.
* MOZ_LOG
* networking
  * DNS
* filmstrips
* screenshots
* optimization checks
* timeouts
  *  ```run_time_limit``` (180, sec) which is __"Time limit for all steps in a single test run"__ in https://github.com/WPO-Foundation/webpagetest/blob/7b8d5d0821ae18b547f475133cae28a3c2b2778a/www/settings/settings.ini.sample#L60
  *  ```time``` (???, ???) which __"Set[s] the timeout on a per-test basis (not documented because I was a bit worried about abuse but it's there)."__ found in  https://www.webpagetest.org/forums/showthread.php?tid=3653&pid=25308#pid25308
  *  ```--timeout``` (120, sec) which is __"<seconds>: timeout for polling and waiting results [no timeout]"__ in https://github.com/marcelduran/webpagetest-api#test-works-for-test-command-only
  *  ```timeout``` and ```time_limit```** in https://github.com/WPO-Foundation/wptagent/blob/11222c7ab48bafb1203494dc4089fa298e75e040/internal/webpagetest.py#L429-L430
  *  ```maxtime```(600, sec) which is __"Maximum amount of time for a test run (if requested by timeout=X)"__  in https://github.com/mozilla-services/cloudops-deployment/blob/73ecc43a1c3a3da7c73a4d3d939b16e70cacf112/projects/wpt/puppet/modules/wpt/templates/settings/settings.ini.epp#L23-L24
  *  ```max_run_minutes``` (60, min) which is __"Force individual runs to end if they didn't complete."__ in https://github.com/WPO-Foundation/webpagetest/blob/7b8d5d0821ae18b547f475133cae28a3c2b2778a/www/settings/settings.ini.sample#L63
Also see https://github.com/WPO-Foundation/webpagetest/commit/ae11833a986260cf83f66b10fff4a9648f8dfa23, which added it
  *  ```step_timeout``` (120, sec) which is __"Default timeout for each step of a test (in seconds)"__ in https://github.com/WPO-Foundation/webpagetest/blob/7b8d5d0821ae18b547f475133cae28a3c2b2778a/www/settings/settings.ini.sample#L54
* video
* webPageReplay

### webpagetest
* [```install/index.php```](https://github.com/WPO-Foundation/webpagetest/blob/53590782310e26654fd068bd1431667305b6443d/www/install/index.php) (Post-install-check page)
###  wptagent
* [```Dockerfile```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/Dockerfile)
* [```desktop_browser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/desktop_browser.py) (base class for _all_ browsers)
* [```browsers.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/browsers.py) (self-described "Main entry point for controlling browsers")
* [```webpagetest.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/webpagetest.py) (WebPageTest core)
* [```firefox.py```](https://github.com/WPO-Foundation/wptagent/blob/master/internal/firefox.py) (hey, that's us!)
* [```traffic-shaping.py```](https://github.com/WPO-Foundation/wptagent/blob/master/internal/traffic_shaping.py) (cross-platform rate-limiting, latency, etc. support)
* [```page_data.js```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/js/page_data.js) (user timings, vendor-specific/internal metrics & configs/settings)
* [```firefox_log_parser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/firefox_log_parser.py) (where the metric-culling magic happens)
* [```visual_metrics.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/visualmetrics.py) (the guts of the visual-comparison algorithms)
* [```trace_parser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/trace_parser.py) (calculates metrics with "recipes"/formulas)
* [```pcap_parser.py```](https://github.com/WPO-Foundation/wptagent/blob/3f2128a9815838f462187b870be3c666ebd13d95/internal/support/pcap-parser.py) (parses PCAP files)

# Infrastructure
### WebPageTest Instance (*Mozilla-internal*)
* https://wpt.stage.mozaws.net - Auth0/OAuth
* https://wpt-api.stage.mozaws.net - Basic Auth
   * (You'll need to file a bug in Cloud Ops, requesting access from either :jbuck or :jrgm

To run a test via the API, using our **internal instance** at [wpt-api.stage.mozaws.net](https://wpt-api.stage.mozaws.net/), is something like:  
```
$ webpagetest test www.twitch.tv  --server https://username:passkey@wpt-api.stage.mozaws.net --location us-east-1-linux:Firefox --bodies --keepua -r 3 --first --reporter json
```

To run the same test, but using the **public instance** now at [www.webpagetest.org](https://www.webpagetest.org), would be:

```
$ webpagetest test www.twitch.tv -k your_API_key --location ec2-us-east-1:Firefox --bodies --keepua -r 3 --first --reporter json
```

### Jenkins
* [```https://qa-preprod-master.fxtest.jenkins.stage.mozaws.net/job/wpt/```](https://qa-preprod-master.fxtest.jenkins.stage.mozaws.net/job/wpt/)
   * To obtain access, see these (internal-only) [Mana docs](https://mana.mozilla.org/wiki/display/TestEngineering/qa-preprod-master.fxtest.jenkins.stage.mozaws.net)

### AWS EC2 Setup
* **1** ```c5.xlarge``` __WebPageTest__ core server-instance (Linux, EC2 AMI ID: ami-024df0ababa7118ae)
* **6** x ```c5.xlarge``` __wptagent__ server instances (Linux, EC2 AMI ID: ami-a88c20d5)

## Dependencies
#### Core:
* [WebPageTest](https://github.com/WPO-Foundation/webpagetest/blob/master/README.md) (core/server)
* [wptagent](https://github.com/WPO-Foundation/wptagent/blob/master/README.md) (configs/runs browser tests, and collects and submits metrics/data to be post-processed by the above core WebPageTest server; a rewrite of the project formerly known as "wptdriver."
* [webpagetest-api](https://github.com/marcelduran/webpagetest-api/blob/master/README.md) (NodeJS-wrapped core API)

## W3C (Draft) Specs

* [Perf-Timing Primer](https://w3c.github.io/perf-timing-primer/)
* [Navigation Timing](https://www.w3.org/TR/navigation-timing/)
* [Navigation Timing Level 2](https://www.w3.org/TR/navigation-timing-2/) (Working Draft)
* [High-Resolution Time Level 2](https://www.w3.org/TR/hr-time-2/)
* [Performance Timeline Level 2](https://www.w3.org/TR/performance-timeline-2/)
* [User Timing](https://www.w3.org/TR/user-timing/)

## Assorted Links:
* [Example Mobile-Device Lab Config](https://github.com/WPO-Foundation/webpagetest-docs/blob/2db35b31fe1c992c02650a5b401f7ed208d8fa27/user/Private%20Instances/mobile/example.md)
* [Web-platform tests we run](https://github.com/web-platform-tests/wpt/tree/744325921ba52791bc8db1b45d2aed097577753a/navigation-timing)
***

## Contributions
***psst, this could be you!***

## Credits & Thanks
* Pat Meenan - too numerous to call out; thanks for the project, the community, the tooling, and your direct support
* Rick Viscomi, Andy Davies, & Marcel Duran - thanks for /the/ published book, and the numerous blog posts, Tweets, talks, forum posts, etc., to further its adoption/ecosystem
* Peter Hedenskog - for everything (he knows!)
* [Sitespeed.io crew](https://www.sitespeed.io/) - for helping me re-envision and prototype an idea from 2010!
* Wikimedia Performance Team for their [amazing + open WebPageTest setup](https://wikitech.wikimedia.org/wiki/Performance)
* Firefox Performance Team for their support & shared focus
