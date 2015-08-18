 {
  title: "Python",
  description: "How to run Selenium tests on Sauce Labs using python",
  category: 'Tutorials',
  index: 3,
  image: "/images/tutorials/python.png"
}

## Getting Started with Python

Sauce Labs is a cloud platform for executing automated and manual mobile and web tests. Sauce Labs supports running automated tests with Selenium WebDriver (for web applications) and Appium (for native and mobile web applications).

In this tutorial we are going to show you how to run a test with Selenium WebDriver on Sauce Labs. 

## Table of Contents
1. [Dependencies](#dependencies)
2. [Code Example](#code-example)
3. [Running Tests on Sauce](#running-tests-on-sauce)
4. [Running Against Local Applications](#running-tests-against-local-applications)
5. [Reporting to the Sauce Labs Dashboard](#reporting-to-the-sauce-labs-dashboard)

## Dependencies
First, add the [Selenium WebDriver API](http://www.seleniumhq.org/download/) to your local Python environment using [pip](https://pypi.python.org/pypi/pip):

```
pip install selenium
```
	
## Code Example
Now let's look at a simple Python test script testing the Google front page. Despite the simplicity of this example test, it actually contains everything you need to know in order to run an automated test on Sauce Labs:

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.desired_capabilities import DesiredCapabilities

# This is the only code you need to edit in your existing scripts. 
# The command_executor tells the test to run on Sauce, while the desired_capabilties 
# parameter tells us which browsers and OS to spin up.
desired_cap = {
    'platform': "Mac OS X 10.9",
    'browserName': "chrome",
    'version': "31",
}
driver = webdriver.Remote(
   command_executor='http://sauceUsername:sauceAccessKey@ondemand.saucelabs.com:80/wd/hub',
   desired_capabilities=desired_cap)

# This is your test logic. You can add multiple tests here.
driver.implicitly_wait(10)
driver.get("http://www.google.com")
if not "Google" in driver.title:
    raise Exception("Unable to load google page!")
elem = driver.find_element_by_name("q")
elem.send_keys("Sauce Labs")
elem.submit()
print driver.title

# This is where you tell Sauce Labs to stop running tests on your behalf.  
# It's important so that you aren't billed after your test finishes.
driver.quit()
```
Copy this code and save it into a file called first_test.py. Make sure your username and access key are included in the URL passed through to the command_executor. Then open terminal, navigate to the directory where the file is located, and execute the test by typing:

```
python first_test.py
```

Check your [dashboard](http://www.saucelabs.com/dashboard) and you will see that your test has just run on Sauce!

__Note:__ The [implicitly wait](https://selenium-python.readthedocs.org/waits.html#implicit-waits) method tells the browser to wait a set amount of time (in seconds) for elements to appear on the page before giving up. Without it, slow loading DOMs could cause our tests to fail when they might otherwise pass.

Let's look at the test a little closer so you can write one like it, or set your existing tests to run on Sauce.


## Running Tests on Sauce
If you wanted to run Selenium locally, you might initiate a driver for the browser that you want to test on like so:
```python
driver = webdriver.Firefox()
```

However, to run on Sauce we use the general webdriver.Remote() instead of the specific webdriver.Firefox(), and then pass it two paramaters: command_executor and desired_capabilties.

(__Note:__ webdriver.Remote() is a standard Selenium interface, so you can do anything that you could do with a local Selenium test. The only code specific to Sauce Labs was the URL that makes the test run using a browser on the Sauce Labs servers.)


```python
# this is how you set up a test to run on Sauce Labs
desired_cap = {
    'platform': "Mac OS X 10.9",
    'browserName': "chrome",
    'version': "31"
}
driver = webdriver.Remote(
    command_executor='http://sauceUsername:sauceAccessKey@ondemand.saucelabs.com:80/wd/hub',
    desired_capabilities=desired_cap)
```

#### Command Executor
The command_executor code tells our script to use browsers in the Sauce Labs cloud instead of a local browser. Here we simply pass in a URL that contains your Sauce username and access key, which can be found on your [dashboard](http://www.saucelabs.com/dashboard).

#### Testing on Different Platforms
Because we are not specifying webdriver.Firefox() as we were before, we must use desired capabilities to specify what browser/OS combination(s) to spin up and execute against.

If you want to run against different platforms, simply update your desired capabiltlies to something new, like so:

```python
desired_cap = {'browserName': "chrome", 'platform': "Windows 8.1", 'version': "42.0"}
```

The desired capabilities are a set of keys and values that will be sent to the Selenium server running in the Sauce Labs cloud. These keys and values tell the Selenium server the specifications of the automated test that you will be running. Using our [Platforms Configurator](https://docs.saucelabs.com/reference/platforms-configurator) you can easily determine the correct desired capabilities for your test.

## Running Tests Against Local Applications
If your test application is not publicly available, you will need to use Sauce Connect so that Sauce can reach it. 

Sauce Connect is a tunneling app that allows you to execute tests securely when testing behind firewalls or on localhost. For more detailed information, please visit see the [Sauce Connect docs](https://docs.saucelabs.com/reference/sauce-connect/). 
 
## Reporting to the Sauce Labs Dashboard
####Recording Pass/Failure Results
"Wait," you might be asking, "My test says 'Complete' but what happens if it fails?"

Unfortunately, Sauce has no way to determine whether your test passed or failed automatically, since it is determined entirely by your business logic. We can, however, tell Sauce about the results of our tests automatically using the [Sauce python client](https://pypi.python.org/pypi/sauceclient):
```
pip install sauceclient
```
Then add this to your test:
```python
# this authenticates you 
from sauceclient import SauceClient
sauce_client = SauceClient("sauceUsername", "sauceAccessKey")

# this belongs in your test logic
sauce_client.jobs.update_job(driver.session_id, passed=True)
```
#### Setting a Build Number
Now, you may want to associate this with a build id in your Continuous Integration pipeline. To do this, just include a build number in your desired capabilities:

```python
desired_cap = {
    'platform': "Mac OS X 10.9",
    'browserName': "chrome",
    'version': "31",
    'build': "build-1234",
}
```

This is important for organizing tests in our new dashboard. Please see our docs for more information [here](https://docs.saucelabs.com/reference/test-configuration/#recording-build-numbers). 

#### Tagging Your Tests

You can also create custom tags for your tests that you can use to search on the [archives](https://saucelabs.com/beta/archives) page:

```python
desired_cap = {
    'platform': "Mac OS X 10.9",
    'browserName': "chrome",
    'version': "31",
    'build': "build-1234",
    'tags': [ "tag1", "tag2", "tag3" ]
}
```