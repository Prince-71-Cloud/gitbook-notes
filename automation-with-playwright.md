# 🍕 Automation with Playwright

## Advanced Playwright Techniques: Optimizing Speed, Stability, and Cloud Testing

These notes work as a technical reference for:

* Beginner test engineers
* Intermediate automation engineers
* Senior QA architects

## 1. Optimizing test speed in Playwright

### 1.1 Green testing: speed, efficiency, and sustainability

#### Beginner: What is green software development?

Green software development means designing, coding, testing, and operating software with less waste.

In automated testing, green testing means reducing:

* Unnecessary CPU cycles
* Repeated browser work
* Redundant setup
* Duplicate tests
* Long CI/CD execution time

#### Why test speed matters

Fast tests give fast feedback.

Slow tests delay releases, waste developer time, increase CI/CD cost, and use more compute.

{% code overflow="wrap" %}
```
Optimized tests can reduce runtime from 1 hour to 11 minutes, or even under 15 minutes, depending on the suite and infrastructure.
```
{% endcode %}

#### Immediate actions

{% code overflow="wrap" %}
```
Use storageState to eliminate redundant logins.
```
{% endcode %}

Recommended actions:

* Use `storageState` so tests do not repeat UI logins.
* Identify slow steps with VS Code Playwright tools, traces, and reports.
* Enable parallel execution when tests are isolated.
* Remove outdated or redundant tests.
* Mock slow external dependencies when the test only covers frontend behavior.

#### How can you start?

#### Senior-level considerations

Before optimizing, measure:

* Total suite duration
* Slowest test files
* Slowest test steps
* Retry count
* Flaky tests
* Worker count
* CI resource usage
* Browser and project matrix
* Back-end and API bottlenecks

A good optimization workflow:

1. Measure.
2. Remove low-value tests.
3. Reuse authentication state.
4. Replace hard waits.
5. Mock unstable dependencies.
6. Paralleled safely.
7. Track runtime and flakiness after changes.

***

### 1.2 Diagnosing performance bottlenecks with VS Code metrics

A performance bottleneck is the slowest part of a test.

Common bottlenecks include:

* Navigation
* Repeated login
* Hard waits
* Slow API calls
* Overly broad assertions

#### Navigation example

<details>

<summary>Example 01</summary>

```ts
Don’t use :
await page.goto(‘https://playwright.dev/’)

Use :
await page.goto(‘https://playwright.dev/’ , (waitUntil: “commit”);
// hover on goto keyword you see options
```

</details>



<details>

<summary>Example 02: </summary>

{% code overflow="wrap" %}
```javascript
await page.goto('https://playwright.dev/', {
  waitUntil: 'commit',
});
```
{% endcode %}

</details>

#### Explanation

`page.goto()` navigates to a URL.

The `waitUntil` option controls how long Playwright waits.

| Value              | Meaning                                                               | Use case                                            |
| ------------------ | --------------------------------------------------------------------- | --------------------------------------------------- |
| `commit`           | Waits until response headers are received and navigation is committed | Fastest useful option when full load is unnecessary |
| `domcontentloaded` | Waits until HTML is parsed                                            | Useful when DOM is enough                           |
| `load`             | Waits until the load event fires                                      | General default                                     |
| `networkidle`      | Waits for network quietness                                           | Use carefully because it can slow tests             |

Example:

```ts
await page.goto('https://playwright.dev/', {
  waitUntil: 'commit',
});

await expect(
  page.getByRole('heading', { name: /playwright/i })
).toBeVisible();
```

#### Assertion example

<details>

<summary>Detailed example</summary>

```ts
Don’t use:
await  expect(page).toHaveTitle(/part of title/)

Use:
await  expect(await page.title().toBe(“Full title”);
```

</details>

Or

```ts
expect(await page.title()).toBe('Full title');
```

Recommended Playwright web-first assertion:

```ts
await expect(page).toHaveTitle('Full title');
```

#### Explanation

This direct assertion checks once:

```ts
expect(await page.title()).toBe('Full title');
```

This Playwright assertion retries until the title matches or timeout expires:

```ts
await expect(page).toHaveTitle('Full title');
```

Use direct assertions when the value is already stable.

Use Playwright assertions when the UI may update asynchronously.

***

### 1.3 Using `storageState` to avoid repeated logins

#### Beginner: What is `storageState`?

`storageState` is a saved browser state file.

It can store cookies and local storage so tests start already authenticated.

