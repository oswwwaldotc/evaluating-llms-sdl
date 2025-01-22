# Module 2: Writing Feature Files 🔬

## Learning Objectives
- Understand the structure and purpose of feature files in behavior-driven development (BDD).
- Write clear, concise, and human-readable scenarios using Gherkin syntax.
- Define acceptance criteria for test automation in a collaborative format.

---

## Key Topics

### **1. What are Feature Files?**
Feature files are plain-text documents written in Gherkin syntax that:

- Describe application behavior from a user's perspective.
- Serve as a bridge between non-technical stakeholders and developers/testers.
- Contain **Features**, **Scenarios**, and **Steps** that define what the application should do.

### **2. Gherkin Syntax Overview**
Gherkin is a domain-specific language used to describe BDD tests. Key elements include:

- **Feature:** A high-level description of the functionality being tested.
- **Scenario:** A specific use case or test case.
- **Steps:** Detailed instructions to execute the test, starting with keywords like `Given`, `When`, `Then`, `And`, or `But`.

#### Example Gherkin Syntax:
```gherkin
Feature: User login functionality

  Scenario: Successful login
    Given the user is on the login page
    When the user enters valid credentials
    Then the user should be redirected to the dashboard
```

### **3. Structuring Feature Files**

#### Best Practices:
- **One Feature Per File:** Keep feature files focused on a single feature.
- **Use Descriptive Titles:** Make it easy to understand the purpose of the feature.
- **Keep Scenarios Independent:** Avoid dependencies between scenarios.
- **Avoid Implementation Details:** Focus on the "what," not the "how."

#### Folder Structure:
Organize feature files systematically:
```
features/
  login/
    login.feature
  search/
    search.feature
```

---

## Hands-On Exercises

### **Exercise 1: Write Your First Feature File**
1. Create a folder named `features` in your project directory.
2. Inside `features`, create a file named `login.feature`.
3. Write the following Gherkin syntax:
   ```gherkin
   Feature: Login functionality

     Scenario: Successful login
       Given the user is on the login page
       When the user enters "validUser" and "validPassword"
       Then the user should see the dashboard

     Scenario: Failed login
       Given the user is on the login page
       When the user enters "invalidUser" and "invalidPassword"
       Then the user should see an error message
   ```

### **Exercise 2: Add Steps with Data Tables**
1. Extend the `login.feature` file to include data tables:
   ```gherkin
   Scenario Outline: Login with different credentials
     Given the user is on the login page
     When the user enters <username> and <password>
     Then the user should see <outcome>

     Examples:
       | username   | password       | outcome          |
       | validUser  | validPassword | the dashboard    |
       | invalidUser| invalidPassword| an error message |
   ```

2. Save the file and ensure it follows proper indentation.

---

## Suggested Duration
2-3 hours

---

## Sources
- [Cucumber Gherkin Reference](https://cucumber.io/docs/gherkin/reference/)
- Article: "How to Write Good Gherkin Scenarios" by SmartBear.
- YouTube: "BDD with Cucumber.js and Gherkin" by Automation Step by Step.

