# Visual regression testing with Jest, Puppeteer and jest-image-snapshot

## What is visual regression testing?

Visual regression tests help catch unintentional changes to the visual rendering of a web page or component. As engineers and testers, we already do this manually, but it takes a lot of time, and as humans, are prone to overlooking breaking changes. Automating this process gives us back time to focus on more important things, and gives us added confidence when updating code.

## How it works

These tests are run in headless Chrome using [Puppeteer](https://pptr.dev/). They are written using the [Jest](https://jestjs.io/docs/en/getting-started) testing framework. [jest-puppeteer](https://github.com/smooth-code/jest-puppeteer) is added to facilitate writing and running the tests, and [jest-image-snapshot](https://github.com/americanexpress/jest-image-snapshot) does the actual image comparison and lets us know when something has changed.

These tests run automatically in two scenarios. First, on a developer's machine (inside a Docker container) before a `git push` (using [Husky](https://github.com/typicode/husky)), and again in Bitbucket Pipelines CI (which uses the same Docker container) when a pull request is created (see `bitbucket-pipelines.yml`). In most cases, breaking changes will never make it up to the remote repository, but running the tests again in CI ensures we catch _everything_.

We run the tests in a Docker container to ensure there are no differences between local and CI environments. These tests should not be run on a developer's native OS because minor rendering differences between operating systems in the browser **will** cause test failures.

## System requirements

- [NodeJS](https://nodejs.org/en/)
- [Docker CE](https://docs.docker.com/v17.12/install/)

## Setup

Install NodeJS dependencies by running `npm install`.

## Running the tests

To run the tests manually, run `npm test`. This will spin up a Docker container on your machine, start the development server _inside_ the container, and then run the tests.

For any given test, if no reference screenshot exists, one will be taken and saved alongside the test file in the `__image_snapshots__` directory. Afterwards, be sure to commit these new reference files to version control so that they may be used for future tests.

If there are any failures due to differences between current and existing screenshots, a `.png` file containing the image diff will be saved in `__diff_output__` directory alongside the existing reference screenshot for the test. These diff files are ignored by version control.

## Updating reference screenshots

If you have made _intentional_ changes to the way a component is rendered, you can update the existing reference screenshots by running `npm run test:update`. This will replace _all_ existing reference screenshots with a new version. Afterwards, commit the new and/or updated files to version control so that your changes are respected during future test runs.

## Writing tests

Test files must follow the naming pattern `*.test.js` and reside in the `src` directory, preferably alongside their component's source file. Tests are written using Jest and Puppeteer. A number of globals (e.g. `browser` and `page`) are made available by [jest-puppeteer](https://github.com/smooth-code/jest-puppeteer#api) to reduce the amount of setup and teardown code required for each test.

As a basic example, here is how you would write a test that takes a screenshot of a specific page and expects that it matches an existing reference screenshot.

```js
// landing.test.js
describe('landing page', () => {
  const url = `${global.config.baseUrl}/landing`;

  test('should render correctly', async () => {
    await page.goto(url);
    const screenshot = await page.screenshot();
    expect(screenshot).toMatchImageSnapshot();
  });
});
```

A few things to note here:

1. The variable `global.config.baseUrl` which is defined in `test/setup.js` should be used for defining the base URL, since it changes between testing environments.
1. The Puppeteer method `page.goto` instructs the browser to load our URL in the browser.
1. The Puppeteer method `page.screenshot` takes a screenshot of the entire page.
1. The `toMatchImageSnapshot` method is used to assert that our screenshot taken matches an existing reference screenshot.

As a general rule, a test should only take one screenshot. Screenshot files are named after the test description. If you need to take multiple screenshots to test different states of a component, create a separate test for testing each state.
