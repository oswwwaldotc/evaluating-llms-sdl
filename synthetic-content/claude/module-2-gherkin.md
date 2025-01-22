# Module 2: Writing Feature Files with Gherkin

## Understanding Gherkin Fundamentals

Gherkin is a Business Readable, Domain-Specific Language that describes software behavior without detailing how that behavior is implemented. It serves as a common language between technical and non-technical team members.

### Core Gherkin Syntax

A well-structured feature file contains several key elements:

```gherkin
@tag
Feature: Account Management
  As a bank customer
  I want to manage my account online
  So that I can handle my finances conveniently

  Background:
    Given I am logged into my banking account
    And I am on the account dashboard

  @critical
  Scenario: View account balance
    When I click on the "Account Summary" button
    Then I should see my current balance
    And I should see my recent transactions

  @regression
  Scenario Outline: Transfer money between accounts
    When I initiate a transfer of <amount> from <source> to <target>
    Then the transfer should be successful
    And the <source> balance should decrease by <amount>
    And the <target> balance should increase by <amount>

    Examples:
      | amount | source        | target        |
      | 100    | Checking     | Savings       |
      | 500    | Savings      | Checking      |
      | 1000   | Investment   | Checking      |
```

### Writing Effective Features

Consider this complete example of a shopping cart feature:

```gherkin
@shopping
Feature: Shopping Cart Management
  As an online shopper
  I want to manage items in my shopping cart
  So that I can purchase my desired products

  Background:
    Given I am logged into my account
    And my shopping cart is empty

  Scenario: Add item to cart
    Given I am viewing a product "Wireless Mouse"
    When I click "Add to Cart"
    Then the item "Wireless Mouse" should be in my cart
    And the cart total should be updated
    And I should see a confirmation message

  Scenario: Update item quantity
    Given I have "Wireless Mouse" in my cart
    When I update the quantity to 2
    Then the cart should show 2 "Wireless Mouse" items
    And the cart total should reflect the updated quantity

  Scenario Outline: Apply discount codes
    Given I have items worth <initial_amount> in my cart
    When I apply discount code "<code>"
    Then I should see a discount of <discount_amount>
    And my final total should be <final_amount>

    Examples:
      | initial_amount | code      | discount_amount | final_amount |
      | 100.00        | SAVE10    | 10.00          | 90.00        |
      | 200.00        | SPECIAL20 | 40.00          | 160.00       |
      | 50.00         | FLASH15   | 7.50           | 42.50        |
```

### Advanced Gherkin Features

Data Tables for Complex Input:

```gherkin
Scenario: Place order with multiple items
  Given I have the following items in my cart:
    | Product         | Quantity | Price |
    | Wireless Mouse  | 2        | 29.99 |
    | USB Cable      | 3        | 9.99  |
    | Laptop Stand   | 1        | 49.99 |
  When I proceed to checkout
  Then my order summary should show the correct total
  And shipping should be calculated based on the total items
```

Doc Strings for Large Text Content:

```gherkin
Scenario: Submit product review
  Given I am on the product review page
  When I submit the following review:
    """
    This wireless mouse exceeded my expectations. The battery life
    is excellent, lasting over three months on a single charge.
    The ergonomic design makes it comfortable for all-day use,
    and the customizable buttons are a great productivity feature.
    
    The only minor drawback is that the USB receiver is a bit large,
    but this doesn't impact the overall performance.
    """
  Then my review should be submitted successfully
  And I should see it displayed on the product page
```

### Best Practices for Feature Files

1. Feature Organization:
```gherkin
Feature: User Authentication
  # Group related scenarios
  @login @critical
  Scenario: Standard user login
    # Login scenario details

  @login @edge-case
  Scenario: Login with special characters
    # Special case login scenario

  @password @security
  Scenario: Password reset
    # Password reset scenario
```

2. Background Usage:
```gherkin
Feature: Order Management

  Background:
    Given I am logged in as a verified customer
    And I have active payment methods
    And I have items in my cart

  Scenario: Express checkout
    When I select express checkout
    Then my order should be placed immediately

  Scenario: Standard checkout
    When I proceed with standard checkout
    Then I should be able to review my order
```

3. Scenario Outline Patterns:
```gherkin
Feature: Product Search

  Scenario Outline: Search with different criteria
    Given I am on the search page
    When I search for products with the following criteria:
      | category   | price_range   | rating   |
      | <category> | <price_range> | <rating> |
    Then I should see products matching all criteria
    And the results should be sorted by relevance

    Examples:
      | category    | price_range | rating |
      | Electronics | 100-500     | 4+     |
      | Books       | 10-50       | 3+     |
      | Clothing    | 25-100      | 4.5+   |
```

### Common Pitfalls and Solutions

Incorrect:
```gherkin
Scenario: Technical implementation details
  Given the database connection is established
  When the SQL query "SELECT * FROM users" is executed
  Then the result set should contain user records
```

Correct:
```gherkin
Scenario: View user list
  Given I am logged in as an administrator
  When I navigate to the user management page
  Then I should see a list of all active users
```

### Exercise: Feature File Development

Create a feature file for an email system with the following requirements:
1. Compose and send emails
2. Manage draft emails
3. Handle attachments
4. Apply filters and labels

The solution should demonstrate:
- Proper use of tags
- Background steps
- Scenario Outlines
- Data Tables
- Doc Strings

This exercise will help reinforce the concepts covered in this module while providing practical experience with Gherkin syntax.
