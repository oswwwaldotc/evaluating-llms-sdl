# Module 5: Debugging and Testing Best Practices

## Advanced Debugging Techniques

### Console Output Enhancement

Implement sophisticated console logging to capture detailed test execution information:

```javascript
// support/debug_helper.js
class DebugHelper {
    constructor(page) {
        this.page = page;
        this.logs = [];
        this.setupConsoleListener();
        this.setupNetworkListener();
    }

    setupConsoleListener() {
        this.page.on('console', message => {
            const logEntry = {
                type: message.type(),
                text: message.text(),
                timestamp: new Date().toISOString(),
                location: message.location()
            };
            this.logs.push(logEntry);
            
            if (message.type() === 'error') {
                console.error(`Browser Console Error: ${message.text()}`);
            }
        });
    }

    setupNetworkListener() {
        this.page.on('requestfailed', request => {
            console.error(`Failed Request: ${request.url()}`);
            console.error(`Error: ${request.failure().errorText}`);
        });
    }

    async saveDebugSnapshot() {
        const state = {
            url: this.page.url(),
            cookies: await this.page.cookies(),
            localStorage: await this.page.evaluate(() => Object.entries(localStorage)),
            console: this.logs
        };
        return state;
    }
}
```

### Visual Debugging Support

Create a comprehensive visual debugging system:

```javascript
// support/visual_debug.js
class VisualDebugger {
    constructor(page, screenshotPath) {
        this.page = page;
        this.screenshotPath = screenshotPath;
    }

    async highlightElement(selector) {
        await this.page.evaluate((sel) => {
            const element = document.querySelector(sel);
            if (element) {
                const originalStyle = element.getAttribute('style');
                element.style.border = '3px solid red';
                element.style.backgroundColor = 'yellow';
                
                setTimeout(() => {
                    element.style = originalStyle;
                }, 3000);
            }
        }, selector);
    }

    async captureScreenshot(name, fullPage = true) {
        const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
        const path = `${this.screenshotPath}/${name}_${timestamp}.png`;
        
        await this.page.screenshot({
            path,
            fullPage
        });
        
        return path;
    }

    async recordVideo(scenarioName) {
        // Implementation depends on your testing environment
        // This is a conceptual example
        const videoPath = `${this.screenshotPath}/videos/${scenarioName}.mp4`;
        await this.page.video().start({
            path: videoPath,
            width: 1280,
            height: 720
        });
        return videoPath;
    }
}
```

### Performance Monitoring

Implement comprehensive performance tracking:

```javascript
// support/performance_monitor.js
class PerformanceMonitor {
    constructor(page) {
        this.page = page;
        this.metrics = {};
    }

    async startTracking() {
        await this.page.tracing.start({
            path: 'trace.json',
            screenshots: true
        });
    }

    async stopTracking() {
        await this.page.tracing.stop();
        
        const metrics = await this.page.metrics();
        const performanceTimings = await this.page.evaluate(() => 
            JSON.stringify(performance.getEntriesByType('navigation'))
        );
        
        this.metrics = {
            ...metrics,
            navigationTimings: JSON.parse(performanceTimings)
        };
        
        return this.metrics;
    }

    async getPageLoadMetrics() {
        const navigationTimings = await this.page.evaluate(() => {
            const timing = performance.getEntriesByType('navigation')[0];
            return {
                dnsLookup: timing.domainLookupEnd - timing.domainLookupStart,
                tcpConnection: timing.connectEnd - timing.connectStart,
                serverResponse: timing.responseEnd - timing.requestStart,
                domComplete: timing.domComplete - timing.domLoading,
                pageLoad: timing.loadEventEnd - timing.navigationStart
            };
        });
        
        return navigationTimings;
    }
}
```

### Error Reporting System

Create a sophisticated error reporting mechanism:

```javascript
// support/error_reporter.js
class ErrorReporter {
    constructor(world) {
        this.world = world;
        this.errors = [];
    }

    async captureError(error, context) {
        const errorReport = {
            timestamp: new Date().toISOString(),
            error: {
                message: error.message,
                stack: error.stack
            },
            context: {
                url: await this.world.page.url(),
                scenario: this.world.currentScenario,
                step: context.step
            },
            browserLogs: await this.world.debugHelper.getLogs(),
            screenshot: await this.world.visualDebugger.captureScreenshot('error')
        };
        
        this.errors.push(errorReport);
        return errorReport;
    }

    generateErrorReport() {
        return {
            summary: {
                totalErrors: this.errors.length,
                uniqueErrors: new Set(this.errors.map(e => e.error.message)).size
            },
            details: this.errors
        };
    }
}
```

### Integration with World Object

Connect all debugging tools through the World object:

```javascript
// support/world.js
class CustomWorld {
    async initialize() {
        this.browser = await puppeteer.launch(this.config.browserOptions);
        this.page = await this.browser.newPage();
        
        // Initialize debugging tools
        this.debugHelper = new DebugHelper(this.page);
        this.visualDebugger = new VisualDebugger(this.page, './screenshots');
        this.performanceMonitor = new PerformanceMonitor(this.page);
        this.errorReporter = new ErrorReporter(this);
        
        await this.performanceMonitor.startTracking();
    }

    async captureDebugInfo(context) {
        const debugInfo = {
            screenshot: await this.visualDebugger.captureScreenshot('debug'),
            console: await this.debugHelper.getLogs(),
            performance: await this.performanceMonitor.getPageLoadMetrics(),
            state: await this.debugHelper.saveDebugSnapshot()
        };
        
        return debugInfo;
    }
}
```

### Hooks for Debugging

Implement comprehensive debugging hooks:

```javascript
// support/hooks.js
const { Before, After, Status } = require('@cucumber/cucumber');

Before(async function(scenario) {
    this.currentScenario = scenario;
    await this.initialize();
});

After(async function(scenario) {
    if (scenario.result.status === Status.FAILED) {
        const debugInfo = await this.captureDebugInfo(scenario);
        
        // Attach debug information to the scenario
        this.attach(JSON.stringify(debugInfo.console, null, 2), 'application/json');
        this.attach(debugInfo.screenshot, 'image/png');
        this.attach(JSON.stringify(debugInfo.performance, null, 2), 'application/json');
        
        // Generate and save error report
        const errorReport = await this.errorReporter.captureError(
            scenario.result.exception,
            { step: scenario.pickle.steps[scenario.result.stepIndex] }
        );
    }
    
    await this.cleanup();
});
```

### Best Practices Implementation

The module demonstrates several key testing best practices:

1. Comprehensive Error Handling
2. Detailed Debug Information Capture
3. Performance Monitoring
4. Visual Debugging Support
5. Console and Network Monitoring
6. State Management
7. Reporting Integration

These tools and practices enable effective debugging and maintain high-quality test automation. They help identify issues quickly and provide detailed information for troubleshooting.

### Testing Environment Considerations

The debugging tools adapt to different testing environments:

1. Local Development
2. Continuous Integration
3. Production Testing
4. Cross-browser Testing

Each environment may require different debugging approaches and tools, which this module's implementation supports through configuration options.

### Exercise: Debug Framework Implementation

Create a debugging framework that includes:

1. Custom error handling for specific types of failures
2. Visual debugging tools for UI testing
3. Performance monitoring for critical user paths
4. Comprehensive reporting system
5. Integration with existing test framework

This exercise will help reinforce debugging concepts while providing practical experience with implementing debugging tools and best practices.
