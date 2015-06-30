

# Running tests #

## `ant` targets ##

```
ant runtests
```

> Runs all tests (including browser tests, but not including
> jQuery tests).

> Note: junit output will show failures for known-fail tests
> marked with `@FailureIsAnOption`.  Known-fails are accounted
> for after all tests are run, when generating the test
> summary.

> The test summary is at `ant-reports/*/report/index.html`

```
ant nobrtest
```

> Runs only non-browser tests.
> This is identical to `"ant runtests -Dtest.exclude=*BrowserTest"`

```
ant brtest
```

> Runs only browser tests (but not jQuery tests).
> This is identical to `"ant runtests -Dtest.filter=*BrowserTest"`

```
ant brserve
```

> Starts the local HTTP server used by browser tests, then
> waits forever, which is useful for manually running and
> debugging browser tests.

> `brserve` will print a URL `.../test-index.html` that is a
> page with links to all the individual browser tests.

```
ant brserve+
```

> Same as `brserve`, but also starts the browser used by
> browser tests.

## `runtests` flags ##

```
-Dtest.browser= (default: "firefox")
```

> Chooses which browser to test.
> `"firefox"` and `"chrome"` work for local browser testing.
> `"firefox"`, `"chrome"`, and `"ie"` work for remote browser testing.

```
-Dtest.browser.close= (default: "no")
```

> Normally when a browser test fails, the test runner will try to
> leave the browser open so that it can be examined later.
> When this flag is truthy, the test runner will always close the
> browser when done, whether it passes or fails.

```
-Dtest.capture.passes= (default: "no")
```

> When browser testing captures screenshots and logs, it normally captures just failed tests.
> If this flag is truthy, passed tests will also be captured.

```
-Dtest.capture.to= {default: "", but set by build.xml to a subdir of ant-reports)
```

> When non-empty, this should be the path to a directory,
> and browser testing will save screenshots and logs to that directory.

```
-Dtest.chrome.args= (default: "")
```

> Semicolon-separated list of command-line args to use when
> starting Chrome.

> This applies to both local and remote Chrome testing.

