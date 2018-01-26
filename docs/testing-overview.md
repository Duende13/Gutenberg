# Testing Overview

Gutenberg contains both PHP and JavaScript code, and encourages testing and code style linting for both.

## JavaScript Testing

Tests for JavaScript use [Jest](http://facebook.github.io/jest/) as the test runner. If needed, you can also use [Enzyme](https://github.com/airbnb/enzyme) for React component testing.

Assuming you've followed the instructions above to install Node and project dependencies, tests can be run from the command-line with NPM:

```
npm test
```

Code style in JavaScript is enforced using [ESLint](http://eslint.org/). The above `npm test` will execute both unit tests and code linting. Code linting can be verified independently by running `npm run lint`.

To run unit tests only, without the linter, use `npm run test-unit` instead.

### Snapshot testing

This is an overview of [snapshot testing] and how to best leverage snapshot tests.

#### TL;DR Broken snapshots

When a snapshot test fails, it just means that a component's rendering has changed. If that was unintended, then the snapshot test just prevented a bug 😊

However, if the change was intentional, follow these steps to update the snapshot. Run the following to update the snapshots:
   ```sh
   # --testPathPattern is optional but will be much faster by only running matching tests
   npm run test-unit -- --updateSnapshot --testPathPattern path/to/tests
   ```
1. Review the diff and ensure the changes are expected and intentional.
1. Commit.

#### What are snapshots?

Snapshots are just a representation of some data structure generated by tests. Snapshots are stored in files and committed alongside the tests. When the tests are run, the data structure generated is compared with the snapshot on file.

It's very easy to make a snapshot:

```js
test( 'foobar test', () => {
  const foobar = { foo: 'bar' };

  expect( foobar ).toMatchSnapshot();
} );
```

This is the produced snapshot:

```js
exports[`test foobar test 1`] = `
  Object {
    "foo": "bar",
  }
`;
```

You should never create or modify a snapshot directly, they are generated and updated by tests.

#### Advantages

* Trivial and concise to add tests.
* Protect against unintentional changes.
* Simple to work with.
* Reveal internal structures without running the application.

#### Disadvantages

* Not expressive.
* Only catch issues when changes are introduced.
* Are problematic for anything non-deterministic.

#### Use cases

Snapshot are mostly targeted at component testing. They make us conscious of changes to a component's structure which makes them _ideal_ for refactoring. If a snapshot is kept up to date over the course of a series of commits, the snapshot diffs record the evolution of a component's structure. Pretty cool 😎

```js
import { shallow } from 'enzyme';
import SolarSystem from 'solar-system';
import { Mars } from 'planets';

describe( 'SolarSystem', () => {
  test( 'should render', () => {
    const wrapper = shallow( <SolarSystem /> );

    expect( wrapper ).toMatchSnapshot();
  } );

  test( 'should contain mars if planets is true', () => {
    const wrapper = shallow( <SolarSystem planets /> );

    expect( wrapper ).toMatchSnapshot();
    expect( wrapper.find( Mars ) ).toHaveLength( 1 );
  } );
} );
```

Reducer tests are also be a great fit for snapshots. They are often large, complex data structures that shouldn't change unexpectedly, exactly what snapshots excel at!

#### Working with snapshots

You might be blindsided by CI tests failing when snapshots don't match. You'll need to [update snapshots] if the changes are expected. The quick and dirty solution is to invoke Jest with `--updateSnapshot`. In Calypso you can do that as follows:

```sh
npm run test-unit -- --updateSnapshot --testPathPattern path/to/tests
```

`--testPathPattern` is not required, but specifying a path will speed things up by running a subset of tests.

It's a great idea to keep `npm run test-unit:watch` running in the background as you work. Jest will run only the relevant tests for changed files, and when snapshot tests fail, just hit `u` to update a snapshot!

#### Pain points

Non-deterministic tests may not make consistent snapshots, so beware. When working with anything random, time-based, or otherwise non-deterministic, snapshots will be problematic.

Connected components are tricky to work with. To snapshot a connected component you'll probably want to export the unconnected component:

```js
// my-component.js
export { MyComponent };
export default connect( mapStateToProps )( MyComponent );

// test/my-component.js
import { MyComponent } from '..';
// run those MyComponent tests…
```

The connected props will need need to be manually provided. This is a good opportunity to audit the connected state.

#### Best practices

If you're starting a refactor, snapshots are quite nice, you can add them as the first commit on a branch and watch as they evolve.

Snapshots themselves don't express anything about what we expect. Snapshots are best used in conjunction with other tests that describe our expectations, like in the example above:

```js
test( 'should contain mars if planets is true', () => {
  const wrapper = shallow( <SolarSystem planets /> );

  // Snapshot will catch unintended changes
  expect( wrapper ).toMatchSnapshot();

  // This is what we actually expect to find in our test
  expect( wrapper.find( Mars ) ).toHaveLength( 1 );
} );
```

[`shallow`](http://airbnb.io/enzyme/docs/api/shallow.html) rendering is your friend:

> Shallow rendering is useful to constrain yourself to testing a component as a unit, and to ensure that your tests aren't indirectly asserting on behavior of child components.

It's tempting to snapshot deep renders, but that makes for huge snapshots. Additionally, deep renders no longer test a single component, but an entire tree. With `shallow`, we snapshot just the components that are directly rendered by the component we want to test.

### Code Coverage

Code coverage is measured for each PR using the [codecov.io](https://codecov.io/gh/WordPress/gutenberg) tool.
[Code coverage](https://en.wikipedia.org/wiki/Code_coverage) is a way of measuring the amount of code covered by the tests in the test suite of a project.  In Gutenberg, it is currently measured for JavaScript code only.

## End to end Testing

If you're using the built-in [local environment](https://github.com/WordPress/gutenberg/blob/master/CONTRIBUTING.md#local-environment), you can run the e2e tests locally using this command:

```bash
npm run test-e2e
```

or interactively

```bash
npm run test-e2e:watch
```

If you're using another local environment setup, you can still run the e2e tests by overriding the base URL and the default WP username/password used in the tests like so:

```bash
cypress_base_url=http://my-custom-basee-url cypress_username=myusername cypress_password=mypassword npm run test-e2e
```

## PHP Testing

Tests for PHP use [PHPUnit](https://phpunit.de/) as the testing framework. If you're using the built-in [local environment](https://github.com/WordPress/gutenberg/blob/master/CONTRIBUTING.md#local-environment), you can run the PHP tests locally using this command:

```bash
npm run test-php
```

Code style in PHP is enforced using [PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer). It is recommended that you install PHP_CodeSniffer and the [WordPress Coding Standards for PHP_CodeSniffer](https://github.com/WordPress-Coding-Standards/WordPress-Coding-Standards#installation) ruleset using [Composer](https://getcomposer.org/). With Composer installed, run `composer install` from the project directory to install dependencies. The above `npm run test-php` will execute both unit tests and code linting. Code linting can be verified independently by running `npm run lint-php`.

To run unit tests only, without the linter, use `npm run test-unit-php` instead.

[snapshot testing]: https://facebook.github.io/jest/docs/en/snapshot-testing.html
[update snapshots]: https://facebook.github.io/jest/docs/en/snapshot-testing.html#updating-snapshots