This avoids logging in through the UI before every test.

code:

```ts
test.use({ storageState: 'auth.json' });
```

Recommended location:

```
playwright/.auth/user.json
```

Recommended `.gitignore`:

```gitignore
playwright/.auth
```

Corrected `global-setup.ts`:

```ts
import { chromium, type FullConfig } from '@playwright/test';

async function globalSetup(config: FullConfig) {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();

  await page.goto('https://practicesoftwaretesting.com/auth/login');

  await page
    .getByRole('textbox', { name: 'Email Address *' })
    .fill('test@test.com');

  await page
    .getByRole('textbox', { name: 'Password *' })
    .fill('Testing123!!');

  await page
    .getByRole('button', { name: 'Login' })
    .click();

  await page.waitForURL('https://practicesoftwaretesting.com/account');

  await page.context().storageState({
    path: 'playwright/.auth/user.json',
  });

  await browser.close();
}

export default globalSetup;
```

#### Common mistakes you should be aware

| Original              | Corrected               | Reason                                |
| --------------------- | ----------------------- | ------------------------------------- |
| `Import`              | `import`                | TypeScript keywords are lowercase     |
| `Async`               | `async`                 | JavaScript keywords are lowercase     |
| `getbyRole`           | `getByRole`             | Playwright methods are case-sensitive |
| Missing browser close | `await browser.close()` | Prevents resource leaks               |
|                       |                         |                                       |

#### `playwright.config.ts`

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  globalSetup: require.resolve('./global-setup'),
});
```

#### Recommended setup project alternative

```ts
import { test as setup, expect } from '@playwright/test';

setup('authenticate user', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com/auth/login');

  await page
    .getByRole('textbox', { name: 'Email Address *' })
    .fill('test@test.com');

  await page
    .getByRole('textbox', { name: 'Password *' })
    .fill('Testing123!!');

  await page
    .getByRole('button', { name: 'Login' })
    .click();

  await expect(page).toHaveURL(/.*account/);

  await page.context().storageState({
    path: 'playwright/.auth/user.json',
  });
});
```

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'setup',
      testMatch: /.*\.setup\.ts/,
    },
    {
      name: 'chromium',
      use: {
        ...devices['Desktop Chrome'],
        storageState: 'playwright/.auth/user.json',
      },
      dependencies: ['setup'],
    },
  ],
});
```

#### Senior-level considerations

Avoid using one shared account across many parallel tests when tests modify account data.

Better patterns:

* One account per worker
* Role-specific auth states
* API-based test data setup
* Isolated test tenants
* Separate admin and customer states
* Database cleanup after tests

***

### 1.4 Configuring project dependencies for cookie setup

Project dependencies let one Playwright project run before another.

This is useful for:

* Login
* Cookie generation
* Test data setup
* Environment preparation

Code:

```ts
dependencies: ['cookie-spec'] // playwright.config.ts
```

Cookie code:

```ts
import fs from 'fs';
import { test } from '@playwright/test';

test.beforeEach(async ({ context }) => {
  const cookies = JSON.parse(fs.readFileSync('cookies.json', 'utf-8'));

  await context.addCookies(cookies);
});
```

#### Full cookie setup example

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  projects: [
    {
      name: 'cookie-spec',
      testMatch: /cookie\.setup\.ts/,
    },
    {
      name: 'chromium',
      dependencies: ['cookie-spec'],
    },
  ],
});
```

```ts
import fs from 'fs';
import { test } from '@playwright/test';

test('save cookies', async ({ page, context }) => {
  await page.goto('https://practicesoftwaretesting.com/auth/login');

  await page
    .getByRole('textbox', { name: 'Email Address *' })
    .fill('test@test.com');

  await page
    .getByRole('textbox', { name: 'Password *' })
    .fill('Testing123!!');

  await page
    .getByRole('button', { name: 'Login' })
    .click();

  await page.waitForURL('**/account');

  const cookies = await context.cookies();

  fs.writeFileSync('cookies.json', JSON.stringify(cookies, null, 2));
});
```

#### Cookies vs `storageState`

Use cookies when you need cookie-level control.

Use `storageState` when you want complete browser authentication reuse.

***

### 1.5 Parallelization: when to use it and when to avoid it

#### What is parallel execution?

Parallel execution means running tests at the same time with worker processes.

Code:

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  fullyParallel: true,
  workers: process.env.CI ? 1 : '100%',
});
```

