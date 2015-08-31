{
  title: "Ruby",
  description: "How to run Selenium tests on Sauce Labs using Ruby",
  category: 'Tutorials',
  index: 1,
  image: "/images/tutorials/java.png"
}

## Getting Started with Ruby

Sauce Labs is a cloud platform for executing automated and manual mobile and web tests. Sauce Labs supports running automated tests with Selenium WebDriver (for web applications) and Appium (for native and mobile web applications).

In this tutorial we are going to show you how to run a test with Selenium WebDriver on Sauce Labs.

## Table of Contents

1. [Quickstart](#quickstart)
2. [Prerequisites](#prerequisites)
3. [Code Example](#code-example)
4. [Running Tests in the Sauce Cloud](#running-tests-in-the-sauce-cloud)
5. [Running Against Local Applications](#running-tests-against-local-applications)
6. [Running Tests in Parallel](#running-tests-in-parallel)
7. [Reporting on Test Results](#reporting-on-test-results)


## Quickstart

It's very easy to get your existing Ruby tests up and running on Sauce. 

If you wanted to run a test on Selenium locally, you would initiate a driver for the browser you want to test against like this:
```
driver = Selenium::WebDriver.for :firefox
```
To run an existing test on Sauce, all you need to change is your `driver` definition, and make sure that the `sauce_endpoint` variable includes your Sauce `USERNAME` and `ACCESSKEY.`

```ruby
driver = Selenium::WebDriver.for :remote, :url => sauce_endpoint, :desired_capabilities => caps
```
```ruby
sauce_endpoint = "http://sauceUsername:sauceAccessKey@ondemand.saucelabs.com:80/wd/hub"
```
The **Code Sample** and the section **Running Your Tests on Sauce** provides more detail on the driver and endpoint specifications.

## Prerequisites

-   Make sure you have the [selenium-webdriver](https://rubygems.org/gems/selenium-webdriver) gem installed
-   If you want to run tests in parallel you will also need the [parallel_tests](https://github.com/grosser/parallel_tests) gem
-   Our parallel test examples use RSpec. If you don't have [RSpec](http://rspec.info/) or a similar behavior-driven development (BDD) framework installed, you will need to pick one that you would like to use. If you have a BDD other than RSpec installed, you should be able to adapt our code examples to your favorite flavor of BDD. 

## Code Example

This code examples shows a very simple test to see if the title of Google's home page has changed, but it includes everything you need to set up and run an automated test on Sauce. 
```ruby
require "selenium/webdriver"
 
sauce_endpoint = "http://sauceUsername:sauceAccessKey@ondemand.saucelabs.com:80/wd/hub"
 
caps = {
  :platform => "Mac OS X 10.9",
  :browserName => "Chrome",
  :version => "31"
}
 
driver = Selenium::WebDriver.for :remote, :url => sauce_endpoint, :desired_capabilities => caps
driver.manage.timeouts.implicit_wait = 10
driver.navigate.to "http://www.google.com"
 
raise "Unable to load Google." unless driver.title.include? "Google"
 
query = driver.find_element :name, "q"
query.send_keys "Sauce Labs"
query.submit
 
puts driver.title
driver.quit
```
1.  Copy this code and save it into a file called `simple.rb`.
2.  Open a terminal and navigate to the file location. 
3.  Run `simple.rb`.

`ruby simple.rb`

Voila! You have just run your first test on Sauce, which you can verify by checking [your Dashboard](https://saucelabs.com/beta/dashboard).

## Running Tests in the Sauce Cloud

Let's look at our code sample in more detail to understand how you would use it as the basis for setting up your own tests from scratch, or how you can set up existing tests to run in the Sauce cloud. 

### Setting the Remote Option for Your Driver

If you wanted to run Selenium locally, you would initiate a driver for the browser you want to test against like this:

`driver = Selenium::WebDriver.for :firefox`

To run your test on Sauce, you use the `:remote` option for `driver`, and pass two parameters to it, `:url` and `:desired capabilities`. These parameters define how to access the Sauce cloud, and what platforms, browsers, and versions you want to test against. The values for these parameters and defined by the `sauce_endpoint` and `caps` variables, respectively.

```ruby
driver = Selenium::WebDriver.for :remote, :url => sauce_endpoint, :desired_capabilities => caps
```

If you had an existing test that you wanted to run on Sauce, all you would need to change is your `driver` definition, and make sure that the `sauce_endpoint` variable included your Sauce `USERNAME` and `ACCESSKEY.`

```ruby
sauce_endpoint = "http://sauceUsername:sauceAccessKey@ondemand.saucelabs.com:80/wd/hub"
```
**NOTE**: You can find your Access Key under **Accounts > Settings**.

### Defining the Platform, Browser, and Version You Want to Test Against

Now that your test is set up to run in the Sauce cloud, you need to define the platform, browser, and version you want the test to run against, which is where `:desired_capabilites` and `caps` come into play. 

```ruby
caps = { :platform => "Mac OS X 10.9", :browserName => "Chrome", :version => "31" }
```

You can manually enter the values you want for `:platform`, `:browsername`, and `:version`, or you can use the handy [Sauce Labs Platform Configurator](https://docs.saucelabs.com/reference/platforms-configurator/#/) to generate the `caps` values for any combination of platform, browser, and browser version. 


## Running Tests Against Local Applications
If your test application is not publicly available, you will need to use Sauce Connect so that Sauce can reach it. 

Sauce Connect is a tunneling app that allows you to execute tests securely when testing behind firewalls or on localhost. For more detailed information, please visit see the [Sauce Connect docs](https://docs.saucelabs.com/reference/sauce-connect/). 

## Running Tests in Parallel


"Well," you say, "this all looks really easy if I'm just running a single test against a single platform/browser/version, but I have multiple tests I need to run against multiple configurations." There are many ways you could accomplish this, but we're going to show the example of using RSpec, the behavior-driven development (BDD) framework for Ruby. 

First up, you'll need to add the RSpec, Selenium and ParallelTests gems into your Gemfile, then run `bundle install`.

```
source "https://www.rubygems.org"

gem "rspec", "~> 3.0.0"
gem "selenium-webdriver", "~> 2.47.1"
gem "parallel_tests", "~> 1.6.1"
```

Create a `spec` directory in your project's root directory. Inside that directory, you should create a file called `sauce_driver.rb`, which nicely encapsulates the behaviour we need to create Selenium drivers with Sauce Labs:

```ruby
require "selenium/webdriver"

module SauceDriver
  class << self
    def sauce_endpoint
      "http://#{ENV["SAUCE_USERNAME"]}:#{ENV['SAUCE_ACCESS_KEY']}@ondemand.saucelabs.com:80/wd/hub"
    end

    def caps(name)
      caps = {
        :platform => "Mac OS X 10.9",
        :browserName => "Chrome",
        :version => "31",
        :name => name
      }
    end

    def new_driver(name)
      Selenium::WebDriver.for :remote, :url => sauce_endpoint, :desired_capabilities => caps(name)
    end
  end
end

```
Now, you need to make RSpec create a new driver for each spec. The cleanest way to do that is by defining an `around` hook, which will be run, uh, around every spec. Create a file in the `spec` directory, called `spec_helper.rb`. This file will get `require` `d by every spec you create, as this is where, by convention, any setup code is placed:

```ruby
require "rspec"
require_relative "sauce_driver"
 
RSpec.configure do |config|
  config.around(:example, :run_on_sauce => true) do |example|
    @driver = SauceDriver.new_driver example.full_description
    job_id = @driver.session_id
    begin
      example.run
    ensure
      @driver.quit
    end
  end
end
```

On line 5 you set up a new block of code to be run `around` every example tagged with `:run_on_sauce => true`. This lets you have non-Selenium tests that don't spin up Selenium sessions. You have to make sure you include `:run_on_sauce` on all your Selenium tests, though!

On line 6 you use the `SauceDriver` class you set up earlier to create a new Selenium driver, 'pointed' at Sauce Labs. Then, on line 8, you run the example.  On lines 9 - 11 we call `quit` on the Selenium session; This is done in an `ensure` block so that each test always closes off its driver, even if something goes wrong.

Now you have RSpec set up to create drivers and close them down. Your next question might be, "How do I use these drivers?" Super simple.  Because `driver` is an instance variable, and the `around` block runs in the context of the spec, it can use the driver directly. Consider this example spec:

```ruby
require_relative "spec_helper"
 
describe "Google's Search Functionality" do
  it "can find search results", :run_on_sauce => true do 
    @driver.manage.timeouts.implicit_wait = 10
    @driver.navigate.to "http://www.google.com"
 
    raise "Unable to load Google." unless @driver.title.include? "Google"
       
    query = @driver.find_element :name, "q"
    query.send_keys "Sauce Labs"
    query.submit
 
    puts @driver.title
  end
end
```

On line 1 you require `spec_helper`. Then, on line 4 you add `:run_on_sauce => true` to make sure the `around` block runs. You can use the created driver, `@driver`, without any further setup, and you don't have to do anything to close it off either. This means you're WAY less likely to forget to do so when you write more tests.

Now, parallel testing! If you go ahead and create some more specs like this one (you can copy `google_spec.rb` into other files in the `spec` directory, just make sure the filenames end in `spec.rb`), then run the specs from your project root directory with:

`bundle exec parallel_rspec -n 2 spec/`

You should be able to log into Sauce Labs and see your tests running in parallel. If your account has more than two concurrency slots (meaning, you have a paid account), you can increase the number after `-n` to match your concurrency, and `parallel_tests` will spin up more tests at once.

## Reporting on Test Results

Selenium is a protocol focused on automating the actions of a browse, and as such, it doesn't encompass the notions of a 'test', 'passing' or 'failure'. Sauce Labs lets you notify us of test status using our [REST API.](https://docs.saucelabs.com/reference/rest-api/) All you need is the ID Sauce Labs gave the job, and the status your test ended with. Then, you can use the [Update Job](https://docs.saucelabs.com/reference/rest-api/#update-job) method on the REST API to set the job's status.

The Job ID is the simplest part of the process. The ID assigned to each job by Sauce Labs is the same as the Session ID for the corresponding Selenium session, which you can pull off the driver like so:

`job_id = driver.session_id`

Using the REST API is best done with the [SauceWhisk](https://rubygems.org/gems/sauce_whisk) gem, so add that to your Gemfile, and then run `bundle install`.

`gem "sauce_whisk"`

The SauceWhisk gem assumes you've set your Sauce username and access key as the `SAUCE_USERNAME` and `SAUCE_ACCESS_KEY` environment variables, so go ahead and do that.

Now, assuming you're using RSpec as shown above, you can make a simple change to` spec_helper.rb` to check the status of the test, and pass that to the REST API:

```ruby
require "rspec"
require_relative "sauce_driver"
require "sauce_whisk"
 
RSpec.configure do |config|
  config.around(:example, :run_on_sauce => true) do |example|
    @driver = SauceDriver.new_driver example.full_description
    job_id = @driver.session_id
    begin
      example.run
    ensure
      SauceWhisk::Jobs.change_status job_id, example.exception.nil?
      @driver.quit
    end
  end
end
```
That's it!  Your tests will now be reporting their status to the [Sauce Labs REST API](https://docs.saucelabs.com/reference/rest-api/).