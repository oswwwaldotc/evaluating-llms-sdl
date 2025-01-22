# Module 5: Advanced Topics 🚀

## Learning Objectives

- Learn effective debugging techniques for Puppeteer and Cucumber.js.
- Apply best practices to maintain scalable and reusable test frameworks.
- Integrate Puppeteer and Cucumber.js with CI/CD pipelines for automated testing.

---

## Key Topics

### **1. Debugging Puppeteer and Cucumber.js**

#### **Using Debugging Tools in Puppeteer**

1. **Headed Mode:**

   - By default, Puppeteer runs in headless mode. Use the `headless: false` option to see the browser interactions:
     ```javascript
     const browser = await puppeteer.launch({ headless: false });
     ```

2. **SlowMo:**

   - Add a delay between actions to observe the steps clearly:
     ```javascript
     const browser = await puppeteer.launch({ headless: false, slowMo: 50 });
     ```

3. **DevTools Protocol:**

   - Enable DevTools to inspect the browser during execution:
     ```javascript
     const browser = await puppeteer.launch({ headless: false, devtools: true });
     ```

4. **Console Logs in Tests:**

   - Log step execution to identify where errors occur:
     ```javascript
     console.log('Navigating to login page');
     await page.goto('https://example.com');
     ```

#### **Debugging Step Definitions in Cucumber.js**

- Use `console.log()` to track variable values.
- Run tests with the `--format` option for verbose output:
  ```bash
  npx cucumber-js --format progress-bar
  ```
- Add a `debugger` statement in your step definition to pause execution and inspect:
  ```javascript
  Given('I debug the step', async function () {
    debugger;
  });
  ```

---

### **2. Best Practices for Scalable Test Frameworks**

#### **Organizing Tests and Code**

1. **Folder Structure:**

   - Group tests by feature or functionality:
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
     |-- utils/
     |   |-- helper.js
     ```

2. **Reusable Utilities:**

   - Create helper functions for repetitive actions, like logging in or clicking buttons:
     ```javascript
     module.exports = {
       async login(page, username, password) {
         await page.type('#username', username);
         await page.type('#password', password);
         await page.click('#loginButton');
       },
     };
     ```

#### **Naming Conventions**

- Use consistent and descriptive names for step definitions and feature files.
- Prefix utility functions with their purpose (e.g., `getElementText`, `navigateToPage`).

#### **Version Control**

- Use Git to track changes and collaborate effectively.
- Commit often with meaningful messages (e.g., `git commit -m "Add login step definitions"`).

---

### **3. Integrating with CI/CD Pipelines**

#### **Setting Up CI/CD for Automated Testing**

1. **Choosing a CI/CD Tool:**

   - Popular tools include GitHub Actions, GitLab CI, CircleCI, and Jenkins.

2. **Creating a Workflow File (GitHub Actions Example):**

   ```yaml
   name: Puppeteer Cucumber Tests

   on:
     push:
       branches:
         - main

   jobs:
     test:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout code
           uses: actions/checkout@v3

         - name: Set up Node.js
           uses: actions/setup-node@v3
           with:
             node-version: '18'

         - name: Install dependencies
           run: npm install

         - name: Run tests
           run: npx cucumber-js
   ```

3. **Running Tests in Docker:**

   - Create a `Dockerfile` to containerize your tests:

     ```dockerfile
     FROM node:18

     WORKDIR /app

     COPY package*.json ./
     RUN npm install

     COPY . ./

     CMD ["npx", "cucumber-js"]
     ```

   - Build and run the Docker container:

     ```bash
     docker build -t puppeteer-tests .
     docker run puppeteer-tests
     ```

---

## Hands-On Exercises

### **Exercise 1: Debugging Browser Interactions**

1. Modify your Puppeteer launch options to enable `headless: false` and `slowMo: 100`.
2. Use `console.log()` to print step details.
3. Add a `debugger` statement in a step definition and observe execution.

### **Exercise 2: Refactor to Use Utilities**

1. Move repetitive code into helper functions.
2. Refactor existing step definitions to use these helpers.
3. Verify that all tests still pass.

### **Exercise 3: Set Up CI/CD Pipeline**

1. Create a CI workflow file for GitHub Actions.
2. Run the pipeline and fix any configuration issues.
3. Experiment with adding tags or profiles to run selective tests in CI.

---

## Suggested Duration

3-5 hours

---

## Sources

- [Puppeteer Debugging Guide](https://pptr.dev/)
- [Cucumber.js Best Practices](https://cucumber.io/docs/guides/best-practices/)
- YouTube: "CI/CD with Puppeteer and Cucumber.js" by Automation Step by Step

