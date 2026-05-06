---
description: Notes taking while learning K6
---

# ☕ Learn K6

### Install K6 (Linux)

```bash
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

### Run Test Script

**Create New Script File**

```bash
k6 run script.js
```

**Add VUs**

```bash
k6 run --vus 10 --duration 30s script.js
```

**Set options** Instead of typing --vus 10 and --duration 30s each time you run the script, you can set the options in your JavaScript file:

```js
import http from 'k6/http';
import { sleep } from 'k6';
export const options = {
  vus: 10,
  duration: '30s',
};
export default function () {
  http.get('http://test.k6.io');
  sleep(1);
}
```

**Ramp VUs** Ramp Up -> Ramp Down

```js
export const options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '1m30s', target: 10 },
    { duration: '20s', target: 0 },
  ],
};
```

**Execution Modes** k6 supports three execution modes to run a k6 test:

1. local

```bash
   k6 run script.js
```

3. distributed

```js
apiVersion: k6.io/v1alpha1
kind: TestRun
metadata:
  name: k6-sample
spec:
  parallelism: 4
  script:
    configMap:
      name: 'k6-test'
      file: 'script.js'   
```

```js
kubectl apply -f /path/to/k6-testrun-resource.yaml
```

4. cloud

```bash
k6 cloud run script.js
```

***

#### Result Output Types

1. Metrics
   * http\_req\_duration
   * http\_req\_failed
   * iterations
2. End-of-Test Summary
   * Median & average values
   * Minimum & Maximum values
   * p90, p95 & p99 values
3. Custom Reports with handleSummary()
   * At the end of the test, k6 automatically creates an object with all aggregated statistics. The handleSummary() function can process this object into a custom report in any text format: JSON, HTML, XML, and whatever else.
4. Time series and external outputs

```bash
k6 run \
--out json=test.json \
--out influxdb=http://localhost:8086/k6

```

The available built-in outputs include:

* Amazon CloudWatch
* Cloud
* CSV
* Datadog
* Dynatrace
* Elasticsearch
* Grafana Cloud Prometheus
* InfluxDB
* JSON
* Netdata
* New Relic
* Prometheus
* TimescaleDB
* StatsD

***

#### Using K6

**K6 deps Command:** This outputs dependency information in human-readable format to stdout. `k6 deps [options] <script>`

**Run a K6 test script**

1.  Run local machine:

    <pre class="language-bash"><code class="lang-bash"><strong> k6 run script.js
    </strong></code></pre>
2. Run a test using Grafana Cloud K6:

```bash
k6 cloud login --token <API_TOKEN> --stack <STACK_SLUG>
k6 cloud run cloud_demo.js
```

***

#### Requests

1. HTTP Requests
   * Get request looks like

```js
  import http from 'k6/http';

  export default function () {
  http.get('http://test.k6.io');
}
```

* Post request looks like

```js
import http from 'k6/http';

