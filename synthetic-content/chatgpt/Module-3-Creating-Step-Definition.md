# Module 3: Creating Step Definitions 🔧

## Learning Objectives

- Write JavaScript functions to implement Gherkin steps.
- Use Puppeteer’s API to automate browser actions in step definitions.
- Debug and organize step definitions effectively.

---

## Key Topics

### **1. Step Definitions Basics**

Step definitions connect the steps in your feature files to JavaScript code. They:

- Use regular expressions (regex) to match Gherkin steps.
- Execute the logic needed to fulfill the test case.

#### Example Structure:

A simple step definition looks like this:

```javascript
const { Given, When, Then } = require('@cucumber/cucumber');

Given('the user is on the login page', async function () {
  await this.page.goto('https://example.com/login');
});

When('the user enters valid credentials', async function () {
  await this.page.type('#username', 'validUser');
  await this.page.type('#password', 'validPassword');
  await this.page.click('#loginButton');
});

Then('the user should see the dashboard', async function () {
  const dashboardTitle = await this.page.title();
  if (dashboardTitle !== 'Dashboard') {
    throw new Error('Dashboard not loaded');
  }
});
```

### **2. Integrating Puppeteer with Step Definitions**

#### Setting Up Puppeteer in Step Definitions:

1. **Initialize Puppeteer in the Cucumber.js context:** Modify the `World` configuration to include Puppeteer:

   ```javascript
   const { setWorldConstructor } = require('@cucumber/cucumber');
   const puppeteer = require('puppeteer');

   class CustomWorld {
     async openBrowser() {
       this.browser = await puppeteer.launch({ headless: true });
       this.page = await this.browser.newPage();
     }

     async closeBrowser() {
       await this.browser.close();
     }
   }

   setWorldConstructor(CustomWorld);
   ```

2. **Use the **``** context in your step definitions:** The browser and page instances are now accessible via `this.browser` and `this.page`.

#### Common Puppeteer Commands in Step Definitions:

- Navigate to a URL:
  ```javascript
  await this.page.goto('https://example.com');
  ```
- Interact with elements:
  ```javascript
  await this.page.click('#button');
  await this.page.type('#inputField', 'text');
  ```
- Capture a screenshot:
  ```javascript
  await this.page.screenshot({ path: 'screenshot.png' });
  ```

### **3. Matching Gherkin Steps with Regex**

- Each step definition uses regex to match Gherkin steps.
- Example:
  ```gherkin
  Given the user enters "validUser" and "validPassword"
  ```
  Corresponding step definition:
  ```javascript
  Given(/^the user enters "([^"]*)" and "([^"]*)"$/, async function (username, password) {
    await this.page.type('#username', username);
    await this.page.type('#password', password);
  });
  ```

---

## Hands-On Exercises

### **Exercise 1: Create Basic Step Definitions**

1. Write a step definition to navigate to the login page:
   ```gherkin
   Given the user is on the login page
   ```
2. Implement the step in JavaScript:
   ```javascript
   Given('the user is on the login page', async function () {
     await this.page.goto('https://example.com/login');
   });
   ```

### **Exercise 2: Add Steps with Dynamic Data**

1. Update the `login.feature` file with dynamic data:
   ```gherkin
   Scenario: Login with dynamic credentials
     Given the user is on the login page
     When the user enters "{username}" and "{password}"
     Then the user should see "{message}"
   ```
2. Implement the dynamic step definition:
   ```javascript
   When('the user enters {string} and {string}', async function (username, password) {
     await this.page.type('#username', username);
     await this.page.type('#password', password);
     await this.page.click('#loginButton');
   });

   Then('the user should see {string}', async function (message) {
     const pageContent = await this.page.content();
     if (!pageContent.includes(message)) {
       throw new Error(`Expected message: ${message} not found`);
     }
   });
   ```

### **Exercise 3: Implement Error Handling in Steps**

1. Add error handling for element interaction:
   ```javascript
   When('the user clicks the login button', async function () {
     try {
       await this.page.click('#loginButton');
     } catch (error) {
       throw new Error('Login button not found: ' + error.message);
     }
   });
   ```

---

## Suggested Duration

3-4 hours

---

## Sources

- [Puppeteer API Documentation](https://pptr.dev/)
- [Cucumber.js Step Definitions Guide](https://cucumber.io/docs/gherkin/step-definitions/)
- YouTube: "Writing Step Definitions with Puppeteer" by Automation Step by Step.

