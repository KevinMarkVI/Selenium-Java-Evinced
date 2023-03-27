---
title: "Selenium Java SDK"
metaTitle: "Selenium SDK Java Documentation"
index: 4
---

The Evinced Selenium Java SDK integrates with new or existing Selenium tests to automatically detect accessibility issues. With the addition of as few as 5 lines of code to your Selenium framework, you can begin to analyze all the pages and DOM changes to offer a dynamic view of how your site can become more accessible. At the conclusion of the test, a rich and comprehensive JSON or HTML report is generated to track issues in any reporting tool.

Interested in seeing this in action? [Contact us] to get started!

## Prerequisites

-   Selenium version 3.141.59 or higher
-   Java Version 1.8 or higher
-   ChromeDriver (More coming soon!)

## Get Started

### Install Evinced Selenium SDK from a local ZIP archive

You can perform installation right from the standalone .jar distribution. In this case, you need to do the following:

1. Download the .jar file.
2. Unpack the provided selenium-sdk.zip to any desirable location
3. Add the following dependencies entries pointing to selenium-sdk-all.jar
    1. Gradle:
        ```java
         implementation files('/Users/<path-to-your-upacked-folder>/selenium-sdk.jar'
        ```
    2. Maven:
        1. First, install the “all“ jar into your local Maven repository by the following command:
            ```java
             mvn install:install-file –Dfile=/Users/<path-to-unpacked-folder>/selenium-sdk.jar -DgroupId=com.evinced -DartifactId=java-selenium -Dversion=0.0.23
            ```
        2. Add the corresponding dependency into your pom.xml:
            ```xml
             <dependency>
                 <groupId>com.evinced</groupId>
                 <artifactId>java-selenium</artifactId>
                 <version>0.0.26</version>
             </dependency>
            ```
4. Trigger your package manager download new dependencies either via IDE or command line

ZIP archive also contains two complimentary .jar files which you can utilize:

-   `selenium-sdk-javadoc.jar` contains rich JavaDoc documentation which you can view in any web browser
-   `selenium-sdk-sources.jar` can be attached to your Evinced Selenium SDK installation in order to let you leverage JavaDoc right in your IDE. Please, refer to the corresponding documentation of your developer tools.

### License