#### More practical configuration

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  fullyParallel: true,
  workers: process.env.CI ? 4 : undefined,
  retries: process.env.CI ? 2 : 0,
});
```

#### Challenges with parallel execution

Parallel execution can create problems when tests share mutable state.

Common risks:

* Shared user accounts
* Shared database records
* Order-dependent tests
* Rate limits
* Resource exhaustion
* Conflicting file writes
* Backend overload

#### Real-world insights

#### Parallelism vs sharding

Parallelism runs tests concurrently within one job.

Sharding splits tests across multiple jobs or machines.

```bash
npx playwright test --shard=1/7
npx playwright test --shard=2/7
npx playwright test --shard=3/7
npx playwright test --shard=4/7
npx playwright test --shard=5/7
npx playwright test --shard=6/7
npx playwright test --shard=7/7
```

#### Senior-level guidance

Start with a conservative worker count.

Increase it gradually.

Measure:

* Runtime
* Retry rate
* Flaky failures
* CPU and memory
* API load
* Database contention
* CI cost

***

### 1.6 Challenge: optimise a slow test in VS Code

#### Optimization checklist

* Remove hard waits.
* Reuse authentication with `storageState`.
* Navigate directly to the tested page.
* Use `waitUntil: 'commit'` when safe.
* Use reliable locators.
* Mock slow network calls when they are not part of the test objective.
* Keep the same test outcome.

#### Example slow test

```ts
import { test, expect } from '@playwright/test';

test('slow account test', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com/auth/login');
  await page.waitForTimeout(5000);

  await page.getByRole('textbox', { name: 'Email Address *' }).fill('test@test.com');
  await page.waitForTimeout(5000);

  await page.getByRole('textbox', { name: 'Password *' }).fill('Testing123!!');
  await page.waitForTimeout(5000);

  await page.getByRole('button', { name: 'Login' }).click();
  await page.waitForTimeout(5000);

  await expect(page).toHaveURL(/.*account/);
});
```

#### Optimized version

```ts
import { test, expect } from '@playwright/test';

test.use({
  storageState: 'playwright/.auth/user.json',
});

test('fast account test', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com/account', {
    waitUntil: 'commit',
  });

  await expect(page).toHaveURL(/.*account/);
});
```

***

### 1.7 Solution: optimize a slow test in VS Code

#### Recommended solution pattern

```ts
import { test, expect } from '@playwright/test';

test.use({
  storageState: 'playwright/.auth/user.json',
});

test('authenticated account page loads quickly', async ({ page }) => {
  await page.goto('/account', {
    waitUntil: 'commit',
  });

  await expect(
    page.getByRole('heading', { name: /account/i })
  ).toBeVisible();
});
```

***

## 2. Reducing test flakiness in Playwright

### 2.1 Mastering flaky tests

#### What is a flaky test?

A flaky test sometimes passes and sometimes fails without a meaningful application change.

#### Common causes

<details>

<summary>Common Causes of Test Flakiness</summary>

```
* Timing Issues: delays or race conditions
* Dependency fluctuations: API or database inconsistencies
* Environment instability: resource constraints or network issues
* Poor test design: hard-coded waits or lack of isolation
Example
```

</details>

#### Example: bad hard wait

```ts
await page.getByRole('button', { name: 'Submit' }).click();
await page.waitForTimeout(3000);
await expect(page.getByText('Success')).toBeVisible();
```

#### Better condition-based wait

```ts
await page.getByRole('button', { name: 'Submit' }).click();
await expect(page.getByText('Success')).toBeVisible();
```

#### Senior-level guidance

To reduce flakiness:

* Use web-first assertions.
* Avoid `waitForTimeout()`.
* Mock unstable APIs.
* Isolate test data.
* Avoid shared mutable state.
* Avoid order dependencies.
* Use repeat runs to verify stability.
* Treat retries as a signal, not a solution.

***

### 2.2 Handling Nuxt page hydration issue in tests

#### Explanation

Hydration happens when server-rendered HTML becomes interactive after JavaScript loads and attaches event listeners.

Before hydration completes:

* Buttons may appear but not work.
* Inputs may accept text and then reset.
* Clicks may be ignored.
* UI state may be replaced.

#### Nuxt hydration wait

```ts
await page.waitForFunction(() => {
  return window.useNuxtApp?.().isHydrating === false;
});
```

#### TypeScript-friendly version

```ts
await page.waitForFunction(() => {
  const nuxtApp = (window as any).useNuxtApp?.();

  return nuxtApp?.isHydrating === false;
});
```

Completed thought:

```
The most probable reason behind that is poor page hydration.
```

Best application-level fix:

Disable interactive controls until hydration completes.

Example app readiness marker:

```html
<body data-hydrated="true">
```

Test:

```ts
await expect(page.locator('body')).toHaveAttribute('data-hydrated', 'true');
```

***

### 2.3 Implementing stable locators

Corrected code:

```ts
// Avoid brittle CSS selectors
await page.click('#navbarSupportedContent > ul > li:nth-child(4) > a');

