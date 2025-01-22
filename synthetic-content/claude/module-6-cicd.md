# Module 6: CI/CD Integration and Advanced Topics

## CI/CD Integration Fundamentals

When integrating Puppeteer and Cucumber.js tests into a CI/CD pipeline, proper configuration and infrastructure setup are essential for reliable test execution. The following sections outline comprehensive implementation approaches for various CI/CD platforms.

### GitHub Actions Implementation

```yaml
# .github/workflows/e2e-tests.yml
name: E2E Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16.x, 18.x]
        
    steps:
      - uses: actions/checkout@v2
      
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v2
        with:
          node-version: ${{ matrix.node-version }}
          
      - name: Install Dependencies
        run: npm install
        
      - name: Install Chrome
        run: |
          wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
          sudo apt install ./google-chrome-stable_current_amd64.deb
          
      - name: Run E2E Tests
        run: npm run test:e2e
        env:
          CI: true
          TEST_ENV: staging
          
      - name: Upload Test Results
        if: always()
        uses: actions/upload-artifact@v2
        with:
          name: test-results
          path: |
            reports/
            screenshots/
            videos/
```

### Jenkins Pipeline Configuration

```groovy
// Jenkinsfile
pipeline {
    agent {
        docker {
            image 'node:18-slim'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    
    environment {
        TEST_ENV = 'staging'
        CHROME_BIN = '/usr/bin/google-chrome'
    }
    
    stages {
        stage('Setup') {
            steps {
                sh 'npm install'
                sh '''
                    apt-get update
                    apt-get install -y wget gnupg
                    wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
                    echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list
                    apt-get update
                    apt-get install -y google-chrome-stable
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm run test:e2e'
            }
            post {
                always {
                    junit 'reports/**/*.xml'
                    archiveArtifacts artifacts: 'reports/**/*', allowEmptyArchive: true
                }
            }
        }
    }
}
```

## Docker Configuration

```dockerfile
# Dockerfile
FROM node:18-slim

# Install Chrome dependencies
RUN apt-get update && apt-get install -y \
    wget \
    gnupg \
    && wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list \
    && apt-get update \
    && apt-get install -y \
    google-chrome-stable \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

CMD ["npm", "run", "test:e2e"]
```

## Cross-Browser Testing Configuration

```javascript
// config/browsers.js
const browsersConfig = {
    chrome: {
        browserName: 'chrome',
        headless: true,
        defaultViewport: null,
        args: [
            '--no-sandbox',
            '--disable-setuid-sandbox',
            '--disable-dev-shm-usage'
        ]
    },
    firefox: {
        browserName: 'firefox',
        headless: true,
        defaultViewport: null,
        args: ['--no-sandbox']
    },
    webkit: {
        browserName: 'webkit',
        headless: true,
        defaultViewport: null
    }
};

module.exports = browsersConfig;
```

## Parallel Test Execution

```javascript
// cucumber.conf.js
const os = require('os');

module.exports = {
    default: {
        parallel: os.cpus().length,
        format: ['json:reports/cucumber-report.json'],
        formatOptions: {
            snippetInterface: 'async-await'
        },
        requireModule: ['ts-node/register'],
        require: ['features/step_definitions/*.js', 'features/support/*.js'],
        paths: ['features/**/*.feature'],
        tags: 'not @skip'
    }
};
```

## Advanced Reporting Integration

```javascript
// support/reporter.js
class AdvancedReporter {
    constructor() {
        this.results = {
            scenarios: [],
            metrics: {
                total: 0,
                passed: 0,
                failed: 0,
                skipped: 0
            },
            performance: {
                duration: 0,
                averageScenarioDuration: 0
            }
        };
    }

    onScenarioComplete(scenario, result) {
        const scenarioResult = {
            name: scenario.name,
            status: result.status,
            duration: result.duration,
            errorMessage: result.exception?.message,
            steps: result.steps.map(step => ({
                text: step.text,
                status: step.status,
                duration: step.duration
            }))
        };

        this.results.scenarios.push(scenarioResult);
        this.updateMetrics(result);
    }

    updateMetrics(result) {
        this.results.metrics.total++;
        this.results.metrics[result.status.toLowerCase()]++;
        this.results.performance.duration += result.duration;
    }

    generateReport() {
        this.results.performance.averageScenarioDuration = 
            this.results.performance.duration / this.results.metrics.total;

        return {
            summary: this.results.metrics,
            performance: this.results.performance,
            details: this.results.scenarios
        };
    }
}
```

## Security Testing Integration

```javascript
// support/security_tests.js
class SecurityTestHelper {
    constructor(page) {
        this.page = page;
    }

    async checkXSSVulnerabilities(selector, input) {
        await this.page.type(selector, input);
        const injectedScript = await this.page.evaluate(() => {
            return document.querySelector('script[data-injected]') !== null;
        });
        return !injectedScript;
    }

    async checkCSRFProtection(endpoint, method, data) {
        const response = await this.page.evaluate(async (url, method, data) => {
            const response = await fetch(url, {
                method,
                body: JSON.stringify(data),
                headers: {
                    'Content-Type': 'application/json'
                }
            });
            return response.status;
        }, endpoint, method, data);
        return response === 403;
    }
}
```

## Load Testing Configuration

```javascript
// support/load_test.js
class LoadTestHelper {
    constructor(page, config) {
        this.page = page;
        this.config = config;
    }

    async simulateUserLoad(scenarios, concurrentUsers) {
        const results = [];
        const chunks = this.createUserChunks(scenarios, concurrentUsers);

        for (const chunk of chunks) {
            const chunkResults = await Promise.all(
                chunk.map(scenario => this.executeScenario(scenario))
            );
            results.push(...chunkResults);
        }

        return this.analyzeResults(results);
    }

    async executeScenario(scenario) {
        const startTime = Date.now();
        let success = true;
        let error = null;

        try {
            await scenario(this.page);
        } catch (e) {
            success = false;
            error = e.message;
        }

        return {
            duration: Date.now() - startTime,
            success,
            error
        };
    }

    analyzeResults(results) {
        return {
            totalExecutions: results.length,
            successRate: results.filter(r => r.success).length / results.length,
            averageDuration: results.reduce((acc, r) => acc + r.duration, 0) / results.length,
            errors: results.filter(r => !r.success).map(r => r.error)
        };
    }
}
```

## Exercise: CI/CD Pipeline Implementation

Create a complete CI/CD pipeline that demonstrates:

1. Automated test execution in multiple environments
2. Cross-browser testing implementation
3. Parallel test execution configuration
4. Advanced reporting integration
5. Security and load testing incorporation

This exercise will help reinforce the concepts covered while providing practical experience with CI/CD integration and advanced testing techniques.

The implementation should focus on:

- Environment-specific configurations
- Test parallelization strategies
- Report generation and analysis
- Security testing integration
- Performance monitoring and optimization
- Cross-browser compatibility
- Error handling and recovery
- Resource cleanup and management