You need an auth token to work with SDK.  
Set it one of the ways in [Selenium Java SDK | Licensing](https://evinced.atlassian.net/wiki/spaces/DOCS/pages/1562443889/Selenium+Java+SDK#Licensing).

## Your first test

### Issue Type Configuration - aXe Needs Review and Best Practices

Evinced uses two separate engines when scanning for accessibility issues, one is the aXe engine and the other is the Evinced engine. By default, Evinced disables the aXe Needs Review and Best Practices issues based on customer request and due to the fact they are mostly false positives. Please note this setting when comparing issue counts directly. See an example of how to enable Needs Review and Best practices issues in our [API](#setexperimentalflagsandaddexperimentalflag) section.

### Initialize the SDK object

Add the import statement to the beginning of your test file.

```java
import com.evinced.EvincedWebDriver;
```

Under the hood, the Evinced SDK utilizes several Selenium commands in order to get accessibility data. For this reason, we need to initialize `EvincedWebDriver` with the `ChromeDriver` instance in the following way:

```java
EvincedWebDriver driver = new EvincedWebDriver(new ChromeDriver());
```

### Add Evinced accessibility checks (Single Run Mode)

This is a simple JUnit example of how to add a single Evinced accessibility scan to a test. Please note the inline comments that give detail on each test step.

```java
	@Test
	public void evAnalyzeSimpleTest() {
	    // Initialize EvincedWebDriver which wraps a ChromeDriver instance
		EvincedWebDriver driver = new EvincedWebDriver(new ChromeDriver());

		// Navigate to the site under test
		driver.get("https://www.google.com");

		// Run analysis and get the accessibility report
		Report report = driver.evAnalyze();

		// Assert that there are no accessibility issues
		assertTrue(report.getIssues().size() == 0);
	}
```

### Add Evinced accessibility checks (Continuous Mode)

This is a simple JUnit example of how to add a continuous Evinced accessibility scan to a test. Using the `evStart` and `evStop` [methods](#evstart), the Evinced engine will continually scan in the background capturing all DOM changes as the test is executed. This will capture all accessibility issues as clicking on dropdowns or similar interactions reveals more of the page. The advantage of continuous mode is that no interaction with the actual test code is needed.

```java
@BeforeClass
public static void setUp() throws MalformedURLException, IOException {
    driver = new EvincedWebDriver(new ChromeDriver());
    driver.setupCredentials(<your service account ID>, <your API key>);
    // Start the Evinced engine scanning for accessibility issues
    driver.evStart();
    //... the rest of our setup code
}

@AfterClass
public static void tearDown() throws MalformedURLException, IOException {
    // Stop the Evinced engine
    Report report = driver.evStop();
    //Output the Evinced report in either JSON or HTML format
    EvincedReporter.evSaveFile("test-results", report, EvincedReporter.FileFormat.HTML);
    EvincedReporter.evSaveFile("test-results", report, EvincedReporter.FileFormat.JSON);
    driver.quit();
}

@Test
public void evAnalyzeSimpleTest() {
	// Navigate to the site under test
	driver.get("https://www.google.com");
	// More test code...
}
```

## API

### `EvincedWebDriver`

#### `constructor`

##### Default constructor

The `EvincedWebDriver` constructor expects an instance of `ChromeWebDriver`. It is possible to pass an instance of a class that inherits or extends ChromeDriver.

```java
ChromeDriver chromeDriver = new ChromeDriver();
EvincedWebDriver driver = new EvincedWebDriver(chromeDriver);
```

##### Additional configuration

An optional second parameter is an `EvincedConfiguration`. When configurations are passed via the constructor, they are set as a default for all future actions.

In the case that other configurations are passed to a specific function, such as `evAnalyze`, they will override the constructor configurations for that specific operation.

See the `EvincedConfiguration` options section below for more details.

```java
ChromeDriver chromeDriver = new ChromeDriver();
EvincedConfiguration configuration = new EvincedConfiguration();
EvincedWebDriver driver = new EvincedWebDriver(chromeDriver, configuration);
```

#### `evAnalyze`

Scans the current page and returns a list of accessibility issues. A Report object is returned containing accessibility issues. This is the recommended method for static page analysis. **Note:** This method is not supported if `evStart` is already running.

```java
Report report = evincedWebDriver.evAnalyze();
```

When the `EvincedConfiguration` object is provided to the `evAnalyze` method it will be used for this run only (replacing it, not merging), and will not be saved for future runs.

```java
Report report = evincedWebDriver.evAnalyze(configuration);
```

#### `evStart`

Continually watches for DOM mutations and page navigation and records all accessibility issues until the evStop() method is called. This method is recommended for dynamic page flows.

```java
evincedWebDriver.evStart();
```

When the `EvincedConfiguration` object is provided to the `evStart` method it will be used for this run only (replacing it, not merging), and will not be saved for future runs.

```java
Report report = evStart.evAnalyze(configuration);
```

#### `evStop`

Returns a Report object containing recorded accessibility issues from the point in time at which the `evStart` method was instantiated.

```java
Report report = evincedWebDriver.evStop();
```

#### `setLogLevel`

Sets the log level for Evinced code. Default is no logging.

```java
evincedWebDriver.setLogLevel(Level.ERROR); // or any other level such as Level.DEBUG
```

### `EvincedReporter`

Manages report file creation. Add the import to your file:

```java
import com.evinced.EvincedWebDriver;
```

#### `evSaveFile`

Writes the accessibility report to a file. Returns the `Path` to the created file.

Two formats are available - HTML and JSON.

```java
    Report report = evincedWebDriver.evStop();
     -- or --
    Report report = evincedWebDriver.evAnalyze();

    // create a JSON file named jsonReport.json
    EvincedReporter.evSaveFile("jsonReport", report, EvincedReporter.FileFormat.JSON);

    // create an HTML file named htmlReport.html
    EvincedReporter.evSaveFile("htmlReport", report, EvincedReporter.FileFormat.HTML);

    // create an SARIF file named sarifReport.html
    EvincedReporter.evSaveFile("sarifReport", report, EvincedReporter.FileFormat.SARIF);
```

#### `FileFormat`

Determines the file type of the outputted report. Options are `JSON` and `HTML`.

```java
// create a JSON file named jsonReport.json
EvincedReporter.evSaveFile("jsonReport", report, EvincedReporter.FileFormat.JSON);

// create an HTML file named htmlReport.html
EvincedReporter.evSaveFile("htmlReport", report, EvincedReporter.FileFormat.HTML);
```

### `EvincedConfiguration`

Use the `EvincedConfiguration` object to pass configuration parameters to the accessibility engines. When passing the configuration to the constructor it will be the default for future `evStart` and `evAnalyze` commands.

```java
ChromeDriver chromeDriver = new ChromeDriver();
EvincedConfiguration configuration = new EvincedConfiguration();
EvincedWebDriver driver = new EvincedWebDriver(chromeDriver, configuration);
```

When passing the configuration to the `evStart` or `evAnalyze` commands it will temporarily override the constructor configurations (replacing it, not merging).

#### `setRootSelector`

Choose a single CSS selector to run the analysis, For example - run analysis on the element that holds only the menu bar. When no configuration is passed, it will scan the entire page. Default is null.

```java
EvincedConfiguration configuration = new EvincedConfiguration();
configuration.setRootSelector(".some-selector"); // css selector
EvincedWebDriver driver = new EvincedWebDriver(chromeDriver, configuration);
```

#### `axeConfig`

Evinced leverages Axe open-source accessibility toolkit as part of its own accessibility detection engine. The Evinced engine is able to pinpoint far more issues than Axe alone. See [Axe Configuration Options]

```java
EvincedConfiguration configuration = new EvincedConfiguration();
AxeConfiguration axeConfig = new AxeConfiguration();

// Axe's syntax to make `html-has-lang` validation disabled
axeConfig.setRules(Collections.singletonMap("html-has-lang", Collections.singletonMap("enabled", false)));
configuration.setAxeConfig(axeConfig);

// passing axe config on constructor
EvincedWebDriver driver = new EvincedWebDriver(chromeDriver, configuration);

-- or --
// passing axe config via evStart
EvincedWebDriver driver = new EvincedWebDriver(chromeDriver);
driver.evStart(configuration);
```

#### `includeIframes [BETA]`

When set to `true`, the accessibility tests run analysis on iframes that exist inside the page. Default is `false`.

```java
EvincedConfiguration configuration = new EvincedConfiguration();
configuration.setIncludeIframes(true);
driver.evStart(configuration);

// the report will include issues that exist on a page or in iframes inside it
Report report = driver.evStop();
```

### `setIncludeShadowDOM [BETA]`

When set to `true`, the accessibility tests run analysis on shadow DOM elements that exist inside the page. Default is `false`.

```java
EvincedConfiguration configuration = new EvincedConfiguration();
configuration.setIncludeShadowDOM(true);
driver.evStart(configuration);

// the report will include issues that exist on a page and its shadow DOM elements
Report report = driver.evStop();
```

### `setExperimentalFlags and addExperimentalFlag`

You can add one or more experimental flags to the `EvincedConfig` object.

```java
// Globally
EvincedConfiguration globalConfig = new EvincedConfiguration();
Map<String, Boolean> toggles = new HashMap<String, Boolean>(){{
  put(FLAG_NAME_1, false);
  put(FLAG_NAME_2, true);
}};
globalConfig.setExperimentalFlags(toggles);
driver.setConfiguration(globalConfig);

// Example enabling aXe Needs Review and Best Practices Issues
EvincedConfiguration globalConfig = new EvincedConfiguration();
globalConfig.addExperimentalFlag(USE_AXE_NEEDS_REVIEW, true);
globalConfig.addExperimentalFlag(USE_AXE_BEST_PRACTICES, true);
driver.setConfiguration(globalConfig);

// While running some command
Map<String, Boolean> experimentalFlags = new HashMap<String, Boolean>(){{
  put(FLAG_NAME_1, false);
  put(FLAG_NAME_2, true);
}};
EvincedConfiguration localConfig = new EvincedConfiguration(flags);
driver.get(page1);
driver.evAnalyze(localConfig);

localConfig.addExperimentalFlag(ONE_MORE_FLAG, true);
driver.get(page2);
driver.evAnalyze(localConfig);
```

## Tutorials

### Generating a comprehensive accessibility report for your application

In this tutorial, we will enhance our existing Selenium UI test with the Evinced Selenium SDK in order to check our application for accessibility issues. In order to get started you will need the following:

1. All of the prerequisites for the Evinced Selenium SDK should be met
2. Evinced Selenium SDK should be added to your project

#### Preface - existing UI test overview

Let’s consider the following basic UI test as our starting point.

```java
import com.evinced.EvincedReporter;
import com.evinced.EvincedWebDriver;
import com.evinced.dto.results.Issue;
import com.evinced.dto.results.Report;
import org.junit.Assert;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import java.util.List;

public class AccessibilityTests {
	@Test
	public void evAnalyzeWithAnOpenDropdownTest() {
		ChromeDriver driver = new ChromeDriver();
		driver.get("https://demo.evinced.com/");
        WebElement firstDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(1) > div > div.dropdown.line"));
        firstDropdown.click();
        WebElement secondDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(2) > div > div.dropdown.line"));
        secondDropdown.click();
        driver.findElement(By.cssSelector(".react-date-picker")).click();
	}
}
```

We wrote this test for a demo travel site called [TRVL] that has a few known accessibility issues.

The purpose of this test is to check the functionality of the main application screen and ensure a user can successfully select their desired trip. For now, this test is only concerned with the functional testing of the app. However, with the help of the Evinced Selenium SDK, we can also check it for accessibility issues along the way. Let’s go through this process with the following step-by-step instructions.

#### Step #1 - Initialize the Evinced Selenium SDK

Before making any assertions against our app, we need to initialize `EvincedWebDriver` object. This object is used primarily as an entry point to all of the accessibility scanning features. Since we are going to use it scan throughout our test, the best place for its initialization will be our setUp method which gets executed first.

```java
import com.evinced.EvincedReporter;
import com.evinced.EvincedWebDriver;
import com.evinced.dto.results.Issue;
import com.evinced.dto.results.Report;
import org.junit.Assert;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import com.evinced.EvincedWebDriver;
import java.util.List;

public class AccessibilityTests {
    private EvincedWebDriver driver;

    @Before
    public void setUp() {
        driver = new EvincedWebDriver(new ChromeDriver());
    }

	@Test
	public void evAnalyzeWithAnOpenDropdownTest() {
		driver.get("https://demo.evinced.com/");
        WebElement firstDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(1) > div > div.dropdown.line"));
        firstDropdown.click();
        WebElement secondDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(2) > div > div.dropdown.line"));
        secondDropdown.click();
        driver.findElement(By.cssSelector(".react-date-picker")).click();
	}
}
```

The `EvincedWebDriver` requires only an instance of ChromeDriver.

#### Step #2 - Start the Evinced engine

Now that we have initiated the Selenium driver and added the capabilities needed to scan for accessibility issues, let’s start the Evinced engine. Since we are going to use it scan throughout our test, the best place for its initialization will be our `setUp` method after our Selenium driver is configured.

```java
import com.evinced.EvincedReporter;
import com.evinced.EvincedWebDriver;
import com.evinced.dto.results.Issue;
import com.evinced.dto.results.Report;
import org.junit.Assert;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import com.evinced.EvincedWebDriver;
import java.util.List;

public class AccessibilityTests {
    private EvincedWebDriver driver;

    @Before
    public void setUp() {
        driver = new EvincedWebDriver(new ChromeDriver());
        // Start the Evinced Engine
        driver.evStart();
    }

	@Test
	public void evAnalyzeWithAnOpenDropdownTest() {
		driver.get("https://demo.evinced.com/");
        WebElement firstDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(1) > div > div.dropdown.line"));
        firstDropdown.click();
        WebElement secondDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(2) > div > div.dropdown.line"));
        secondDropdown.click();
        driver.findElement(By.cssSelector(".react-date-picker")).click();
	}
}
```

There are additional configuration options that can be included as well. More information can be found in the API section of this documentation.

#### Step #3 - Stop the Evinced engine and create reports

As our test was executed we collected a lot of accessibility information. We can now perform accessibility assertions at the end of our test suite. Referring back again to our UI test the best place for this assertion will be the method that gets invoked last - `tearDown`. To stop the Evinced engine and generate the actual object representation of your accessibility report simply call the `evStop()` method. We can then output the report files in either HTML or JSON format (or both!).

```java
import com.evinced.EvincedReporter;
import com.evinced.EvincedWebDriver;
import com.evinced.dto.results.Issue;
import com.evinced.dto.results.Report;
import org.junit.Assert;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;
import com.evinced.EvincedWebDriver;
import java.util.List;

public class AccessibilityTests {
    private EvincedWebDriver driver;

    @Before
    public void setUp() {
        driver = new EvincedWebDriver(new ChromeDriver());
        // Start the Evinced Engine
        driver.evStart();
    }

    @After
    public void tearDown() {
        // Stop the Evinced Engine
        Report report = driver.evStop();
        // Output the Accessibility results in JSON or HTML
        EvincedReporter.evSaveFile(testName.getMethodName(), report, EvincedReporter.FileFormat.HTML);
        EvincedReporter.evSaveFile(testName.getMethodName(), report, EvincedReporter.FileFormat.JSON);
        driver.quit();
    }

	@Test
	public void evAnalyzeWithAnOpenDropdownTest() {
		driver.get("https://demo.evinced.com/");
        WebElement firstDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(1) > div > div.dropdown.line"));
        firstDropdown.click();
        WebElement secondDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(2) > div > div.dropdown.line"));
        secondDropdown.click();
        driver.findElement(By.cssSelector(".react-date-picker")).click();
	}
}
```

For the sake of simplicity of this tutorial let’s simply assume that our application is accessible as long as it has no accessibility issues found. Thus, if we have at least one accessibility issue detected - we want our tests to be failed. Let’s add the corresponding assertion to our `tearDown` method

```java
@After
public void tearDown() {
    // Stop the Evinced Engine
    Report report = driver.evStop();
    // Output the Accessibility results in JSON or HTML
    EvincedReporter.evSaveFile(testName.getMethodName(), report, EvincedReporter.FileFormat.HTML);
    EvincedReporter.evSaveFile(testName.getMethodName(), report, EvincedReporter.FileFormat.JSON);

    // Optional assertion for gating purposes
    List<Issue> issues = report.getIssues();
    Assert.assertEquals(0, issues.size());

    driver.quit();
}
```

You are now set to run the test and ensure the accessibility of your application! So, go ahead and run it via your IDE or any other tooling you use for Java development.

## Additional Configuration Examples

### Testing accessibility in a specific state of the application

In this example, we are going to test the accessibility of a page in a specific state.
This test navigates to a page, opens a dropdown, and only then runs the accessibility engines to identify issues.

```java
import com.evinced.EvincedReporter;
import com.evinced.EvincedWebDriver;
import com.evinced.dto.results.Issue;
import com.evinced.dto.results.Report;
import org.junit.Assert;
import org.junit.Test;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.chrome.ChromeDriver;

import java.util.List;

public class AccessibilityTests {

	@Test
	public void evAnalyzeWithAnOpenDropdownTest() {
		EvincedWebDriver driver = new EvincedWebDriver(new ChromeDriver());
		driver.get("https://demo.evinced.com/");

		// interacting with the page - opening all dropdown
		WebElement firstDropdown = driver.findElement(By.cssSelector("div.filter-container > div:nth-child(1) > div > div.dropdown.line"));
		firstDropdown.click();

		Report report = driver.evAnalyze();
		List<Issue> issues = report.getIssues();
		Assert.assertEquals(8, issues.size());

		// write the report to HTML format to a file named: "test-results.html"
		EvincedReporter.evSaveFile("test-results", report, EvincedReporter.FileFormat.HTML);
	}
}
```

### Running on multiple tabs

Evinced supports performing accessibility tests that involve multiple tabs.

#### Running evAnalyze on multiple tabs

The `evAnalyze` command will works as expected on the primary tab. When testing on an additional browser tab, simply switch the driver from the primary Selenium “window handle” to the “window handle” of the new tab and then run `evAnalyze`. This code is shown on lines `14-15` of the example below.

Example In the example below, `page1.html` has a link that opens in a new tab.

```java
// navigate to page1
driver.get("page1.html");

// get report for page1
Report report = driver.evAnalyze();
List<Issue> issues = report.getIssues();
// run some assertions
assertEquals(1, issues.size());

// perform an action that opens a new tab
driver.findElement(new By.ById("open-tab-link")).click();

// switch to new tab
String tabWindowHandle = driver.getWindowHandles().toArray()[1].toString();
driver.switchTo().window(tabWindowHandle);

// get report for the new tab,
// the issues for page1 won't appear here, only ones found on the new tab
report = driver.evAnalyze();
issues = report.getIssues();
assertEquals(3, issues.size());
```

#### Running evStart and evStop on multiple tabs

The `evStart` and `evStop` method scopes are limited to a single tab or Selenium “window handle”. This means that if you have more than one tab, you will need to call `evStop` and then switch to the window handle of the new tab. Once the “window handle” switch has occurred, run a new instance of `evStart` followed `evStop` at the appropriate time.`evStop` must be called on the tab that `evStart` command was called on, before starting `evStart` on a new tab.

In the example below, `page1.html` has a link that opens in a new tab.

```java
   // navigate to page1
    driver.get("page1.html");

    driver.evStart();

    // perform actions on page1, clicks, etc...

    // the evStart and evStop scopes are limited to a single window handle.
    // this means that if you have more than one tab, you need to switch it's window and only then run `evStart` and `evStop`
    // The `evStop` must be called on the tab that `evStart` command was called on, before starting `evStart` on a new tab
    Report report = driver.evStop();
    List<Issue> issues = report.getIssues();
    assertEquals(1, issues.size());

    // perform an action that opens a new tab
    driver.findElement(new By.ById("open-tab-link")).click();

    // move to tab
    String tabWindowHandle = driver.getWindowHandles().toArray()[1].toString();
    driver.switchTo().window(tabWindowHandle);

    // get report for the new tab,
    // the issues for page1 won't appear here, only ones found on the new tab
    driver.evStart();
    report = driver.evStop();
    issues = report.getIssues();
    assertEquals(3, issues.size());
```

#### Running evStart and evStop on a different TargetLocator (frame / iframe)

In case the `evStart` command is called on a different context, for example - after switching WebDriver’s context into an iframe, the Evinced engines will be injected into that iframe only.
In order to get the results back you will need to switch to the same `context` `evStart` was called on, before calling `evStop`.
In case you are using a website that relies deeply on iframes, we suggest using the `IncludeIframes` API configuration.

In the example below, `page1.html` has an iframe in it, with id `iframe1`.

```java
    // navigate to page1
    driver.get("page1.html");

    // switch to the iframe
    driver.switchTo().frame(driver.findElement(By.id("iframe1")));

    // start recording
    driver.evStart();

    // perform actions on iframe1 inside page1, clicks, etc...

    // if you go back to the defaultContent (the iframe parent)
    // the `evStop` method will fail
    driver.switchTo().defaultContent();

    // don't do that, it will fail
    driver.evStop();

    // but you can get the results when inside the iframe
    // switch to the iframe
    driver.switchTo().frame(driver.findElement(By.id("iframe1")));

    // now evStop returns all the issues recorded inside the iframe
    Report report = driver.evStop();
    List<Issue> issues = report.getIssues();
    assertEquals(1, issues.size());
```

### Add accessibility testing to existing tests

It is possible to configure `EvincedWebDriver` to be called also used in `@Before` and `@After` methods. In the example below, `evStart` starts recording before the test starts, and `evStop` is called after each test.

```java
import com.evinced.dto.results.Report;
import io.github.bonigarcia.wdm.WebDriverManager;
import org.junit.*;
import org.junit.rules.TestName;
import org.openqa.selenium.chrome.ChromeDriver;

public class BeforeAfterTest {

	@Rule
	public TestName testName = new TestName();

	private EvincedWebDriver driver;

	/**
	 * Instantiate the WebDriver
	 */
	@BeforeClass
	public void setupClass() {
		WebDriverManager.chromedriver().setup();
		driver = new EvincedWebDriver(new ChromeDriver());
		driver.manage().window().maximize();
	}

	/**
	 * Start recording before each test
	 */
	@Before
	public void setUp() {
		driver.evStart();
	}

	/**
	 * Ensure we close the WebDriver after finishing
	 */
	@After
	public void after() {
		// write an HTML report to file
		Report report = driver.evStop();
		EvincedReporter.evSaveFile(testName.getMethodName(), report, EvincedReporter.FileFormat.HTML);
	}

	@Test
	public void visitDemoPage() {
		driver.get("https://demo.evinced.com");
	}

	@Test
	public void visitGoogle() {
		driver.get("https://www.google.com");
	}
}
```

### Add skip validations

```java
    //define skipValidations list
    List<Validation> skipValidations = new ArrayList<>();

    //define validation with parameters
    // 1. selector
    // 2. url
    // 3... validationTipes
    Validation validation = new Validation("test.selector",
    "http\\.test.url\\.org",
    "WRONG_SEMANTIC_ROLE",
    "NOT_FOCUSABLE",
    "NO_DESCRIPTIVE_TEXT");

    //add all validations to the list
    skipValidations.add(validation);

    //update skip validations list in global configuration
    globalConfig.setSkipValidationsList(skipValidations);

    //update global config in driver
    driver.setConfiguration(globalConfig);
```

### Licensing

`evStart()` and `evAnalyze()` required auth token now for work, it is given in `EvincedSdk`.

`EvincedSdk.getInstance().setCredentials(String secret, String serviceId)` - get auth token form server and set it to `EvincedSdk`.

`EvincedSdk.getInstance().setOfflineCredentials(String authToken, String serviceId)` - set auth token and service id to `EvincedSdk` directly.

`EvincedSDK` is a global object, so you can set credentials once befero the first call of any evinced command.

**We encourage to use environment variables to store credentials and read them in code**

```java
export AUTH_SERVICE_ID=<serviceId>
export AUTH_TOKEN=<secret>
```

**Examplw**

```java
    //define driver for test
    ChromeOptions chromeOptions = ChromeOptionsHelper.getHeadlessOptionsConfiguration();
    EvincedWebDriver driver = new EvincedWebDriver(new ChromeDriver(chromeOptions));
    driver.manage().window().maximize();

    //set token to EvincedSDK
    EvincedSDK.getInstance().setOfflineCredentials(System.getenv("AUTH_TOKEN"), System.getenv("AUTH_SERVICE_ID"));
    //evAnalyze should work corectly
    driver.evAnalyze();
```

## Support

Please feel free to reach out to [support@evinced.com] with any questions.

## FAQ

1. **Do you have support for languages other than Java?**  
   Yes! We have SDK versions for JavaScript as well as Cypress and WebdriverIO. We have implementations for Ruby, Python, and C# coming soon.
2. **Can I configure which validations to run?**  
   Yes, see the API section for details on how to configure validations to your needs.
3. **Can I run tests with Evinced using cloud-based services like Sauce Labs, Perfecto, or BrowserStack?**  
   Yes, we have tested the Evinced SDK on many of these types of cloud-based services and expect no issues.

[contact us]: https://www.evinced.com/contact/demo-request
[support@evinced.com]: mailto:support@evinced.com
[axe configuration options]: https://github.com/dequelabs/axe-core/blob/develop/doc/API.md#api-name-axeconfigure
[trvl]: https://demo.evinced.com/