// Prefer accessible locators
await page.getByRole('link', { name: 'Sign in' }).click();
```

#### Locator priority

1. `getByRole()`
2. `getByLabel()`
3. `getByPlaceholder()`
4. `getByText()`
5. `getByTestId()`
6. CSS selectors
7. XPath as a last resort

#### Examples

```ts
await page.getByRole('button', { name: 'Login' }).click();
await page.getByLabel('Email Address *').fill('test@test.com');
await page.getByPlaceholder('Search products').fill('pliers');
await page.getByTestId('product-card').first().click();
```

***

### 2.4 Handling external dependencies to minimise flakiness

External dependencies include:

* APIs
* Databases
* Authentication providers
* Payment services
* Third-party scripts

They cause flakiness because they can be slow, unstable, rate-limited, or return changing data.

#### Mocking API responses

```ts
await page.route('**/api/products', async route => {
  await route.fulfill({
    json: [
      {
        id: 'pliers-001',
        name: 'Long Nose Pliers',
        price: 14.99,
      },
    ],
  });
});
```

#### Example

```ts
import { test, expect } from '@playwright/test';

test('shows long nose pliers from mocked API', async ({ page }) => {
  await page.route('**/products**', async route => {
    await route.fulfill({
      json: {
        data: [
          {
            id: 'pliers-001',
            name: 'Long Nose Pliers',
            price: 14.99,
          },
        ],
      },
    });
  });

  await page.goto('https://practicesoftwaretesting.com');

  await expect(
    page.getByRole('link', { name: /long nose pliers/i })
  ).toBeVisible();
});
```

***

### 2.5 Running tests multiple times to detect flakiness

Command:

```bash
npx playwright test --grep "has title" --repeat-each=50
```

#### Explanation

* `--grep` selects tests by title.
* `--repeat-each=50` runs each selected test 50 times.

Use this to expose intermittent failures.

***

### 2.6 Challenge: fix this flaky test

#### Example solution

```ts
import { test, expect } from '@playwright/test';

test('long nose pliers product link exists', async ({ page }) => {
  await page.route('**/products**', async route => {
    await route.fulfill({
      json: {
        data: [
          {
            id: 'long-nose-pliers',
            name: 'Long Nose Pliers',
            price: 14.99,
          },
        ],
      },
    });
  });

  await page.goto('https://practicesoftwaretesting.com');

  await expect(
    page.getByRole('link', { name: /long nose pliers/i })
  ).toBeVisible();
});
```

***

### 2.7 Solution: fix this flaky test

```
Use mocked API response data.
```

#### Final solution

```ts
import { test, expect } from '@playwright/test';

test('long nose pliers product is visible with mocked API data', async ({ page }) => {
  await page.route('**/products**', async route => {
    await route.fulfill({
      json: {
        data: [
          {
            id: 'long-nose-pliers',
            name: 'Long Nose Pliers',
            price: 14.99,
            image: 'long-nose-pliers.jpg',
          },
        ],
      },
    });
  });

  await page.goto('https://practicesoftwaretesting.com');

  const productLink = page.getByRole('link', {
    name: /long nose pliers/i,
  });

  await expect(productLink).toBeVisible();
});
```

***

## 3. Screenshots and snapshots testing best practices

### 3.1 Visual testing with screenshots and snapshots

#### Explanation

Visual testing checks how the UI looks.

Functional tests may pass even when layout, spacing, colors, or alignment are broken.

#### Expanded comparison

| Technique  | Compares                               | Best for                               |
| ---------- | -------------------------------------- | -------------------------------------- |
| Screenshot | Rendered pixels                        | Layout, visual regression, CSS changes |
| Snapshot   | Text, HTML, JSON, or serialized output | Markup, structure, serialized data     |

***

### 3.2 How do you capture a screenshot?

&#x20;Code:

```ts
import { test, expect } from '@playwright/test';

