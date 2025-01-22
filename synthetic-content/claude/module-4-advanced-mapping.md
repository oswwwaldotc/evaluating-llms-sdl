# Module 4: Advanced Mapping and Configuration

## Custom Parameter Types

Custom parameter types allow you to transform Gherkin step parameters into specific JavaScript objects. This enables more robust and maintainable step definitions.

```javascript
// support/parameter_types.js
const { defineParameterType } = require('@cucumber/cucumber');

defineParameterType({
    name: 'currency',
    regexp: /\$\d+\.?\d*/,
    transformer: amount => parseFloat(amount.replace('$', ''))
});

defineParameterType({
    name: 'date',
    regexp: /\d{4}-\d{2}-\d{2}/,
    transformer: dateString => new Date(dateString)
});

defineParameterType({
    name: 'user',
    regexp: /customer|admin|guest/,
    transformer: role => ({
        role,
        permissions: getPermissionsForRole(role),
        testData: loadTestDataForRole(role)
    })
});
```

## World Configuration

The World object provides shared context between steps. Here's an advanced implementation that includes environment management and utilities:

```javascript
// support/world.js
const { setWorldConstructor } = require('@cucumber/cucumber');
const puppeteer = require('puppeteer');
const config = require('../config');
const PageObjectManager = require('./page_object_manager');
const TestDataManager = require('./test_data_manager');

class CustomWorld {
    constructor({ attach, parameters }) {
        this.attach = attach;
        this.parameters = parameters;
        this.config = config[process.env.TEST_ENV || 'development'];
        this.testData = new TestDataManager(this.config);
    }

    async initialize() {
        this.browser = await puppeteer.launch(this.config.browserOptions);
        this.page = await this.browser.newPage();
        this.pageObjects = new PageObjectManager(this.page);
        
        // Set up request interception
        await this.page.setRequestInterception(true);
        this.page.on('request', request => {
            if (this.config.mockResponses[request.url()]) {
                request.respond(this.config.mockResponses[request.url()]);
            } else {
                request.continue();
            }
        });

        // Set up console error logging
        this.page.on('console', msg => {
            if (msg.type() === 'error') {
                this.attach(`Console Error: ${msg.text()}`);
            }
        });
    }

    async cleanup() {
        if (this.browser) {
            await this.browser.close();
        }
        await this.testData.cleanup();
    }
}

setWorldConstructor(CustomWorld);
```

## Environment Configuration

Create a robust environment configuration system:

```javascript
// config/index.js
const environmentConfigs = {
    development: {
        baseUrl: 'http://localhost:3000',
        apiBaseUrl: 'http://localhost:3001',
        browserOptions: {
            headless: false,
            slowMo: 50,
            defaultViewport: null
        },
        mockResponses: {}
    },
    staging: {
        baseUrl: 'https://staging.example.com',
        apiBaseUrl: 'https://api-staging.example.com',
        browserOptions: {
            headless: true,
            args: ['--no-sandbox']
        },
        mockResponses: {
            'https://api-staging.example.com/payment': {
                status: 200,
                body: JSON.stringify({ success: true })
            }
        }
    },
    production: {
        baseUrl: 'https://www.example.com',
        apiBaseUrl: 'https://api.example.com',
        browserOptions: {
            headless: true,
            args: ['--no-sandbox']
        },
        mockResponses: {}
    }
};

module.exports = environmentConfigs;
```

## Test Data Management

Implement a sophisticated test data management system:

```javascript
// support/test_data_manager.js
class TestDataManager {
    constructor(config) {
        this.config = config;
        this.testData = new Map();
    }

    async createTestData(type, data) {
        const response = await fetch(`${this.config.apiBaseUrl}/test-data/${type}`, {
            method: 'POST',
            body: JSON.stringify(data)
        });
        const result = await response.json();
        this.testData.set(result.id, { type, data: result });
        return result;
    }

    async getTestData(id) {
        return this.testData.get(id);
    }

    async cleanup() {
        for (const [id, { type }] of this.testData.entries()) {
            await fetch(`${this.config.apiBaseUrl}/test-data/${type}/${id}`, {
                method: 'DELETE'
            });
        }
        this.testData.clear();
    }
}

module.exports = TestDataManager;
```

## Page Object Manager

Create a centralized page object management system:

```javascript
// support/page_object_manager.js
const LoginPage = require('../page_objects/login_page');
const ProductPage = require('../page_objects/product_page');
const CheckoutPage = require('../page_objects/checkout_page');

class PageObjectManager {
    constructor(page) {
        this.page = page;
        this.pageObjects = new Map();
    }

    getPage(PageClass) {
        if (!this.pageObjects.has(PageClass)) {
            this.pageObjects.set(PageClass, new PageClass(this.page));
        }
        return this.pageObjects.get(PageClass);
    }

    get loginPage() {
        return this.getPage(LoginPage);
    }

    get productPage() {
        return this.getPage(ProductPage);
    }

    get checkoutPage() {
        return this.getPage(CheckoutPage);
    }
}

module.exports = PageObjectManager;
```

## Step Organization

Implement a step organization system that promotes reusability:

```javascript
// step_definitions/shared_steps.js
const { Given } = require('@cucumber/cucumber');

Given('I am logged in as a/an {user}', async function(user) {
    await this.pageObjects.loginPage.login(user.credentials);
    this.currentUser = user;
});

// step_definitions/product_steps.js
const { When, Then } = require('@cucumber/cucumber');

When('I view product with ID {string}', async function(productId) {
    const product = await this.testData.createTestData('product', {
        id: productId,
        name: 'Test Product',
        price: 99.99
    });
    await this.pageObjects.productPage.navigate(product.id);
});

// step_definitions/checkout_steps.js
const { When, Then } = require('@cucumber/cucumber');

When('I complete checkout with {currency} payment', async function(amount) {
    await this.pageObjects.checkoutPage.setPaymentAmount(amount);
    await this.pageObjects.checkoutPage.completePayment();
});
```

## Advanced Configuration Example

Here's a complete configuration setup that brings everything together:

```javascript
// cucumber.js
const common = {
    requireModule: ['ts-node/register'],
    require: [
        'support/**/*.js',
        'step_definitions/**/*.js'
    ],
    format: [
        'progress',
        'html:reports/cucumber-report.html',
        'json:reports/cucumber-report.json'
    ],
    formatOptions: {
        snippetInterface: 'async-await'
    },
    publishQuiet: true
};

module.exports = {
    default: {
        ...common,
        parallel: 2,
        tags: 'not @manual and not @skip',
    },
    smoke: {
        ...common,
        tags: '@smoke and not @manual',
    },
    regression: {
        ...common,
        tags: '@regression and not @manual',
    }
};
```

These configurations enable features like:
- Parallel test execution
- Environment-specific settings
- Custom parameter types
- Centralized page object management
- Automated test data cleanup
- Flexible test reporting
- Tag-based test execution

The module provides a foundation for creating maintainable, scalable test automation frameworks that can handle complex testing scenarios while remaining organized and efficient.

## Exercise: Framework Enhancement

Create a complete test automation framework that demonstrates:
1. Custom parameter types for complex data structures
2. Environment-specific configuration management
3. Automated test data handling
4. Centralized page object management
5. Shared step definition implementation

This exercise will help reinforce the concepts covered while providing practical experience with advanced framework configuration.