export default function () {
  const url = 'http://test.k6.io/login';
  const payload = JSON.stringify({
    email: 'aaa',
    password: 'bbb',
  });

  const params = {
    headers: {
      'Content-Type': 'application/json',
    },
  };

  http.post(url, payload, params);
}
```

The http module handles all kinds of HTTP requests and methods:

| Name      | Value                                                       |
| --------- | ----------------------------------------------------------- |
| batch()   | Issue multiple HTTP requests in parallel (like browsers do) |
| del()     | Issue an HTTP DELETE request                                |
| get()     | Issue an HTTP GET request                                   |
| head()    | Issue an HTTP HEAD request                                  |
| options() | Issue an HTTP OPTIONS request                               |
| patch()   | Issue an HTTP PATCH request                                 |
| post()    | Issue an HTTP POST request                                  |
| put()     | Issue an HTTP PUT request                                   |
| request() | Issue any type of HTTP request                              |

***

**Follow Redirects** By default, k6 automatically follows a set number of redirects before stopping and returning the last response:

* maxRedirects
* Params.redirects

**HTTP Request Tags**

| Name               | Description                                                                                        |
| ------------------ | -------------------------------------------------------------------------------------------------- |
| expected\_response | By default, response statuses between 200 and 399 are true. Change using `setResponseCallback`.    |
| group              | When the request runs inside a group, the tag value is the group name. Default is empty.           |
| name               | Defaults to the requested URL                                                                      |
| method             | Request method (GET, POST, PUT, etc.)                                                              |
| scenario           | When the request runs inside a scenario, the tag value is the scenario name. Default is `default`. |
| status             | Response status                                                                                    |
| url                | Defaults to the requested URL                                                                      |

***

#### Built-in Metrics

**Standard built-in metrics:**

| Metric Name         | Type    | Description                                                                                                                                                     |
| ------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| checks              | Rate    | The rate of successful checks.                                                                                                                                  |
| data\_received      | Counter | The amount of received data. This example covers how to track data for an individual URL.                                                                       |
| data\_sent          | Counter | The amount of data sent. Track data for an individual URL.                                                                                                      |
| dropped\_iterations | Counter | The number of iterations that weren’t started due to lack of VUs (arrival-rate executors) or lack of time (expired `maxDuration` in iteration-based executors). |
| iteration\_duration | Trend   | Time to complete one full iteration, including setup and teardown.                                                                                              |
| iterations          | Counter | Total number of times VUs execute the JS script (default function).                                                                                             |
| vus                 | Gauge   | Current number of active virtual users.                                                                                                                         |
| vus\_max            | Gauge   | Maximum possible number of virtual users (pre-allocated to avoid scaling performance impact).                                                                   |

**HTTP-specific built-in metrics:** For all http\_req\_\* metrics, the timestamp is emitted the end of the request.

| Metric Name                 | Type    | Description                                                                                |
| --------------------------- | ------- | ------------------------------------------------------------------------------------------ |
| http\_req\_blocked          | Trend   | Time spent blocked (waiting for a free TCP connection slot) before initiating the request. |
| http\_req\_connecting       | Trend   | Time spent establishing TCP connection to the remote host.                                 |
| http\_req\_duration         | Trend   | Total request time = sending + waiting + receiving (excludes DNS lookup/connection time).  |
| http\_req\_failed           | Rate    | Rate of failed requests according to `setResponseCallback`.                                |
| http\_req\_receiving        | Trend   | Time spent receiving response data from the remote host.                                   |
| http\_req\_sending          | Trend   | Time spent sending data to the remote host.                                                |
| http\_req\_tls\_handshaking | Trend   | Time spent performing TLS handshake with the remote host.                                  |
| http\_req\_waiting          | Trend   | Time spent waiting for response (Time To First Byte - TTFB).                               |
| http\_reqs                  | Counter | Total number of HTTP requests generated by k6.                                             |

**Browser metrics**

| Metric Name              | Description                                                                                                             |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| browser\_web\_vital\_cls | Measures visual stability by quantifying unexpected layout shifts of visible page content (Cumulative Layout Shift).    |
| browser\_web\_vital\_fid | Deprecated in favor of INP. Measures delay between a user’s first interaction and browser response (First Input Delay). |
| browser\_web\_vital\_inp | Measures page responsiveness (Interaction to Next Paint).                                                               |
| browser\_web\_vital\_lcp | Measures time for the largest content element on the page to become visible (Largest Contentful Paint).                 |

**Built-in WebSocket metrics**

| Metric Name           | Type    | Description                                                                    |
| --------------------- | ------- | ------------------------------------------------------------------------------ |
| ws\_connecting        | Trend   | Total duration for the WebSocket connection request.                           |
| ws\_msgs\_received    | Counter | Total number of messages received.                                             |
| ws\_msgs\_sent        | Counter | Total number of messages sent.                                                 |
| ws\_ping              | Trend   | Duration between a ping request and its pong reception.                        |
| ws\_session\_duration | Trend   | Duration of the WebSocket session (from connection start to VU execution end). |
| ws\_sessions          | Counter | Total number of started WebSocket sessions.                                    |

***

#### Checks

**Check for HTTP response code**

```js
import { check } from 'k6';
import http from 'k6/http';