test('capture full page screenshot', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  await expect(page).toHaveScreenshot('homepage.png', {
    fullPage: true,
  });
});
```

#### Element screenshot example

```ts
import { test, expect } from '@playwright/test';

test('capture navbar screenshot', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  await expect(page.locator('nav')).toHaveScreenshot('navbar.png');
});
```

#### Update baselines

```bash
npx playwright test --update-snapshots
```

***

### 3.3 How do you capture a snapshot?

Code:

```ts
import { test, expect } from '@playwright/test';
import prettier from 'prettier';

async function sanitiseHTML(html: string): Promise<string> {
  const cleaned = html.replace(/_ngcontent-[^=]+="[^"]*"/g, '');

  const formatted = await prettier.format(cleaned, {
    parser: 'html',
    singleAttributePerLine: false,
  });

  return formatted;
}

test('snapshot of top menu', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  const html = await page.locator('#navbarSupportedContent').innerHTML();
  const sanitisedHTML = await sanitiseHTML(html);

  expect(sanitisedHTML).toMatchSnapshot('top-menu.html');
});
```

#### Key corrections

* `parse` changed to `parser`.
* `Return` changed to `return`.
* Smart quotes replaced with straight quotes.
* Variable name changed from `sanitiseHTML` to `sanitisedHTML` to avoid function shadowing.
* Snapshot assertion now uses sanitized HTML.

***

### 3.4 Challenge: implement a test to screenshot a pageCorrected heading:

```
Challenge: Implement a test to screenshot a page
```

#### Example solution

```ts
import { test, expect } from '@playwright/test';

test('homepage visual screenshot', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  await expect(page).toHaveScreenshot('homepage.png', {
    fullPage: true,
  });
});
```

***

### 3.5 Solution: implement a test to screenshot a page

#### Full solution

```ts
import { test, expect } from '@playwright/test';

test('capture full page screenshot', async ({ page }) => {
  await page.goto('https://practicesoftwaretesting.com');

  await expect(page).toHaveScreenshot('homepage.png', {
    fullPage: true,
  });
});
```

#### Stable visual test config example

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  use: {
    viewport: {
      width: 1280,
      height: 720,
    },
  },
  expect: {
    toHaveScreenshot: {
      maxDiffPixels: 100,
    },
  },
});
```

***

## 4. Running tests on Microsoft Playwright Testing service

### 4.1 What is Microsoft Playwright Testing (MPT) service?

#### Explanation

Microsoft Playwright Testing and Playwright Workspaces is a cloud execution service for running Playwright tests on managed remote browsers.

It helps teams scale test execution without maintaining their own browser infrastructure.

#### Polished comparison

| Area      | Playwright Framework             | Microsoft Playwright Testing / Workspaces |
| --------- | -------------------------------- | ----------------------------------------- |
| Type      | Open-source automation framework | Cloud execution service                   |
| Execution | Local or self-managed CI         | Microsoft-managed cloud browsers          |
| Scaling   | Manually configured              | Managed by the service                    |
| Best for  | Writing and running tests        | Running tests at scale                    |
| Debugging | Reports, traces, screenshots     | Cloud reports, logs, artifacts            |

#### Benefits of using MPT

<details>

<summary>Benefits of Using MPT</summary>

```
* No need to maintain test infrastructure
* Parallel test execution for faster feedback
* Built-in debugging tools and logs
```

</details>

#### Who should use MPT?

<details>

<summary>Who SHould use MPT</summary>

<pre><code><strong>* QA teams looking for scalable test execution
</strong>* Developers who want instant feedback on test runs
* CI/CD pipeline engineers automating deployment checks
</code></pre>

</details>

Corrected heading:

```
Who Should Use MPT?
```

Good users include:

* QA teams with large suites
* Developers needing fast browser feedback
* CI/CD engineers
* QA architects
* Platform teams managing test infrastructure

***

### 4.2 Creating a resource on Microsoft Azure

<details>

<summary>Original note preserved</summary>

```
Go to playwright workspaces
```

</details>

#### Expanded steps

1. Sign in to Azure Portal.
2. Search for Playwright Workspaces or Azure App Testing.
3. Create a new Playwright workspace.
4. Select subscription, resource group, and region.
5. Configure access and authentication.
6. Follow the generated quickstart instructions.
7. Add service configuration to your Playwright project.

