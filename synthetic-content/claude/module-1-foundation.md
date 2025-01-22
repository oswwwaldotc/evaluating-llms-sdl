# Module 1: Foundation - Getting Started with Puppeteer and Cucumber.js

## Prerequisites
Before beginning this module, ensure you have:
- Node.js (version 14 or higher) installed
- A code editor (VS Code recommended)
- Basic JavaScript knowledge
- Understanding of npm package management

## Project Setup

### Step 1: Initialize Your Project
First, create a new directory and initialize a Node.js project:

```bash
mkdir puppeteer-cucumber-project
cd puppeteer-cucumber-project
npm init -y
```

### Step 2: Install Dependencies
Install the required packages:

```bash
npm install --save-dev @cucumber/cucumber puppeteer
npm install --save-dev chai
```

### Step 3: Project Structure
Create the following directory structure:

```plaintext
puppeteer-cucumber-project/
├── features/
│   ├── step_definitions/
│   │   └── search_steps.js
│   ├── support/
│   │   ├── world.js
│   │   └── hooks.js
│   └── search.feature
├── package.json
└── cucumber.js
```

### Step 4: Configuration Files

Create a `cucumber.js` configuration file:

```javascript
module.exports = {
  default: {
    requireModule: ['ts-node/register'],
    require: ['features/step_definitions/*.js', 'features/support/*.js'],
    format: ['progress', 'html:reports/cucumber-report.html'],
    formatOptions: { snippetInterface: 'async-await' },
    publishQuiet: true
  }
};
```

Update your `package.json` with test scripts:

```json
{
  "scripts": {
    "test": "cucumber-js",
    "test:report": "cucumber-js --format html:reports/cucumber-report.html"
  }
}
```

## Initial Implementation

### Step 1: Create Your First Feature File
Create `features/search.feature`:

```gherkin
Feature: Search Functionality
  As a user
  I want to search for content
  So that I can find relevant information

  Scenario: Basic search
    Given I am on the search page
    When I enter "automated testing" into the search box
    And I click the search button
    Then I should see search results
```

### Step 2: Implement World Configuration
Create `features/support/world.js`:

```javascript
const { setWorldConstructor } = require('@cucumber/cucumber');
const puppeteer = require('puppeteer');

class CustomWorld {
  async init() {
    this.browser = await puppeteer.launch({
      headless: false,
      slowMo: 50,
      defaultViewport: null
    });
    this.page = await this.browser.newPage();
  }

  async cleanup() {
    if (this.browser) {
      await this.browser.close();
    }
  }
}

setWorldConstructor(CustomWorld);
```

### Step 3: Implement Hooks
Create `features/support/hooks.js`:

```javascript
const { Before, After } = require('@cucumber/cucumber');

Before(async function() {
  await this.init();
});

After(async function() {
  await this.cleanup();
});
```

### Step 4: Implement Step Definitions
Create `features/step_definitions/search_steps.js`:

```javascript
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('chai');

Given('I am on the search page', async function() {
  await this.page.goto('https://www.example.com/search');
});

When('I enter {string} into the search box', async function(searchTerm) {
  await this.page.type('#search-input', searchTerm);
});

When('I click the search button', async function() {
  await this.page.click('#search-button');
});

Then('I should see search results', async function() {
  await this.page.waitForSelector('.search-results');
  const results = await this.page.$$('.search-results');
  expect(results.length).to.be.greaterThan(0);
});
```

## Understanding the Implementation

### Key Components

1. **Feature File**: Describes the test scenario in human-readable format using Gherkin syntax.

2. **World Configuration**: Provides a shared context for test execution, including:
   - Browser initialization
   - Page object management
   - Common utilities and helpers

3. **Hooks**: Handle setup and teardown operations:
   - Before: Initialize browser and page
   - After: Cleanup resources

4. **Step Definitions**: Implement the actual test logic using Puppeteer commands.

### Important Concepts

1. **Browser Configuration**:
   - `headless: false`: Shows the browser during test execution
   - `slowMo: 50`: Adds delay between actions for visibility
   - `defaultViewport: null`: Uses default viewport settings

2. **Page Interactions**:
   - Navigation: `page.goto()`
   - Element selection: `page.$()`
   - Typing: `page.type()`
   - Clicking: `page.click()`
   - Waiting: `page.waitForSelector()`

3. **Assertions**:
   - Using Chai for assertions
   - Verifying element presence
   - Checking element content

## Best Practices

1. **Error Handling**:
```javascript
When('I perform an action', async function() {
  try {
    await this.page.click('#button');
  } catch (error) {
    throw new Error(`Failed to click button: ${error.message}`);
  }
});
```

2. **Waiting Strategies**:
```javascript
// Wait for element
await this.page.waitForSelector('#element', { visible: true });

// Wait for network idle
await this.page.waitForNavigation({ waitUntil: 'networkidle0' });
```

3. **Selector Management**:
```javascript
// Create a selectors object
const selectors = {
  searchInput: '#search-input',
  searchButton: '#search-button',
  results: '.search-results'
};

// Use in step definitions
await this.page.type(selectors.searchInput, searchTerm);
```

## Exercise: Your First Test

1. Create a new feature file for testing a login form
2. Implement step definitions for:
   - Navigating to the login page
   - Entering credentials
   - Submitting the form
   - Verifying successful login
3. Run the tests using different configuration options

This exercise will help reinforce the concepts covered in this module while providing hands-on experience with the tools.
