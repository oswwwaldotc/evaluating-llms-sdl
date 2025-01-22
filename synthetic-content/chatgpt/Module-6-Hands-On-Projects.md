# Module 6: Hands-On Projects 📚

## Learning Objectives

- Apply the concepts learned from modules 1-5 to real-world scenarios.
- Gain practical experience in browser automation and behavior-driven development (BDD).
- Build and demonstrate a robust Puppeteer and Cucumber.js testing framework.

---

## Project 1: Login Automation Framework

### **Scenario**: Automate the login process for a web application.

#### **Learning Objectives:**
- Use Puppeteer to interact with a login form.
- Write Gherkin scenarios to handle successful and unsuccessful login attempts.
- Create reusable step definitions and utilities.

#### **Steps:**

1. **Write Feature File:**
   ```gherkin
   Feature: Login functionality

   Scenario: Successful login
     Given the user is on the login page
     When the user enters valid credentials
     Then the user should see the dashboard

   Scenario: Unsuccessful login
     Given the user is on the login page
     When the user enters invalid credentials
     Then the user should see an error message
   ```

2. **Implement Step Definitions:**
   - Navigate to the login page.
   - Use Puppeteer to fill in the username and password fields.
   - Verify the success or error message.

3. **Organize Utilities:**
   - Create a `utils/loginHelper.js` to handle common login actions.

4. **Run Tests:**
   - Use `npx cucumber-js` to execute the tests.

#### **Deliverable:**
A working login automation framework with reusable components.

---

## Project 2: E-Commerce Search and Filter Testing

### **Scenario**: Test the search and filter functionality of an e-commerce website.

#### **Learning Objectives:**
- Test dynamic UI interactions using Puppeteer.
- Write comprehensive Gherkin scenarios for multiple test cases.

#### **Steps:**

1. **Write Feature File:**
   ```gherkin
   Feature: Search and filter functionality

   Scenario: Search for a product
     Given the user is on the homepage
     When the user searches for "laptop"
     Then search results should display items related to "laptop"

   Scenario: Filter results
     Given the user is on the search results page
     When the user applies the "Price: Low to High" filter
     Then the results should be sorted by price in ascending order
   ```

2. **Implement Step Definitions:**
   - Automate the search functionality.
   - Verify that the search results are accurate.
   - Apply a filter and check the order of results.

3. **Organize Folder Structure:**
   - Create `features/search.feature` and `step-definitions/search-steps.js`.

#### **Deliverable:**
A fully functional test suite for search and filter operations.

---

## Project 3: Automated Form Submission and Validation

### **Scenario**: Validate form submission with dynamic field inputs.

#### **Learning Objectives:**
- Handle forms with Puppeteer (e.g., input fields, dropdowns, checkboxes).
- Validate success and error messages based on inputs.

#### **Steps:**

1. **Write Feature File:**
   ```gherkin
   Feature: Form submission

   Scenario: Submit valid form data
     Given the user is on the form page
     When the user fills out the form with valid data
     And submits the form
     Then the user should see a success message

   Scenario: Submit invalid form data
     Given the user is on the form page
     When the user fills out the form with invalid data
     And submits the form
     Then the user should see an error message
   ```

2. **Implement Step Definitions:**
   - Automate filling out the form fields.
   - Verify form validation logic.

3. **Dynamic Data Handling:**
   - Use test data from a `.json` or `.csv` file to cover multiple scenarios.

#### **Deliverable:**
A reusable form submission test framework.

---

## Project 4: Integration with CI/CD Pipelines

### **Scenario**: Integrate the test framework with a CI/CD tool.

#### **Learning Objectives:**
- Configure a CI/CD pipeline to run automated tests.
- Use Docker to containerize the Puppeteer and Cucumber.js framework.

#### **Steps:**

1. **Create a Dockerfile:**
   ```dockerfile
   FROM node:18

   WORKDIR /app

   COPY package*.json ./
   RUN npm install

   COPY . ./

   CMD ["npx", "cucumber-js"]
   ```

2. **Set Up GitHub Actions Workflow:**
   ```yaml
   name: CI for Puppeteer Tests

   on:
     push:
       branches:
         - main

   jobs:
     build-and-test:
       runs-on: ubuntu-latest

       steps:
         - name: Checkout code
           uses: actions/checkout@v3

         - name: Set up Node.js
           uses: actions/setup-node@v3
           with:
             node-version: 18

         - name: Install dependencies
           run: npm install

         - name: Run tests
           run: npx cucumber-js
   ```

3. **Trigger the Workflow:**
   - Push changes to the repository and observe the test results.

#### **Deliverable:**
A CI/CD pipeline capable of running the test framework automatically.

---

## Suggested Duration

- **Project 1:** 4-5 hours
- **Project 2:** 5-6 hours
- **Project 3:** 4-5 hours
- **Project 4:** 6-8 hours

---

## Sources

- [Puppeteer API Documentation](https://pptr.dev/)
- [Cucumber.js Official Guide](https://cucumber.io/docs/)
- YouTube: "End-to-End Testing Projects with Puppeteer and Cucumber.js" by Automation Step by Step