***

### 4.3 Setting up your framework to run tests on the cloud

<details>

<summary>Original heading preserved</summary>

```
4.3 Setting up your framework to run tests on the cloud
```

</details>

#### Typical setup

A cloud setup usually includes:

* Azure workspace
* Authentication
* Service config file
* CI environment variables
* Playwright CLI command using service config

Conceptual service config example:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  workers: 20,
});
```

For local or private apps, you may need a network exposure option such as:

```ts
exposeNetwork: '<loopback>'
```

Always follow the generated Azure Playwright Workspace configuration for your project.

***

### 4.4 Running tests via CLI

<details>

<summary>Original command preserved</summary>

```bash
npx playwright test –config=playwright.service.config.ts –workers=20
```

</details>

Corrected command:

```bash
npx playwright test --config=playwright.service.config.ts --workers=20
```

#### Explanation

* `--config=playwright.service.config.ts` uses the service config file.
* `--workers=20` runs with up to 20 workers.

Single test:

```bash
npx playwright test tests/example.spec.ts --config=playwright.service.config.ts
```

Full suite:

```bash
npx playwright test --config=playwright.service.config.ts --workers=20
```

***

### 4.5 Challenge: run tests against your local server on MPT

<details>

<summary>Original challenge preserved</summary>

```
Execute Local Server Tests on MPT
* Your Task: configure and execute Playwright tests against your local server using the Microsoft Playwright Testing service
* Use the Playwright CLI to direct tests to your running local environment
```

</details>

#### Example local server test

```ts
import { test, expect } from '@playwright/test';

test('local app homepage loads from cloud browser', async ({ page }) => {
  await page.goto('http://localhost:3000');

  await expect(page.getByRole('heading')).toBeVisible();
});
```

#### Example command

```bash
npx playwright test --config=playwright.service.config.ts --workers=20
```

#### Important

The service config must expose the local app or network to the remote browser.

Conceptual option:

```ts
exposeNetwork: '<loopback>'
```

***

### 4.6 Solution: run tests against your local server on MPT

<details>

<summary>Original section preserved</summary>

```
4.6 Solution: Run tests against your local server on MPT
```

</details>

#### Example solution workflow

1. Start the local server.
2. Confirm the app is reachable locally.
3. Configure service network exposure.
4. Run Playwright using the service config.

Start server:

```bash
npm run dev
```

Confirm server:

```bash
curl http://localhost:3000
```

Run tests:

```bash
npx playwright test --config=playwright.service.config.ts --workers=20
```

#### Example `playwright.config.ts` with web server

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
  use: {
    baseURL: 'http://localhost:3000',
  },
});
```

#### Test using `baseURL`

```ts
import { test, expect } from '@playwright/test';

test('homepage loads', async ({ page }) => {
  await page.goto('/');

  await expect(page.getByRole('heading')).toBeVisible();
});
```

***

## Quick reference: corrected commands

```bash
npx playwright test
```

```bash
npx playwright test --grep "has title"
```

```bash
npx playwright test --grep "has title" --repeat-each=50
```

```bash
npx playwright test --update-snapshots
```

```bash
npx playwright test --config=playwright.service.config.ts
```

```bash
npx playwright test --config=playwright.service.config.ts --workers=20
```

```bash
npx playwright test --shard=1/4
```

***

## Quick reference: corrected code

### `storageState`

```ts
test.use({
  storageState: 'playwright/.auth/user.json',
});
```

### Fast navigation

```ts
await page.goto('https://playwright.dev/', {
  waitUntil: 'commit',
});
```

### Stable locator

```ts
await page.getByRole('link', { name: 'Sign in' }).click();
```

### Mock API

```ts
await page.route('**/products**', async route => {
  await route.fulfill({
    json: {
      data: [
        {
          id: 'long-nose-pliers',
          name: 'Long Nose Pliers',
          price: 14.99,
        },
      ],
    },
  });
});
```

### Screenshot

```ts
await expect(page).toHaveScreenshot('homepage.png', {
  fullPage: true,
});
```

### Snapshot

```ts
expect(sanitisedHTML).toMatchSnapshot('top-menu.html');
```

***

## LinkedIn course reference preserved

```
https://www.linkedin.com/learning/advanced-playwright-techniques-optimizing-speed-stability-and-cloud-testing/challenge-run-tests-against-your-local-server-on-mpt?autoSkip=true&resume=false&u=74413660
```

