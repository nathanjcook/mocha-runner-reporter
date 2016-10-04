Test Runner
=============
- [Description](#description)
- [Usage](#usage)
- [Structure](#structure)
- [Report Generation](#report-generation)

### Description ###

test-runner is a util which lets you easily run a set of mocha tests programatically, and report the results.

The test runner automatically generates `-h | --help` parameter helper outputs using the data provided during setup. The generated help output displays each available argument that can be used with the test suite, along with any provided typing and description for the options, as well as provided usage descriptions or notes.

Test runs return a Promise containing the results to allow you to easily perform custom reporting alongside (or instead of) the built in reporting tool.


### Usage ###

```
> node run.js --your arguments --go --here
```

```javascript
'use strict';
const Runner = require('mocha-runner-reporter');
const Reporter = Runner.Reporter; // Optional

const params = {...}; // See Structure
const reportData = {...} // See Structure

const runner = new Runner(['path/to/tests/', '/actual/test/file.js'], ['pattern-to-ignore', 'file/to/ignore.js'], params);

runner.run()
	.then(results => {
		// Optional custom reporting using the raw data
		const report = runner.generateReport('test title', results);

		// Optional reporting using the generated report
		const reporter = new Reporter(reportData.provider, reportData.email, reportData.password, reportData.alias);
		return reporter.sendEmail(reportData.recipients, reportData.subject, report);
	})
```


### Structure ###

#### Constructor parameters

The test runner's constructor takes

```javascript
const params = {
	title: 'Test Runner Tester',
	description: 'Tests the thing which tests the tests',
	notes: [
		'Foo bar baz',
		'Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco'
	],
	args: [
		{
			aliases: ['r', 'run', 'runner'],
			description: 'does the thing',
			type: 'string',
			function: value => {
				config.option = value;
			}
		},
		{
			aliases: ['n', 'num', 'number'],
			description: 'doesn\'t do the thing',
			type: 'number',
			function: value => {
				if (value === something) {
					doSomething();
				} else {
					doSomethingElse();
				}
			}
		},
		{
			aliases: ['useless'],
			description: 'returns the supplied value(s), but with a typo',
			type: ['string','string[]']
		},
		{
			aliases: ['d'],
			description: 'this one has a really long description, because sometimes you need them.'
		}
	]
}
```
Which generates a helper output for the test suite

```
> node run.js -h

┌────────────────────┐
│                    │
│ Test Runner Tester │
│                    │
└────────────────────┘


Description:

   Tests the thing which tests the tests


Options:

   -r, --run, --runner {string}    does the thing
   -n, --num, --number {number}    doesn't do the thing
   --useless {string|string[]}     returns the supplied value(s), but with a typo
   -t, --timeout {number}          Time in milliseconds to wait before a test is counted as failing (Defaults to 300000)
   -s, --slow {number}             Time in milliseconds to wait before a test is counted as slow (Defaults to 10000)
   -R, --reporter {string}         Mocha reporter to use (Defaults to spec)


Notes:

   Foo bar baz

   Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore
   et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco
```
#### Email Reporter data

The test runner also includes an optional promise based wrapper around nodemailer, which also simplifies the setup and sending of emails.
To use the reporter, you need to have the following data (not required to be in an object as per the [usage](#usage) section)

```javascript
const reportData = {
	provider: 'gmail',            // See here for supported providers - https://github.com/nodemailer/nodemailer-wellknown#supported-services
	email: 'exmaple@gmail.com',   // The email address to send from
	password: '<password>',       // Password for the above email address
	alias: 'Test Reporter',       // [Optional] Alias to use for the sent email
	recipients: [],               // Array of recipients to use for the sent report
	subject: 'Test Results'       // Subject to use for the email
};
```

### Report Generation ###

The runner will also let you generate a report from the data gathered during the test run. The report can be generated by using the `.generateReport()` function of the runner.

```javascript
runner.run().then(results => {
	const report = runner.generateReport('test title', results);
	console.log(report);
})
```

```
/***************\

   test title
  Test Results

\***************/

Overview:
- Test Suites Ran : 2
- Total Passes    : 8 (72.72%)
- Total Failures  : 3 (27.27%)
- Total Skipped   : 0 (0%)
- Total Duration  : 1m25s
- Start Time      : Mon Sep 19 2016 10:08:06 GMT+0100 (BST)

‖==============================
‖ Password Policy
‖==============================

- File       : /path/to/some/file.js
- Passes     : 0 (0%)
- Failures   : 1 (100%)
- Skipped    : 0 (0%)
- Duration   : 0ms

|--------------------
| Failures:
|--------------------

Failure 1:

Name     : "before all" hook: Initiate the browser instance
Duration : NaN
Error    : RuntimeError
     (UnknownError:13) An unknown server-side error occurred while processing the command.
     Problem: unknown error: cannot determine loading statusfrom disconnected: received Inspector.detached event

     Callstack:
     -> url("https://test.example.com")


     at /path/to/some/file.js:12:34
     at ...

=============================================

‖==============================
‖ Test signing in
‖==============================

- File       : /path/to/other/file.js
- Passes     : 8 (80%)
- Failures   : 2 (20%)
- Skipped    : 0 (0%)
- Duration   : 1m25s

|--------------------
| Failures:
|--------------------

Failure 1:

Name     : should fail to sign in with an incorrect valid user id
Duration : 832ms
Error    : expected '' to equal 'Sample Text.'

     at /path/to/other/file.js:12:34
     at ...

------------------------------

Failure 2:

Name     : should change the password via My Profile
Duration : 4s 652ms
Error    : RuntimeError
     (NoSuchWindow:23) A request to switch to a different window could not be satisfied because the window could not be found.
     Problem: no such window: target window already closed from unknown error: web view not found

     Callstack:
     -> url("https://test.example.com/profile")

     at /path/to/other/file.js:56:10
     at ...

=============================================
```
