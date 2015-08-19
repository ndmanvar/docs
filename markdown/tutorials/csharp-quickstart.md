{
  title: "C#",
  description: "How to run Selenium tests on Sauce Labs using C#",
  category: 'Tutorials',
  index: 10,
  image: "/images/tutorials/csharp.png"
}

## Getting Started with Sauce and C\#.NET

Sauce Labs is a cloud platform for executing automated and manual mobile and web tests. Sauce Labs supports running automated tests with Selenium Webdriver (for web applications) and Appium (for native and mobile web applications).

In this topic we’ll show you how to run a test with Selenium Webdriver for C\#.NET on Sauce Labs.

## Prerequisites

You need to have these components installed to set up testing on Sauce with C\#.NET:

-   Visual Studio
-   Selenium WebDriver
-   Selenium DLLs for .NET installed and referenced by your project
-   Gallio & MbUnit and Gallio Bundl

### Installing Selenium WebDriver

You [can also download the Selenium driver for C\#](http://www.seleniumhq.org/download/) from the official [Selenium HQ website](http://www.seleniumhq.org), where you can find additional documentation about working with Selenium.

1.  Open a new project in Visual Studio.
2.  Select a C\# class library template.
3.  Give the project a name and click **OK**.
4.  In the Visual Studio Tools menu, go to Library Package Manager \> Manage Nuget Package for Solution.
     This will open the **Manage NuGet Packages** screen.
5.  Click **Online**, and then click **Next.**
6.  In the **Search Packages** field, enter Selenium and then click **Search**.
7.  In the search results, select **Selenium Webdriver** and then click **Install**.

### Installing Selenium DLLs

After you’ve set up your project in Visual Studio, you need to make sure that it references the necessary Selenium DLLs for .NET

1.  Download the Selenium DLLs for .NET from <http://selenium-release.storage.googleapis.com/index.html?path=2.47/>
2.  In the Solutions Explorer, select the project and right-click **References**.
3.  Click Add Reference.
4.  Click **Browse** and navigate to the net40 folder of the directory where you saved the Selenium .NET DLLs.
5.  **Add** the WebDriver.Support.dll reference to your project.

### Installing Gallio & MbUnit and Gallio Bundle

The Gallio Automation Platform is an open, extensible, and neutral system for .NET that provides a common object model, runtime services and tools (such as test runners) that can be utilized by a variety  of test frameworks.

MbUnit is an extensible unit testing framework for the .NET Framework that takes in and goes beyond xUnit pattern testing. MbUnit is part of the Gallio bundle. For more information, see In the Visual Studio Tools menu, go to Library Package Manager \> Manage Nuget Package for Solution.
 This will open the **Manage NuGet Packages** screen.

1.  Click **Online**, and then click **Next.**
2.  In the Search Packages field enter Gallio, and then click **Search**.
3.  In the search results, select **Gallio & MbUnit**, and then click **Install**.
4.  After installing Gallio & MbUnit, select Gallio Bundle from the search results, and then click Install.

For more information on setting up the environment, see [Selenium downloads](http://www.seleniumhq.org/download/).

## Code Example

Now let’s take a look at some simple C\#.NET code. This example verifies the title, a link, and the presence of a user agent element on the page <https://saucelabs.com/test/guinea-pig>. It connects to Sauce Labs, run commands to remote control the target browser, and reports the results. It also runs against several browsers simultaneously, so you can see how parallelized testing works.


GuineaPigTests.cs
```csharp
using Gallio.Framework;
using Gallio.Model;
using MbUnit.Framework;
using OpenQA.Selenium;
using OpenQA.Selenium.Remote;
using OpenQA.Selenium.Support.UI;
using System;

namespace SauceLabs.SeleniumExamples

{
    /// \<summary\>tests for the sauce labs guinea pig page\</summary\>
    [TestFixture]
   [Header("browser", "version", "platform")] // name of the parameters in the rows
    [Row("internet explorer", "11", "Windows 7")] // run all tests in the fixture against IE 11 for windows 7
    [Row("chrome", "35", "linux")] // run all tests in the fixture against chrome 35 for linux
    [Row("safari","6","OS X 10.8")] // run all tests in the fixture against safari 6 and mac OS X 10.8
    public class GuineaPigTests

    {
        #region Setup and Teardown
        
        /// \<summary\>starts a sauce labs sessions\</summary\>
        /// \<param name="browser"\>name of the browser to request\</param\>
        /// \<param name="version"\>version of the browser to request\</param\>
        /// \<param name="platform"\>operating system to request\</param\>
        private IWebDriver \_Setup(string browser, string version, string platform)
        {
            // construct the url to sauce labs
            Uri commandExecutorUri = new Uri("<http://ondemand.saucelabs.com/wd/hub>");
            // set up the desired capabilities
            DesiredCapabilities desiredCapabilites = new DesiredCapabilities(browser, version, Platform.CurrentPlatform); // set the desired browser
            desiredCapabilites.SetCapability("platform", platform); // operating system to use
            desiredCapabilites.SetCapability("username", Constants.SAUCE\_LABS\_ACCOUNT\_NAME); // supply sauce labs username
            desiredCapabilites.SetCapability("accessKey", Constants.SAUCE\_LABS\_ACCOUNT\_KEY);  // supply sauce labs account key
            desiredCapabilites.SetCapability("name", TestContext.CurrentContext.Test.Name); // give the test a name

            // start a new remote web driver session on sauce labs
            var \_Driver = new RemoteWebDriver(commandExecutorUri, desiredCapabilites);
            \_Driver.Manage().Timeouts().ImplicitlyWait(TimeSpan.FromSeconds(30));

            // navigate to the page under test
            \_Driver.Navigate().GoToUrl("<https://saucelabs.com/test/guinea-pig>");

            return \_Driver;
        }

        /// \<summary\>called at the end of each test to tear it down\</summary\>
        public void CleanUp(IWebDriver \_Driver)
        {
            // get the status of the current test
            bool passed = TestContext.CurrentContext.Outcome.Status == TestStatus.Passed;
            try
            {
                // log the result to sauce labs
                ((IJavaScriptExecutor)\_Driver).ExecuteScript("sauce:job-result=" + (passed ? "passed" : "failed"));
            }
            finally
            {
                // terminate the remote webdriver session
                \_Driver.Quit();
            }
        }

        \#endregion

        \#region Tests

        /// \<summary\>tests the title of the page\</summary\>
        [Test, Parallelizable] // denotes that this method is a test and can be run in parallel
        public void PageTitle(string browser, string version, string platform)
        {
            // start the remote webdriver session with sauce labs
            var \_Driver = \_Setup(browser, version, platform);


            // verify the page title is correct
            Assert.Contains(\_Driver.Title, "I am a page title - Sauce Labs");
            CleanUp(\_Driver);
        }

/// \<summary\>tests that the link works on the page\</summary\>

        [Test, Parallelizable] // denotes that this method is a test and can be run in parallel
        public void LinkWorks(string browser, string version, string platform)
        {
            // start the remote webdriver session with sauce labs
            var \_Driver = \_Setup(browser, version, platform);

            // find and click the link on the page
            var link = \_Driver.FindElement(By.Id("i am a link"));
            link.Click();

            // wait for the page to change
            WebDriverWait wait = new WebDriverWait(\_Driver, TimeSpan.FromSeconds(30));
            wait.Until((d) =\> { return d.Url.Contains("guinea-pig2"); });
            // verify the browser was navigated to the correct page
            Assert.Contains(\_Driver.Url, "[saucelabs.com/test-guinea-pig2.html](http://saucelabs.com/test-guinea-pig2.html)");

            CleanUp(\_Driver);
        }

        /// \<summary\>tests that a useragent element is present on the page\</summary\>
        [Test, Parallelizable] // denotes that this method is a test and can be run in parallel
        public void UserAgentPresent(string browser, string version, string platform)
        {
            // start the remote webdriver session with sauce labs
            var \_Driver = \_Setup(browser, version, platform);

            // read the useragent string off the page
            var useragent = \_Driver.FindElement(By.Id("useragent")).Text;

            Assert.IsNotNull(useragent);
            CleanUp(\_Driver);
        }

        \#endregion
    }
}
```
 

Constants.cs

Use this class to set your Sauce Labs account name and access key. You can find these in the User Profile menu of your Sauce Labs dashboard.

```csharp
namespace SauceLabs.SeleniumExamples

{
    /// \<summary\>contains constants used by the tests such as the user name and password for the sauce labs\</summary\>
    internal static class Constants
    {
        /// \<summary\>name of the sauce labs account that will be used\</summary\>
        internal const string SAUCE\_LABS\_ACCOUNT\_NAME = "sauceUsername";
        /// \<summary\>account key for the sauce labs account that will be used\</summary\>
        internal const string SAUCE\_LABS\_ACCOUNT\_KEY = "sauceAccessKey";

        // NOTE:
        // To change the maximum number of parallel tests edit DegreeOfParallelism in AssemblyInfo.cs

    }
}
```
## Running the Test

1.  In the Solution Explorer, select your project and right-click Properties.
2.  Select **Debug**.
3.  Under Start Action, select **Start an External Program**.
4. Set the path to the **Gallio Icarus GUI Runner**.
 The default path will probably look something like this:  *C:\\Program Files (x86)\\Gallio\\bin\\Gallio.Icarus.exe.*
1.  Set the command line arguments to the name of your DLL.
     For this example you should set it to *SauceLabs.SeleniumExamples.dll.*
2.  Press **F5** to build the project and debug the solution.
     This launches the Gallio test GUI where you can run your tests. The DLL for this project should be preloaded. If it isn’t, you can open it from the **File** menu.

## Running Tests Against Local Applications

If your test application is not publicly available, you will need to use Sauce Connect so that Sauce can reach it.

Sauce Connect is a tunneling app that allows you to execute tests securely when testing behind firewalls or on localhost. For more detailed information, please visit see the [Sauce Connect docs](https://docs.saucelabs.com/reference/sauce-connect/).