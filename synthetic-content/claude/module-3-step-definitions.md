# Module 3: Step Definitions and Puppeteer Integration

## Understanding Step Definitions

Step definitions connect Gherkin statements with JavaScript code that interacts with the browser through Puppeteer. The implementation requires careful consideration of asynchronous operations and proper error handling.

## Implementation Framework

Let's create a robust implementation framework for an e-commerce website testing suite. First, we'll establish our base setup:

```javascript
// step_definitions/common.js
const { Given, When, Then } = require('@cucumber/cucumber');
const { expect } = require('chai');
const puppeteer = require('puppeteer');

// Utility function for error handling
async function retry(action, maxAttempts = 3) {
    let lastError;
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            return await action();
        } catch (error) {
            lastError = error;
            if (attempt === maxAttempts) throw error;
            await new Promise(resolve => setTimeout(resolve, 1000));
        }
    }
}
```

## Page Object Implementation

We'll create a structured page object model to organize our selectors and actions:

```javascript
// page_objects/ProductPage.js
class ProductPage {
    constructor(page) {
        this.page = page;
        this.selectors = {
            productTitle: '.product-title',
            addToCartButton: '#add-to-cart',
            quantityInput: '#quantity',
            priceDisplay: '.price',
            cartCount: '#cart-count',
            errorMessage: '.error-message'
        };
    }

    async addToCart(quantity = 1) {
        await this.page.type(this.selectors.quantityInput, quantity.toString());
        await this.page.click(this.selectors.addToCartButton);
        await this.page.waitForSelector(this.selectors.cartCount);
    }

    async getPrice() {
        const priceElement = await this.page.$(this.selectors.priceDisplay);
        const priceText = await priceElement.evaluate(el => el.textContent);
        return parseFloat(priceText.replace(/[^0-9.-]+/g, ''));
    }
}

module.exports = ProductPage;
```

## Step Definition Implementation

Now let's implement comprehensive step definitions that utilize our page objects:

```javascript
// step_definitions/product_steps.js
const { Given, When, Then } = require('@cucumber/cucumber');
const ProductPage = require('../page_objects/ProductPage');

Given('I am viewing product {string}', async function(productUrl) {
    this.productPage = new ProductPage(this.page);
    await retry(async () => {
        await this.page.goto(`${process.env.BASE_URL}/products/${productUrl}`, {
            waitUntil: 'networkidle0'
        });
    });
});

When('I add {int} items to the cart', async function(quantity) {
    await retry(async () => {
        await this.productPage.addToCart(quantity);
    });
});

Then('the cart should display {int} items', async function(expectedCount) {
    await retry(async () => {
        const cartCount = await this.page.$eval(
            this.productPage.selectors.cartCount,
            el => parseInt(el.textContent)
        );
        expect(cartCount).to.equal(expectedCount);
    });
});
```

## Advanced Puppeteer Integration

### Handling Dynamic Content

```javascript
// step_definitions/dynamic_content.js
When('I wait for dynamic content to load', async function() {
    await this.page.waitForFunction(
        () => !document.querySelector('.loading-spinner'),
        { timeout: 5000 }
    );
});

When('I scroll to load more items', async function() {
    await this.page.evaluate(() => {
        window.scrollBy(0, window.innerHeight);
    });
    
    await this.page.waitForFunction(
        () => {
            const items = document.querySelectorAll('.product-item');
            return items.length > previousCount;
        },
        { timeout: 5000 }
    );
});
```

### Form Interaction and Validation

```javascript
// step_definitions/form_steps.js
When('I fill in the following form details:', async function(dataTable) {
    const formData = dataTable.hashes()[0];
    
    for (const [field, value] of Object.entries(formData)) {
        const selector = `input[name="${field}"]`;
        await this.page.type(selector, value);
    }
});

Then('I should see form validation errors', async function() {
    const errorMessages = await this.page.$$eval(
        '.error-message',
        elements => elements.map(el => el.textContent)
    );
    expect(errorMessages.length).to.be.greaterThan(0);
});
```

### Network Request Handling

```javascript
// step_definitions/network_steps.js
When('I submit the form', async function() {
    const [response] = await Promise.all([
        this.page.waitForResponse(
            response => response.url().includes('/api/submit')
        ),
        this.page.click('button[type="submit"]')
    ]);
    
    expect(response.status()).to.equal(200);
});
```

### File Upload Handling

```javascript
// step_definitions/upload_steps.js
When('I upload file {string}', async function(filename) {
    const fileInput = await this.page.$('input[type="file"]');
    await fileInput.uploadFile(`./test-files/${filename}`);
    
    await this.page.waitForSelector('.upload-success');
});
```

## Error Handling and Screenshots

```javascript
// support/hooks.js
const { After, Status } = require('@cucumber/cucumber');

After(async function(scenario) {
    if (scenario.result.status === Status.FAILED) {
        const screenshot = await this.page.screenshot({
            fullPage: true
        });
        this.attach(screenshot, 'image/png');
    }
});
```

## Performance Optimization

```javascript
// step_definitions/performance_steps.js
Then('the page should load within {int} seconds', async function(seconds) {
    const metrics = await this.page.metrics();
    const loadTime = metrics.TaskDuration / 1000;
    expect(loadTime).to.be.below(seconds);
});
```

## Practical Example: Complete E-commerce Flow

```javascript
// step_definitions/checkout_steps.js
Given('I have the following items in my cart:', async function(dataTable) {
    const items = dataTable.hashes();
    for (const item of items) {
        await retry(async () => {
            await this.page.goto(`${process.env.BASE_URL}/products/${item.productId}`);
            await this.productPage.addToCart(parseInt(item.quantity));
        });
    }
});

When('I proceed to checkout', async function() {
    await retry(async () => {
        await this.page.click('#checkout-button');
        await this.page.waitForSelector('#checkout-form');
    });
});

Then('the order summary should show correct totals', async function() {
    const subtotal = await this.page.$eval(
        '.subtotal',
        el => parseFloat(el.textContent.replace(/[^0-9.-]+/g, ''))
    );
    const tax = await this.page.$eval(
        '.tax',
        el => parseFloat(el.textContent.replace(/[^0-9.-]+/g, ''))
    );
    const total = await this.page.$eval(
        '.total',
        el => parseFloat(el.textContent.replace(/[^0-9.-]+/g, ''))
    );
    
    expect(total).to.equal(subtotal + tax);
});
```

## Exercise: Building a Test Suite

Create step definitions for a user authentication system that includes:
1. Registration with email verification
2. Login with two-factor authentication
3. Password reset functionality
4. Account settings management

The implementation should demonstrate:
- Proper page object implementation
- Error handling and retry logic
- Screenshot capture on failure
- Network request interception
- Form validation handling

This exercise will help reinforce the concepts covered while providing practical experience with Puppeteer integration.
