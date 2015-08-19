{
  title: "PHP",
  description: "How to run Selenium tests on Sauce Labs using PHP",
  category: 'Tutorials',
  index: 2,
  image: '/images/tutorials/php.png'
}

Sauce Labs is a cloud platform for executing automated and manual mobile and
web tests. Sauce Labs supports running automated tests with Selenium WebDriver
(for web applications) and Appium (for native and mobile web applications).

In this topic we'll show you how to run a test with Selenium WebDriver for PHP
on Sauce Labs.

## Prerequisites

You will need to have these components installed and set up before you can use
PHP with Sauce.

  * PHP
  * PHP curl library (for Windows only)
  * curl/OpenSSL support for PHP (for Windows only)
  * Sausage, a PHP framework developed for working with the Sauce Labs API

### Installing PHP

These instructions are for setting up PHP on Windows. For information about
setting up PHP on other operating systems, see [Installation and Configuration
of PHP](http://php.net/manual/en/install.php) on the PHP documentation
website.

  1. Download the latest thread-safe zip archive of the PHP binaries from [PHP Windows Downloads](http://windows.php.net/download/).
  2. Extract the content of the downloaded archive to _C:\PHP_.
  3. From the command prompt, navigate to _C:\PHP_ and prepare the php.ini file using these commands.

```cd C:\PHP```

```copy php.ini-development php.ini```

### Enabling the PHP curl Library

Edit the ```C:\PHP\php.ini``` file and uncomment each of these lines by removing the
semi-colon (;) that precedes them:
````
extension_dir = "ext"
extension=php_curl.dll
extension=php_openssl.dll
````
### Installing curl/OpenSSL Support for PHP

1. Right-click [mk-ca-bundle.vbs](https://raw.github.com/bagder/curl/master/lib/mk-ca-bundle.vbs) and select **Save link as...** , and then save the ```mk-ca-bundle.vbs``` file to your **C:\** directory.
2. From the command prompt, run the ```mk-ca-bundle.vbs``` file using this command:

```C:\.mk-ca-bundle.vbs```

1. In the [PHP] section of the ```PHP.ini``` file, add this line of code:

```curl.cainfo = c:\ca-bundle.crt```

1. Save and close the ```PHP.ini``` file.

### Installing Sausage

Sausage is a PHP framework that you can use with the Sauce Labs REST API. It is a set of classes and libraries that make it easy for you to run Selenium tests, either locally or on Sauce Labs. Sausage offers many additional features for free, like automatic pass/fail reporting. While we don't recommend any specific 3rd party library, we've used Sausage for our samples as an easy reference.

Sausage comes bundled with Paratest (for running your tests in parallel) and
optionally Sauce Connect (for testing locally-hosted sites with Sauce).

These instructions are for setting up Sausage on Windows operating system. For
more information about setting up Sausage on other operating systems, see
[Sausage Setup](https://github.com/jlipps/sausage).

1. Right-click [givememysausage.php](https://raw.githubusercontent.com/jlipps/sausage-bun/master/givememysausage.php), select **Save link as...** and save the ```givememysausage.php``` file to your project directory.
  2. Navigate to your project directory in the command prompt and run the ```givememysausage.php``` file using this command:

```php givememysausage.php -t YOUR_USERNAME YOUR_ACCESS_KEY```

This will download Sausage and all its dependences (PHPUnit for instance). The
Sausage set up might take a few minutes. The set up checks for a number of
requirements and if any are not met, notification messages are displayed on
your screen. Fix the issues, if any, and then run installation command again.

##Quick Start

Now that you have PHP and Sausage in place, let's try running a simple test to
make sure that everything works.

If you're on a Windows system, run this command from your project directory:

```vendor\bin\phpunit.bat WebDriverDemo.php```

If you're on a Mac/Linux system, run this command from your project directory:

```vendor/bin/phpunit WebDriverDemo.php```

This starts the PHPUnit test runner. You might not see any output right away,
but eventually you will see a series of dots inching across the. Each of these
dots represents a test that successfully passed. Tests with errors or failed
tests are represented by printing an E or an F respectively.

You should be able to see each test as it queues, runs, and finishes on your
[Sauce test page](https://saucelabs.com/tests). You will notice that each test
has a name. This information is sent automatically by Sausage at the beginning
of each test. Sausage also automatically notifies Sauce of the status of your
tests after they are complete.

## Code Example

Now let's take a look at some simple PHP code. In this example, we define a
set of browsers to use, and run a simple check to make sure that clicking the
link gets us to the expected new page. This simple example contains everything
needed to run an automated test on Sauce Labs.

Integrating Sauce is simple. We add a set of desired capabilities, and point
to a different Selenium server on the Sauce cloud. We will need to augment the
server address with your Sauce username and access key to ensure secure access
to the tests you run.
````php
<?php

require_once 'vendor/autoload.php';

define('SAUCE_HOST', SAUCE_USERNAME.':'.SAUCE_ACCESS_KEY.'@ondemand.saucelabs.com');

class WebTest extends PHPUnit_Extensions_Selenium2TestCase
{
 protected $start_url = 'http://saucelabs.com/test/guinea-pig';
  
 public static $browsers = array(
        array(
            'browserName' => 'firefox',
            'host' => SAUCE_HOST,
            'port' => 80,
            'desiredCapabilities' => array(
                'version' => '15',
                'platform' => 'Windows 2012'
            )
        )
    );
 
    protected function setUp()
    {
        $this->setBrowserUrl('');  
    }

    public function testTitle()
    {
        $this->url($this->start_url);
        $this->assertContains("I am a page title", $this->title());
    }
}
?>
````

Note that in this example, we assume these values are stored in PHP constants
(```SAUCE_USERNAME``` and ```SAUCE_ACCESS_KEY`` respectively), defined wherever you like.

So you can see, getting started on Sauce with your existing tests is really
quite simple

## Running Tests Against Local Applications

Developing apps on a localhost is quick and efficient. However, the drawback
is that localhost is not a publicly-accessible address on the Internet, so by
default the browsers in the Sauce Labs cloud cannot load and test an app that
you are running locally.

Sauce Connect uses a secure tunnel protocol that gives specific Sauce machines
access to your local network. Sauce Connect sessions are sandboxed from
outside data flows and are a convenient way to securely test apps that aren't
ready to be deployed on the Internet.

Sauce Connect is bundled as a separate composer package (sauce/connect). This
can be added to your _composer.json_ requirements. Edit the _composer.json_
file in your project directory by putting a comma after the sauce/sausage line
and adding the sauce/connect line so that it looks like this:
````
"require": {
    "sauce/sausage": ">=0.8.1",
    "sauce/connect": ">=3.0"
````

### Downloading Sauce Connect

Run this command to download Sauce Connect:

```php composer.phar update```

Note: Sauce Connect is a fairly large binary file, so it may take a little
while to download.

### Installing Sauce Connect

Run the following command to install Sauce Connect on the respective operating
systems:

/*for installing Sauce Connect on Windows*/

```vendor\bin\sauce_connect.bat```

/*for installing Sauce Connect on Mac/Linux*/

```vendor/bin/sauce_connect``

Since you have already configured the Sauce credentials in previous steps,
Sauce Connect starts up automatically. It can take some time to load because
it uses a new clean virtual machine to handle the secure connection.

When you see "connected! You may start your tests…" you are ready to go. Your
tests can now access localhost or a firewalled application.

When using Sauce Connect, all tests you run from your account use the same
network as the machine where Sauce Connect is located.

For more information about Sauce Connect, or to download and configure it on
your own, see [Sauce Connect
documentation](https://docs.saucelabs.com/reference/sauce-connect/).

## Running Tests in Parallel

Now that you're running tests on Sauce, you may wonder how you can run your
tests faster. One way to increase speed is by running tests in parallel across
multiple virtual machines.

Note: Tests can be run in parallel at two levels, and you can run your tests
in parallel across multiple browsers. For example, if you have 10 tests and
want to run on five browsers this would be parallelism of five. You can also
run tests across browsers and each test in parallel. Using our previous
example this would be more like 50 parallel tests. Doing this requires that
your tests are written in a way that they do not collide with one another. For
more on this see the [Selenium WebDriver - Running Your Tests in
Parallel](https://saucelabs.com/selenium/selenium-webdriver) blog.

### Parallel Tests in PHPUnit

Since PHPUnit doesn't have built-in parallel test execution, we can work
around this by using Paratest, a command line tool for running PHPUnit that is
bundled with Sausage. Let's try running some tests in parallel using Paratest.

Navigate to the project directory and run the following command:

/*For Mac/Linux operating system*/

```vendor/bin/paratest -p 2 -f --phpunit=vendor/bin/phpunit WebDriverDemo.php```

/*For Windows operating system*/

```vendor\bin\paratest.bat -p 2 -f --phpunit=vendor\bin\phpunit.bat
WebDriverDemo.php```

This command specifies the path to the test file we want to run and tells
Paratest that we want to simultaneously run two instances of PHPUnit.

Your tests should run approximately twice as fast as before. You can see the
test running simultaneously on your Sauce tests page. Depending on your Sauce
account level, consider increasing the number of processes to speed things up
even more.

By default tests running in parallel may be executed in any order. In the next
section we'll talk about this and discuss some tips for avoiding potential
snags while speeding up test suites using parallelism.

## Best Practices

###Automatic Test Naming

By default, Sauce Labs doesn't know how to display the name of your test.
Sausage comes up with a good name (TestClass::testFunction) and reports it
with your test so it's easy to find on your tests page.

###Automatic Test Status Reporting

By default there is e no way for Sauce Labs to know whether a particular test
was passed or failed. Sausage catches any failed assertions and reports the
status of the test to Sauce after it's complete. As you're looking at your log
of tests you can easily see which passed and which failed.

###Automatic Authorized Link Generation

Upon test failure, Sausage generates an authorized link to the failed job
report on the Sauce Labs website, to facilitate reporting to people who need
to know the details of the test. The job remains private (unless you change
the status yourself), but others can follow the link without needing to log in
with your credentials.

###Using Build IDs

If you're running your tests as part of your build, you can define a build ID,
either by updating the browser arrays to include a 'build' parameter, or (more
reasonably), defining an environment variable SAUCE_BUILD, like so:

`SAUCE_BUILD=build-1234 vendor/bin/phpunit MyAwesomeTestCase.php`

### Using SpinAsserts

SpinAsserts are really useful as well. Luckily, Sausage comes with a
SpinAssert framework built in. Let's say we want to perform a check and we're
not exactly sure how quickly the state will change to what we want. We can do
this:
````
public function testSubmitComments()
{
    $comment = "This is a very insightful comment.";
    $this->byId('comments')->click();
    $this->keys($comment);
    $this->byId('submit')->submit();
    $driver = $this;

    $comment_test = function() use ($comment, $driver) {
        return ($driver->byId('your_comments')->text() == "Your comments: $comment");
    };

    $this->spinAssert("Comment never showed up!", $comment_test);
}
````
This will submit a comment and wait for up to 10 seconds for the comment to
show up before declaring the test failed.

The spinWait function is similar, and allows you to wait for a certain
condition without necessarily asserting anything.



