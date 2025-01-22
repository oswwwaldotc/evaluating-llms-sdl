# Module 4: Mapping Files 🔎

## Learning Objectives

- Understand the role and importance of mapping files in BDD.
- Configure `cucumber.js` to link feature files with step definitions.
- Organize your project structure for scalability and maintainability.

---

## Key Topics

### **1. The Role of Mapping Files**

Mapping files are critical for BDD frameworks as they:

- Link **Gherkin feature files** with their respective **step definitions**.
- Define where Cucumber.js looks for features, step definitions, and support files.
- Provide flexibility to handle complex project structures.

#### **Why Mapping Files Matter**

- Enable efficient debugging by organizing test logic.
- Ensure that test scenarios and implementation are in sync.
- Facilitate scalability for larger projects with multiple teams.

---

### **2. Configuring Cucumber.js with a Mapping File**

The configuration file in Cucumber.js is commonly named `cucumber.js`. It specifies paths and settings for running your tests.

#### **Basic Structure of **``

Here is a minimal configuration file:

```javascript
module.exports = {
  default: `--require-module @babel/register \
    --require ./step-definitions/**/*.js \
    --format progress \
    ./features/**/*.feature`,
};
```

#### **Key Directives**

- `--require-module`: Loads additional modules, such as Babel for ES6+ support.
- `--require`: Specifies paths to step definitions.
- `--format`: Defines the output format (e.g., progress, pretty, JSON).
- `./features/**/*.feature`: Points to the feature files directory.

---

### **3. Folder Structure Best Practices**

A well-structured project enhances readability and simplifies maintenance. Below is a suggested folder structure for a Puppeteer and Cucumber.js project:

```
project-root/
|-- features/
|   |-- login.feature
|   |-- search.feature
|
|-- step-definitions/
|   |-- login-steps.js
|   |-- search-steps.js
|
|-- support/
|   |-- world.js
|
|-- cucumber.js
|-- package.json
```

#### **Explanation**

- `features/`: Contains all the Gherkin feature files.
- `step-definitions/`: Includes JavaScript files implementing the steps.
- `support/`: Holds additional utilities or custom world definitions.
- `cucumber.js`: Configuration file linking everything.

---

### **4. Advanced Mapping with Tags and Profiles**

#### **Using Tags to Run Specific Scenarios**

Tags allow selective execution of scenarios or features. Add tags in your feature file:

```gherkin
@smoke
Feature: User Login
  Scenario: Valid Login
    Given the user is on the login page
    When the user enters valid credentials
    Then the user should see the dashboard
```

Run only tagged scenarios using the `--tags` flag:

```bash
npx cucumber-js --tags @smoke
```

#### **Creating Multiple Profiles**

Profiles allow different configurations for various environments (e.g., dev, staging, production).

Example `cucumber.js` with profiles:

```javascript
module.exports = {
  default: `--require ./step-definitions/**/*.js ./features/**/*.feature`,
  dev: `--require ./step-definitions/dev/**/*.js ./features/**/*.feature`,
  staging: `--require ./step-definitions/staging/**/*.js ./features/**/*.feature`,
};
```

Run a specific profile:

```bash
npx cucumber-js --profile dev
```

---

## Hands-On Exercises

### **Exercise 1: Basic Mapping File Configuration**

1. Create a `cucumber.js` file in your project root.
2. Configure it to:
   - Use the `./features/**/*.feature` directory for feature files.
   - Require all step definition files in `./step-definitions/**/*.js`.
3. Run a sample feature file to verify the setup.

### **Exercise 2: Tagging and Selective Execution**

1. Add tags like `@login` or `@smoke` to your feature file scenarios.
2. Execute specific tags using the `--tags` flag.
3. Verify that only tagged scenarios run.

### **Exercise 3: Profile-Based Configurations**

1. Create profiles for different environments in `cucumber.js`.
2. Write distinct step definitions for each environment (e.g., dev, staging).
3. Run tests for a specific environment using a profile.

---

## Suggested Duration

1-2 hours

---

## Sources

- [Cucumber.js Configuration Guide](https://cucumber.io/docs/guides/configuration/)
- Blog: "Cucumber.js Project Organization" by BrowserStack
- Video: "Setting Up Cucumber.js with Tags and Profiles" by Automation Step by Step