> A comprehensive list of args is available
> [here](http://peter.sh/experiments/chromium-command-line-switches/),
> but probably the only interesting one is `--js-flags`.

> In particular, `--js-flags=--harmony` has the same effect as
> the "Enable Experimental Javascript" option on the
> `chrome://flags` page. (Note, using `--js-flags=--harmony`
> will not change "Enable Experimental Javascript" on the flags
> page, but the effect is the same.)

```
-Dtest.chrome.binary= (default: system-dependent search)
```

> Overrides the automatic search for the Chrome executable when
> testing Chrome. This should be an absolute pathname.

> This applies to both local and remote Chrome testing. For
> remote testing, the value should be a pathname on the remote
> host.

> On MacOS X, this should point at the
> `"Contents/MacOS/Google Chrome"` executable inside the app bundle.

```
-Dtest.debug= (default: "no")
```

> When truthy, listen on port 9999 and wait for a Java debugger
> to connect before starting tests.

```
-Dtest.debug.browser= (default: "no")
```

> When truthy, start the test server and the browser, then
> wait. This is the same as `ant brserve+`.

```
-Dtest.debug.server= (default: "no")
```

> When truthy, start the test server, then wait. This is the
> same as `ant brserve`.

```
-Dtest.exclude= (default: "--none--")
```

> Skip Java test classes that match this glob pattern.

```
-Dtest.failureNotAnOption= (default: "no")
```

> When truthy, the test report generator will ignore the
> `@FailureIsAnOption` annotation on tests.

```
-Dtest.filter= (default: "*Test")
```

> Only run Java test classes that match this glob pattern.

```
-Dtest.filter.method= (default: "*")
```

> Only run Java test methods that match this glob pattern.

> Note, we don't always have a way to skip running a test
> method. Instead, non-matching methods may be recorded as
> passing without being run, which makes it a little tricky to
> tell the difference between tests that passed because they
> ran vs tests that passed because they were skipped.

```
-Dtest.server.hostname= (default: automatic guess)
```

> When running a remote webdriver, the remote browser needs to
> load resources from a local HTTP server started by the test
> driver. The test driver will guess a hostname that the remote
> browser can use to contact the local HTTP server, but the
> guess can be wrong.  Setting this will override the guessed
> hostname.

```
-Dtest.server.port= (default: "0" or "8000")
```

> Browser tests start a local HTTP server.  This setting lets
> you override the server's port.  The value `0` means to use
> any available port.

> When actually running tests, the default is `0`.
> But when you do `"ant brserve"` the default is `8000`,
> since a fixed port is more convenient for manual testing.

```
-Dtest.threads= (default: "2")
```

> `runtests` uses limited parallelism to finish faster.
> Use `"-Dtest.threads=1"` to disable parallel execution.

> Setting `test.threads` to more than 2 will probably not speed
> up tests much, because most of the time is spent on browser
> tests, which run sequentially.

```
-Dtest.webdriver.url= (default: "")
```

> When this is non-null, browser tests are run on a remote
> webdriver instead of a local webdriver.

> The value should be something like `http://remotehost:4444/`
> or `http://remotehost:4444/wd/hub/`

```
-Dtestonly.{X}=
```

> Any property that starts with `"testonly."` will be passed
> through to the test runner, but with the `"testonly."` prefix
> removed.  This is useful for properties that you want visible
> to tests but not visible to `ant`.

```
-Dwebdriver.{X}=
```

> Any property starting with `"webdriver."` will be passed
> through to the test runner unchanged.

```
-Dwebdriver.firefox.bin= (default: system-dependent search)
```

> Overrides the automatic search for the Firefox exectuable
> when testing Firefox.


# Local browser testing #

## Firefox ##

```
ant brtest
```

> Firefox is the default browser for testing.  The test driver
> tries to find `firefox` in standard system locations,
> including your PATH.

> You can override the location with `"-Dwebdriver.firefox.bin=..."`

## Chrome ##

```
ant brtest -Dtest.browser=chrome
```

> First, you need `chromedriver`, which is available
> [here](http://code.google.com/p/chromedriver/downloads/list).

> Use the `chromedriver2` builds. The older chromedriver is
> unable to talk to Chrome version 29 and later.

> If `chromedriver` isn't in your PATH, you can specify the
> location with `"-Dwebdriver.chrome.driver=..."`

> `chromedriver` looks for Chrome in standard system
> locations.  You can override that with
> `"-Dtest.chrome.binary=..."`

> On MacOS, `test.chrome.binary` should point at the
> `"Contents/MacOS/Google Chrome"` binary inside the app bundle.

> If you want more control over `chromedriver` settings, start
> it manually and treat it like a remote browser, described below.

## Safari ##

> Doesn't work yet. Remote testing of Safari does work, so just treat
> local Safari as a remote browser on localhost.

> Note, browser-expectations has a test that will fail if Safari blocks
> popups. You need to manually disable the popup blocker for that to pass.

# Remote browser testing #

> First you need to start a webdriver server on the remote
> host.

## webdriver server for Chrome, Firefox, and Safari ##

```
java -jar selenium-server-standalone-{VERSION}.jar
```

> Run that on the remote host, which can be Linux,
> MacOS, or Windows.  There's a copy of
> `selenium-server-standalone` in `third_party/java/webdriver`,
> but any recent version should be fine.

> For Chrome testing, the remote host will also
> need the `chromedriver` program, which is explained
> in the "Local browser testing" section above.

> If you only want to test Chrome, you can ignore
> `selenium-server-standalone` and just run `chromedriver`
> directly on the remote host to provide webdriver service.

## webdriver server for IE ##

> You need to run `IEDriverServer` from
> [here](http://code.google.com/p/selenium/downloads/list).
> [This page](http://code.google.com/p/selenium/wiki/InternetExplorerDriver)
> has more information on starting and configuring the IE
> webdriver server.

## running the tests ##

```
ant brtest -Dtest.browser=... -Dtest.webdriver.url=http://...
```

> `test.browser` can be `"chrome"`, `"firefox"` (default),
> `"ie"`, or `"safari"`.

> For remote Linux and MacOS, `test.webdriver.url` should be
> something like `http://remote:4444/wd/hub`

> For remote Windows, `test.webdriver.url` should be something
> like `http://remote:4444/`

> You can verify the remote webdriver works by fetching the URL
> `http://remote:4444/wd/hub/status` or
> `http://remote:4444/status`