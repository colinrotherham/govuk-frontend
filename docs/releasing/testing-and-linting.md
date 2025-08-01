# Testing and linting

GitHub Actions lints Sass and JavaScript, runs unit and functional tests with Node.js, and generates snapshots for [visual regression testing](#visual-regression-testing-with-percy).

See the [GitHub Actions **Tests** workflow](https://github.com/alphagov/govuk-frontend/actions/workflows/tests.yml) for more information.

## Testing terminology

We use different types of tests to check different areas of our code are working as expected.

Unit tests are small, modular tests that verify a "unit" of code. We write unit tests to check our JavaScript logic, particularly 'background logic' that does not heavily rely on [Document Object Model (DOM)](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) interaction (where functional tests may be better suited).

Functional tests verify the output of an action and do not check the intermediate states of the system when performing that action. We write functional tests to check component interactions have the expected results, so we also refer to these as component tests. We also write functional tests to check that our Nunjucks code outputs expected HTML, and we refer to these as our Nunjucks tests.

[Snapshot tests](https://facebook.github.io/jest/docs/en/snapshot-testing.html) are used for preventing unintended changes to our component markup. When the snapshot test runs, it compares the previously captured snapshot to the current markup.

Visual regression tests help us check for any unintended visual changes to our components. We use [Percy](https://percy.io/) to generate and store screenshots of our components.

## Running linting and tests

### Running all tests locally

To test the whole codebase, run:

```shell
npm test
```

This will compile JavaScript and Sass, including documentation, then trigger [Jest](https://github.com/facebook/jest) for our testing.

See [Tasks](../contributing/tasks.md) for details of what `npm test` does.

### Running individual tests

You can run a subset of the test suite that only tests components by running:

```shell
npx jest packages/govuk-frontend/src/govuk/components/button
```

Note: There's a watch mode that keeps a testing session open waiting for changes that can be used with:

```shell
npx jest --watch packages/govuk-frontend/src/govuk/components/button
```

### Running all linting checks locally

To lint the whole codebase, run:

```shell
npm run lint
```

This will run the following checks:

1. [EditorConfig](https://editorconfig.org)
2. [Prettier](https://prettier.io)
3. [ESLint](https://eslint.org) (using [JavaScript Standard Style](https://standardjs.com))
4. [Stylelint](https://stylelint.io) (using [GDS Stylelint Config](https://github.com/alphagov/stylelint-config-gds))

Where possible, issues will be fixed automatically on `git commit` using [Husky](https://typicode.github.io/husky/) and [`.lintstagedrc.js`](/.lintstagedrc.js).

To commit changes without automatic checks use `git commit --no-verify`.

### Running only EditorConfig linting

```shell
npm run lint:editorconfig
```

See [.editorconfig](/.editorconfig) for details.

### Running only Prettier linting

```shell
npm run lint:prettier
```

See [.prettierrc.js](/.prettierrc.js) for details.

### Running only Sass linting

```shell
npm run lint:scss
```

See [CSS Coding Standards](/docs/contributing/coding-standards/css.md#linting) for details.

### Running only JavaScript linting

```shell
npm run lint:js
```

See [JavaScript Coding Standards](/docs/contributing/coding-standards/js.md#formatting-and-linting) for details.

## Unit and functional tests with Node.js

We use [Jest](https://jestjs.io/), an automated testing platform with an assertion library, and [Puppeteer](https://pptr.dev/) that is used to control [headless Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome).

Tests should be written using ES modules (`*.mjs`) by default, but use CommonJS modules (`*.js`) for tests using browser [`import()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) in Puppeteer. This avoids Babel transforms until [Jest supports `import()` and ES modules](https://jestjs.io/docs/ecmascript-modules) in future.

### Component tests

We write functional tests for every component to check the output of our Nunjucks code. These are found in `template.test.js` files in each component directory. These Nunjucks tests render the component examples defined in the component yaml files, and assert that the HTML tags, attributes and classes are as expected. For example: checking that when you pass in an `id` to the component using the Nunjucks macro, it outputs the component with an `id` attribute equal to that value.

If a component uses JavaScript, we also write functional tests in a `[component name].test.js` file, for example [checkboxes.test.js](/packages/govuk-frontend/src/govuk/components/checkboxes/checkboxes.test.js). These component tests check that interactions, such as a mouse click, have the expected result.

If you want to inspect a test that's running in the browser, configure Jest Puppeteer in non-headless mode with the environment variable `HEADLESS=false` and then use [Jest Puppeteer's debug mode](https://github.com/argos-ci/jest-puppeteer/blob/main/README.md#debug-mode) to pause the test execution.

```shell
HEADLESS=false npx jest --watch src/govuk/components/tag/accessibility.test.mjs
```

You should also test component Javascript logic with unit tests, in a `[component name].unit.test.mjs` file. These tests are better suited for testing behind-the-scenes logic, or in cases where the final output of some logic is not a change to the component markup.

### Global tests

We write functional tests for checking our JavaScript exports and our global sass variables - see [all.test.mjs](/packages/govuk-frontend/src/govuk/all.test.mjs) and [components/globals.test.js](/packages/govuk-frontend/src/govuk/components/globals.test.js) for examples of global tests we run.

### Conventions

We aim to write the test descriptions in everyday language. For example, "back-link component fails to render if the required fields are not included".

Keep all tests separate from each other. It should not matter the order or amount of tests you run from a test suite.

Try and keep assertions small, so each test only checks for one thing. This makes tests more readable and makes it easier to see what's happening if a test is failing.

## Updating component snapshots

For components, the snapshots are stored in `[component-name directory]/_snapshots_`.

If a snapshot test fails, review the difference in the console. If the change is the correct change to make, run:

```shell
npm test -- -u packages/govuk-frontend/src/govuk/components/button
```

This will update the snapshot file. Commit this file separately with a commit message that explains you're updating the snapshot file and an explanation of what caused the change.

## Visual regression testing with [Percy](https://percy.io/)

We generate at least one screenshot per component. Typically, this screenshot will be of the default example, but we may also generate screenshots for visually distinct versions of default components, such as the **Start** button.

We tell Percy we want to screenshot an example by adding the `screenshot` attribute to an example in a component's YAML fixture. Percy will screenshot that example if `screenshot` is either `true` or has a `variants` attribute containing an array of valid options ('default' or 'no-js'). A `variants` array lets us take multiple screenshots of the same example under different circumstances, such as with JavaScript turned on or off.

The screenshots are public, so you can check them without logging in. A BrowserStack account is needed to approve or reject any changes (if you don't have access, ask your tech lead for help). If you're the reviewer of the pull request code, it's your responsibility to approve or request changes for any visual changes Percy highlights.

When you run the tests locally (for example, using `npm run test:screenshots --workspace @govuk-frontend/review`), Percy commands are ignored and Percy does not generate any screenshots. You will see the following message in your command line output: `[percy] Percy is not running, disabling snapshots`.

### No-JavaScript variants

We take additional screenshots of some component examples with JavaScript turned off at the point the screenshot is taken. This is to account for when JavaScript being on or off impacts how the component looks. For example, the Accordion component without JavaScript is just headings with text.

We only take no-JavaScript screenshots for components which look very different when their JavaScript is turned off. For example, the Button component's JavaScript is just a set of enhancement features and never changes anything in the DOM, so the component looks the same regardless of whether JavaScript is turned on or not.

### Non-component examples

The review app also contains screenshots of examples we build ourselves, such as an example including all our typography or all our form inputs. These examples are not automatically generated by component examples. We can generate screenshots of examples by using the attribute `screenshot: true` in the YAML file.

### PRs from forks

When Github Actions is running against a PR from a fork, the Percy secret is not available and Percy does not generate any screenshots. Other tests will continue to run as normal. You will see the following messages in the output:

```console
[percy] Skipping visual tests
[percy] Error: Missing Percy token
```

Being unable to run Percy on PRs from forks is the reason we're unable to make Percy a required check for this repo. However, we should continue to act as if a Percy check is required. If the Percy build fails, do not merge the pull request even though GitHub would let you. Continue to review the changes, and approve or reject them as you would normally.
