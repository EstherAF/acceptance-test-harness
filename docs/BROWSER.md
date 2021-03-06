# Selecting Browser

This test harness uses WebDriver to execute tests.
The person who runs tests can select which browser to use by using the `BROWSER` environment variable.
The following values are available:

 * `firefox` (default)
 * `ie`
 * `chrome`
 * `safari`
 * `htmlunit`
 * `phantomjs`
 * `saucelabs`
 * `remote-webdriver-firefox`
        _(Requires `REMOTE_WEBDRIVER_URL` to be set to the url of the remote,
          for example `http://0.0.0.0:32779/wd/hub`
          when using something like
          [selenium/standalone-firefox-debug](https://hub.docker.com/r/selenium/standalone-firefox-debug/))_

For example, to run tests with Safari, you'd execute:

    export BROWSER=safari
    mvn install

    # or more concisely
    BROWSER=safari mvn install

See `FallbackConfig.java` for how the browser is selected.

Please note selenium library is sensitive to versions of browser used so it is better to stick with recent stable versions of mainstream web browsers. For more information about Selenium supported platforms visit [this page](http://www.seleniumhq.org/about/platforms.jsp).

## Advanced Browser Configuration
[This test harness internally uses Guice](GUICE.md) to wire components, and that is how we control
WebDriver. To further fine-tune how a browser is selected and configured, bind `WebDriver` to
a specific factory of your choice.

Usually you'd bind WebDriver to the test scope so that each test gets a new browser instance.
For example,

    bind WebDriver toProvider {
        def d = new FirefoxDriver();
        d.manage().addCookie(...);
        return d;
    } as Provider _in TestScope

See [WIRING.md](WIRING.md) for details of where to put this.

## Avoid focus steal with Xvnc on Linux
If you select a real GUI browser, such as Firefox,
a browser window will pop up left and right during tests,
making it practically unusable for you to use your computer.
There is a script to run VNC server and propagate the display number to the test suite using the dedicated variable `BROWSER_DISPLAY`.

    $ eval "$(./vnc.sh)"
    $ mvn test

## Example using remote web driver

Untested pseudo bash example

    docker run -d -P selenium/standalone-firefox-debug > containerId.txt
    export WEBDRIVER_CONTAINER_ID=$(cat containerId.txt)
    export BROWSER=remote-webdriver-firefox
    export REMOTE_WEBDRIVER_URL=http://$(docker port $WEBDRIVER_CONTAINER_ID 4444)/wd/hub
    export JENKINS_LOCAL_HOSTNAME=$(ip -4 addr show docker0 | grep -Po 'inet \K[\d.]+')
    mvn test