export default function () {
  const res = http.get('http://test.k6.io/');
  check(res, {
    'is status 200': (r) => r.status === 200,
  });
}
```

**Check for text in response body**

```js
check(res, {
    'verify homepage text': (r) =>
      r.body.includes('Collection of simple web-pages suitable for load testing'),
  };
```

**Check for response body size**

```js
check(res, {
    'body size is 11,105 bytes': (r) => r.body.length == 11105,
  };
```

**Add multiple checks**

```js
 check(res, {
    'is status 200': (r) => r.status === 200,
    'body size is 11,105 bytes': (r) => r.body.length == 11105,
  };
```

***

#### Assertions

```js
import { expect } from 'https://jslib.k6.io/k6-testing/0.6.1/index.js';
import { browser } from 'k6/browser';
import http from 'k6/http';

export function protocolTest() {
  // Get the home page of k6's Quick Pizza app
  const response = http.get('https://quickpizza.grafana.com/');

  // Simple assertions
  expect(response.status).toBe(200);
  expect(response.error).toEqual('');
  expect(response.body).toBeDefined();
}

export async function browserTest() {
  const page = await browser.newPage();

  try {
    await page.goto('https://quickpizza.grafana.com/');

    // Assert the "Pizza Please" button is visible
    await expect(page.locator('button[name=pizza-please]')).toBeVisible();
  } finally {
    await page.close();
  }
}

export const options = {
  scenarios: {
    // Protocol tests
    protocol: {
      executor: 'shared-iterations',
      vus: 1,
      iterations: 1,
      exec: 'protocolTest',
    },

    // Browser tests
    ui: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
      exec: 'browserTest',
    },
  },
};
```

### Types of assertions

1. **Non-retrying assertions:** Non-retrying assertions evaluate conditions immediately at a single point in time without retrying. They’re ideal for validating static data such as HTTP response content, data structures, or configuration values.

```js
import { expect } from 'https://jslib.k6.io/k6-testing/0.6.1/index.js';
import http from 'k6/http';

export default function () {
  const pizzaRequestPayload = { maxCaloriesPerSlice: 1000, mustBeVegetarian: true };
  const pizzaRequestHeader = {
    'Content-Type': 'application/json',
    'Authorization': 'Token ' + 'abcdef0123456789',
  };

  const response = http.post(
    `https://quickpizza.grafana.com/api/pizza`,
    JSON.stringify(pizzaRequestPayload),
    { headers: pizzaRequestHeader }
  );
  const data = response.json();

  // These assertions evaluate immediately
  expect(response.status).toEqual(200);
  expect(response.headers['Content-Type']).toBeDefined();
  expect(response.headers['Content-Type']).toEqual('application/json');
  expect(data.pizza).toBeDefined();
  expect(data.pizza.name).toBeDefined();
  expect(data.pizza.name).not.toHaveLength(0);
}

```

| Method                   | Description                                 |
| ------------------------ | ------------------------------------------- |
| toBe()                   | Exact equality using Object.is()            |
| toEqual()                | Deep equality comparison                    |
| toBeTruthy()             | Value is truthy                             |
| toBeFalsy()              | Value is falsy                              |
| toBeDefined()            | Value is not undefined                      |
| toBeUndefined()          | Value is undefined                          |
| toBeNull()               | Value is null                               |
| toBeGreaterThan()        | Numeric greater than                        |
| toBeGreaterThanOrEqual() | Numeric greater than or equal               |
| toBeLessThan()           | Numeric less than                           |
| toBeLessThanOrEqual()    | Numeric less than or equal                  |
| toBeCloseTo()            | Floating point comparison                   |
| toContain()              | Array/string contains value                 |
| toContainEqual()         | Array contains object with matching content |
| toHaveLength()           | Array/string has specific length            |
| toHaveProperty()         | Object has specific property                |
| toBeInstanceOf()         | Value is instance of class                  |

2. **Auto-retrying assertions:** It automatically retry until conditions become true or a timeout is reached

```js
   import { expect } from 'https://jslib.k6.io/k6-testing/0.6.1/index.js';
import { browser } from 'k6/browser';

export default async function () {
  // Open a new browser page
  const page = await browser.newPage();

  try {
    // Navigate to the quickpizza website
    await page.goto('https://quickpizza.grafana.com/');

    // Click the 'Pizza please' button
    await page.locator('button[name="pizza-please"]').click();

    // Take a screenshot of the homepage, and save it to the local filesystem
    // so we can inspect it later if needed.
    await page.screenshot({ path: 'homepage.png' });

    // Check if the pizza recipe is displayed
    const textContent = await pizzaRecipeIsDisplayed(page);
    expect(textContent).toEqual('Our recommendation:');
  } finally {
    await page.close();
  }
}

// Browsers are asynchronous, so we need to wait for the content we want to check
// to be visible.
async function pizzaRecipeIsDisplayed(page) {
  const label = await page.locator('h2[id="pizza-name"]');
  await label.isVisible();
  const textContent = (await label.textContent()).trim();

  return textContent;
}

export const options = {
  scenarios: {
    ui: {
      executor: 'shared-iterations',
      options: {
        browser: {
          type: 'chromium',
        },
      },
    },
  },
};
```

| Method            | Description                          |
| ----------------- | ------------------------------------ |
| toBeVisible()     | Element is visible on the page       |
| toBeHidden()      | Element is hidden or not visible     |
| toBeEnabled()     | Element is enabled and interactive   |
| toBeDisabled()    | Element is disabled                  |
| toBeChecked()     | Checkbox or radio button is checked  |
| toBeEditable()    | Element is editable                  |
| toHaveText()      | Element has specific text content    |
| toContainText()   | Element contains specific text       |
| toHaveValue()     | Input element has specific value     |
| toHaveAttribute() | Element has specific attribute value |

#### Assertion features

**Negation:** .not

```js
  // Negated assertions
  expect(response.status).not.toBe(404);
  expect(response.body).not.toHaveLength(0);
  expect(response.headers).not.toHaveProperty('error');
```

**Soft assertions:** Soft assertions continue test execution even when they fail, marking the test as failed but allowing subsequent assertions to run.

```js
  // These will all run even if some fail
  expect.soft(response.status).toBe(200);
  expect.soft(response.headers['Content-Type']).toContain('text/html');
  expect.soft(response.body).toHaveLength(response.body.length);

```

**Custom error message:**

```js
import { expect } from 'https://jslib.k6.io/k6-testing/0.6.1/index.js';
import http from 'k6/http';

export default function () {
  const response = http.get('https://quickpizza.grafana.com/api/pizza', {
    headers: { 'Content-Type': 'application/json' },
  });
  const pizza = response.json();

  expect(response.status, 'API should return successful response').toBe(200);
  expect(pizza.name, 'Pizza should have a valid name').toBeDefined();
  expect(pizza.name, 'Pizza name should not be empty').not.toHaveLength(0);
}
```

#### Configuration & Customization

**Global Configuration:**

```js
import { expect } from 'https://jslib.k6.io/k6-testing/0.6.1/index.js';
import http from 'k6/http';

// Configure global settings
const configuredExpect = expect.configure({
  timeout: 10000, // 10 seconds for retrying assertions
  interval: 500, // Retry check every 500ms (on retriable assertions)
  colorize: true, // Enable colored output

  // Setting `softMode` to 'throw', will make soft assertions fail the current iteration instead of the whole test.
  softMode: 'fail',
});

export default function () {
  const response = http.get('https://quickpizza.grafana.com');

  // All assertions use these settings
  configuredExpect(response.status).toBe(200);
}
```

**Per-scenario configuration:**

```js
import { expect } from 'https://jslib.k6.io/k6-testing/0.6.1/index.js';
import http from 'k6/http';
import { browser } from 'k6/browser';

// Fast assertions for API testing
const fastExpect = expect.configure({
  timeout: 2000,
  interval: 100,
});

// Slow assertions for complex browser interactions
const slowExpect = expect.configure({
  timeout: 30000,
  interval: 1000,
  softMode: 'continue',
});

export default async function () {
  // Use appropriate expectation based on test type
  const response = http.get('https://quickpizza.grafana.com/');
  fastExpect(response.status).toBe(200);

  if (__ENV.BROWSER_TEST) {
    const page = await browser.newPage();
    await page.goto('https://quickpizza.grafana.com/');
    await slowExpect(page.locator('h1')).toBeVisible();
  }
}
```

***

### Thresholds

Thresholds are the pass/fail criteria that you define for your test metrics Thresholds are also essential for load-testing automation:

1. Give your test a threshold.
2. Automate your execution
3. Set up alerts for test failures.

#### Thresholds example for HTTP errors and response duration

```js
import http from 'k6/http';

export const options = {
  thresholds: {
    http_req_failed: ['rate<0.01'], // http errors should be less than 1%
    http_req_duration: ['p(95)<200'], // 95% of requests should be below 200ms
  },
};

export default function () {
  http.get('https://quickpizza.grafana.com');
}
```

#### Threshold Syntax

```js
export const options = {
  thresholds: {
    /* ... */
  },
};
```

***

### Options

Options configure test-run behavior.

```js
import http from 'k6/http';

import http from 'k6/http';

export const options = {
  hosts: { 'test.k6.io': '1.2.3.4' },
  stages: [
    { duration: '1m', target: 10 },
    { duration: '1m', target: 20 },
    { duration: '1m', target: 0 },
  ],
  thresholds: { http_req_duration: ['avg<100', 'p(95)<200'] },
  noConnectionReuse: true,
  userAgent: 'MyK6UserAgentString/1.0',
};

export default function () {
  http.get('http://test.k6.io/');
}
```

### Test lifecycle

1. Code in the **init** context prepares the script, loading files, importing modules, and defining the test lifecycle functions. **Required**.
2. The **setup** function runs, setting up the test environment and generating data. **Optional**.
3. VU code runs in the **default** or scenario function, running for as long and as many times as the **options** define. **Required**.
4. The **teardown** function runs, postprocessing data and closing the test environment. **Optional**.

```js
// 1. init code

export function setup() {
  // 2. setup code
}

export default function (data) {
  // 3. VU code
}

export function teardown(data) {
  // 4. teardown code
}
```

### Defination

* virtual users
* Ramp Up
* Ramp Down
* http\_req\_duration
* http\_req\_failed
* iterations
* p90
* p95
* p99
* metric
* assertionk6 version

***
