{
  title: "Java",
  description: "How to run Selenium tests on Sauce Labs using Java",
  category: 'Tutorials',
  index: 0,
  image: "/images/tutorials/java.png"
}

Questions or Problems with this Tutorial? Go to [our Forum topic for this Java tutorial.](https://support.saucelabs.com/entries/57636760-Java-Tutorial-Questions-Problems)

## Quickstart Tutotiral
If new to Selenium, then checkout our quick start tutorials

## Launching a browser on Sauce

#### Using Selenium-webdriver
```java
    import org.openqa.selenium.WebDriver;
    import org.openqa.selenium.remote.CapabilityType;
    import org.openqa.selenium.remote.DesiredCapabilities;
    import org.openqa.selenium.remote.RemoteWebDriver;
    import java.net.URL;

    String username = "YOUR_USERNAME"; // replace YOUR_USERNAME with actual sauce username
    String password = "YOUR_PASSWORD"; // replace YOUR_USERNAME with actual sauce username
    SauceOnDemandAuthentication authentication = new SauceOnDemandAuthentication(username, password);

    DesiredCapabilities capabilities = new DesiredCapabilities();
    capabilities.setCapability(CapabilityType.BROWSER_NAME, "chrome");
    capabilities.setCapability(CapabilityType.VERSION, "33");
    capabilities.setCapability(CapabilityType.PLATFORM, "Windows 7");

    driver = new RemoteWebDriver(
                new URL("http://" + authentication.getUsername() + ":" + authentication.getAccessKey() +
                        "@ondemand.saucelabs.com:80/wd/hub"),
                capabilities);
```

## Setups with built-in parallelism

Feel free to check out the various repositoies and start writing tests!

#### Selenium
* [Java / JUnit](https://github.com/ndmanvar/SeleniumJavaJunit/)
* [Java / TestNG](https://github.com/ndmanvar/SeleniumJavaTestNG/)

#### Appium
* [Java / JUnit](https://github.com/ndmanvar/SeleniumJavaJunit/)
* [Java / TestNG](https://github.com/ndmanvar/SeleniumJavaTestNG/)


