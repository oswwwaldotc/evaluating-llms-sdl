# Module 1: Introduction to Puppeteer and Cucumber.js 🌐

## Learning Objectives

- Understand the fundamentals of Puppeteer and Cucumber.js.
- Identify their use cases in browser automation and BDD.
- Set up a development environment for integrating Puppeteer with Cucumber.js.

---

## Key Topics

### **1. What is Puppeteer?**

Puppeteer is a Node.js library that provides a high-level API to control Chrome or Chromium browsers programmatically. It can be used for:

- **End-to-End Testing**: Automating UI tests for web applications.
- **Web Scraping**: Extracting data from websites efficiently.
- **Generating PDFs and Screenshots**: Creating visuals for reporting or documentation.
- **Performance Analysis**: Capturing page performance metrics.

### **2. What is Cucumber.js?**

Cucumber.js is a JavaScript framework for Behavior-Driven Development (BDD). It enables developers to:

- Write human-readable test cases in **Gherkin syntax**.
- Bridge the gap between technical and non-technical stakeholders by focusing on business requirements.
- Use a collaborative approach to define application behavior through **feature files**.

### **3. Why Use Puppeteer with Cucumber.js?**

Combining Puppeteer’s browser automation capabilities with Cucumber.js’s BDD approach allows teams to:

- Automate user scenarios described in natural language.
- Validate both business rules and application functionality.
- Create comprehensive and maintainable test frameworks.

---

## Environment Setup

### **Prerequisites**

Before starting, ensure you have the following:

- **Node.js and npm** installed ([Download here](https://nodejs.org/)).
- A code editor like **Visual Studio Code** ([Download here](https://code.visualstudio.com/)).

### **Steps to Set Up Puppeteer and Cucumber.js**

1. **Initialize a Node.js Project:** Open a terminal and create a new directory for your project:

   ```bash
   mkdir puppeteer-cucumber-demo
   cd puppeteer-cucumber-demo
   npm init -y
   ```

2. **Install Dependencies:** Install Puppeteer and Cucumber.js:

   ```bash
   npm install puppeteer @cucumber/cucumber
   ```

3. **Set Up Project Structure:** Create the following folders and files for a well-organized setup:

   ```bash
   mkdir features
   mkdir features/step_definitions
   touch features/example.feature
   touch cucumber.js
   ```

   - `features/`: Contains all your feature files written in Gherkin.
   - `features/step_definitions/`: Houses the JavaScript implementations of steps.
   - `example.feature`: A sample feature file to define test cases.
   - `cucumber.js`: A configuration file for Cucumber.js.

4. **Configure Cucumber.js:** Edit the `cucumber.js` file to specify the paths for feature files and step definitions:

   ```javascript
   module.exports = {
     default: `--require features/step_definitions/**/*.js`
   };
   ```

---

## Hands-On Exercises

### **Exercise 1: Launch a Browser with Puppeteer**

1. Create a `test.js` file and add the following code to launch a browser and navigate to a website:
   ```javascript
   const puppeteer = require('puppeteer');

   (async () => {
     const browser = await puppeteer.launch({ headless: false });
     const page = await browser.newPage();
     await page.goto('https://example.com');
     console.log(await page.title());
     await browser.close();
   })();
   ```
2. Run the file:
   ```bash
   node test.js
   ```

### **Exercise 2: Create a Basic Feature File**

1. Open `features/example.feature` and write a simple Gherkin scenario:
   ```gherkin
   Feature: Navigate to a website
     Scenario: Open the example page
       Given I launch the browser
       When I navigate to "https://example.com"
       Then I see the title "Example Domain"
   ```

### **Exercise 3: Implement Step Definitions**

1. Create a `features/step_definitions/example_steps.js` file and add step definitions:

   ```javascript
   const { Given, When, Then } = require('@cucumber/cucumber');
   const puppeteer = require('puppeteer');

   let browser, page;

   Given('I launch the browser', async () => {
     browser = await puppeteer.launch({ headless: false });
     page = await browser.newPage();
   });

   When('I navigate to {string}', async (url) => {
     await page.goto(url);
   });

   Then('I see the title {string}', async (title) => {
     const pageTitle = await page.title();
     if (pageTitle !== title) {
       throw new Error(`Expected title to be '${title}' but got '${pageTitle}'`);
     }
     await browser.close();
   });
   ```

2. Run the test:

   ```bash
   npx cucumber-js
   ```

---

## Suggested Duration

2-3 hours

---

## Sources

- [Puppeteer Documentation](https://pptr.dev/)
- [Cucumber.js Documentation](https://cucumber.io/docs/installation/javascript/)
- YouTube: "Getting Started with Puppeteer" by Fireship