Original trailing values preserved:

```
41172384512640
2006
```

***

## Appendix A: Original raw course notes preserved

The following section preserves the original uploaded course notes exactly as text.

<details>

<summary>View original raw notes</summary>

```
﻿1. Optimising Test Speed in Playwright
1.1 Green testing: speed, efficiency and sustainability
What is Green Software Development?
* Every Line of code impacts the environment
* CPU cycles consume energy; datacenters emit carbon
* Green software development minimizes environmental impact by focusing on efficiency and sustainability.
Why Does Test Speed Matter?
* Slow tests lead to unhappy developers and stakeholders
* Higher costs in pipeline resources and slower feature delivery
* CPU cycles have a direct environmental impact
* Optimized tests reduces runtime from 1 hour to 11 minutes to under 15 minutes
Immediate Actions You Can Take Today
* Use StorageStage to eliminate redundant logins
* Identify slow steps with VS Code metrics
* Enable parallel execution with worker processes
How Can You Start?
* Audit your current tests: identify slow, redundant, or outdated tests
* Apply the “joy” principle: remove tests that no longer add value to your suite
* Optimize test execution: run tests in parallel and use efficient tools

1.2 Diagnosing performance bottlenecks with VS Code metrics

Don’t use :
await page.goto(‘https://playwright.dev/’)
Use :
await page.goto(‘https://playwright.dev/’ , (waitUntil: “commit”);
// hover on goto keyword you see options

Don’t use: await  expect(page).toHaveTitle(/part of title/)
Use: await  expect(await page.title().toBe(“Full title”);

1.3 Using storageState to avoid repeated logins

test.use({storageState: “auth.json”})

In global-setup.ts
Import { chromium, FullConfig} from “@playwright/test”

Async function globalSetup(config: FullConfig){
  const browser = await chromium.launch()
  const context = await browser.newContext()
  const  page = await context.newPage()
 await page.goto(“https://practicesoftwaretesting.com/auth/login”)
 await page.getbyRole(“textbox”,{name: “Email Address *”}).fill(“test@test.com”)
 await page.getByRole(“textbox”,{name: “Password *”}).fill(“Testing123!!”); await  page.getByRole(“button”, {name: “Login”}).click()
 await page.waitForURL(“https://practicesoftwaretesting.com/account”)
 await page.context().storageState({path: “auth.json”)}
}
Export default globalSetup

In playwright.config.ts
export default defineConfig({ globalSetup: require.resolve(“./global-setup”) }

1.4 Configuring project dependencies for cookie setup

dependencies: [“cookie-spec”] //playwright.config.ts

test.beforEach(async ({context}) => {
 await context.addCookies(
        JSON.parse(fs.readFileSync(“cookies.json”,”utf-8”))
) })

1.5 Parallelisation: When to use it and when to avoid it
Why parallel Execution Matters:
* Parallel execution sounds great - tests running faster, less waiting around
* But it’s not always the right move

Enabling Parallel Execution
import { defineConfig, devices } from “@playwright/test”;
export default defineConfig({
        fulllyParallel: true,
        Workers: process.env.CI ? 1: “100%”,
});

Challenges with Parallel Execution

Real-World Insights
* A student shared that they use four workers and seven shards for E2E tests in CI, and it works
* But assess your own system before blindly adopting this approach

1.6 Challenge: Optimise a slow test in VS Code

This test currently takes over 29 seconds to run, painfully slow, and definitely not green. Your challenge, optimize this test to run under two seconds without changing the outcome.

1.7 Solution: Optimise a slow test in VS Code

2. Reducing Test Flakiness in Playwright
2.1 Mastering flaky tests
What is Test Flakiness?
The Real Cost of Flaky Tests
* Reduce trust in test results
* Waste time debugging nonissues
* Slow down development and CI/CD workflows

Common Causes of Test Flakiness
* Timing Issues: delays or race conditions
* Dependency fluctuations: API or database inconsistencies
* Environment instability: resource constraints or network issues
* Poor test design: hard-coded waits or lack of isolation
Example

2.2 Handling Nuxt page hydration issue in tests
What is Hydration in Modern Web Apps?
Hydration is when a framework like Nuxt sends a pre-rendered HTML page to the browser, but it's not alive yet. The static page looks complete, but it isn't interactive. The playwright sees a button, tries to click, but nothing happens. Why? Because even listeners weren't attached yet, the page hadn't been hydrated. A temporary workaround is to wait for the hydration process to complete by checking window.useNuxtApp?.().isHydrating === false. This tells Playwright to pause until the app confirms it's fully hydrated.

But?
At some point in time, you’ll stumble upon a use case where Playwright performs an action, but nothing seemingly happens. Or you enter some text into the input field and it will disappear. The most probable reason behind that is a poor page

Source: Playwright Documentation
https://playwright.dev/docs/navigations#hydration

2.3 Implementing stable locators
Don’t use await page.click(“#navbarSupportedContent > ui > li:nth-child(4) > a”);
Use page.getByRole(“link”,{name: “Sign in”})

2.4 Handling external dependencies to minimise flakiness

2.5 Running tests multiple times to detect flakiness
npx playwright test –grep “has title” –repeat-each=50

2.6 Challenge: Fix this flaky test
I want you to mock the product API and write a test to check long nose pliers. Product link exists on the page. Let's see how you get on with that.

2.7 Solution: Fix this flaky test
Use Mock api response data

3. Screenshots and Snapshots Testing Best Practices

3.1 Visual testing with screenshots and snapshots
What is visual testing?
* Visual testing ensures your app appears exactly as intended
* It catches UI regressions that functional tests might overlook
Why Visual Testing Matters
* Detects unexpected UI changes before users do
* Maintains a consistent user experience across versions
* Prevents visual bugs from slipping into production

Screenshots vs. Snapshots
Screenshot:
Snapshots:

What’s Next
* Next, we’ll explore the benefits of using screenshots for pixel-perfect output
* Stay tuned to learn how to implement these techniques effectively

3.2 How do you capture a screenshots Code
import {test, expect} from “@playwright/test”

test (“capture full page screenshot”, async ({page}) => {
await page.goto(“https://practicesoftwaretesting.com”)
await expect(page).toHaveScreenshot(“homepage.png”,{“fullPage”:true)}
})

3.3 How do you capture a snapshots
import {test, expect} from “@playwright/test”
import prettier from ‘prettier’

async function sanitiseHTML(html: string): Promise<string>{
const cleaned = html.replace(/_ngcontent-[^=]+=”[^”]*”/g, “”)
 const formatted = await prettier.format(cleaned, { parse: ‘html’, singleAttributePerLine: false })
Return formatted
}

test (“snapshot of top menu”, async ({page}) => {
await page.goto(“https://practicesoftwaretesting.com”)
const html = await page.locator(“#navbarSupportContent”).innerHTML();
const sanitiseHTML = await sanitiseHTML(html);
 expect(html).toMatchSnapshot(“top-menu.html”)
})

3.4 Challenge: Implement a test to screenshots a page

3.5 Solution: Implement a test to screenshot a page

4. Running Tests on Microsoft Playwright Testing Service
4.1 What is Microsoft Playwright Testing(MPT) service?
* A cloud-based service for running Playwright tests at scale
* Provides seamless integration with CI/CD pipelines
* Ensures consistent test execution across environments

Key Differences between MS playwright Framework vs MS playwright testing service:
MS playwright Framework
* A powerful open-source automation framework
* Run tests locally or on self-managed infrastructure
* Requires manual setup for parallel execution and scaling
MS playwright testing service
* A cloud-based test execution platform
* Manages infrastructure automatically
* Provides built-in parallel execution and debugging tools
Benefits of Using MPT
* No need to maintain test infrastructure
* Parallel test execution for faster feedback
* Built-in debugging tools and logs
Who SHould use MPT
* QA teams looking for scalable test execution
* Developers who want instant feedback on test runs
* CI/CD pipeline engineers automating deployment checks

4.2 Creating a resource on Microsoft Azure
Go to playwright workspaces

4.3 Setting up your framework to run tests on the cloud

4.4 Running tests via CLI
npx playwright test –config=playwright.service.config.ts –workers=20

4.5 Challenge: Run tests against your local server on MPT
Execute Local Server Tests on MPT
* Your Task: configure and execute Playwright tests against your local server using the Microsoft Playwright Testing service
* Use the Playwright CLI to direct tests to your running local environment

4.6 Solution: Run tests against your local server on MPT

https://www.linkedin.com/learning/advanced-playwright-techniques-optimizing-speed-stability-and-cloud-testing/challenge-run-tests-against-your-local-server-on-mpt?autoSkip=true&resume=false&u=74413660

41172384512640
2006
```

</details>
