---
layout: post
title: Protractor入门(一)
category: 技术
comments: true
---

Protractor是一个专为AngularJS设计的端到端测试框架. 本文将介绍使用Protractor做UI测试时的一些基础的操作.

`End-to-end test framework` `AngularJS`

**Protractor** is an end-to-end test framework for AngularJS applications. Protractor runs tests against your application running in a real browser, interacting with it as a user would. 

- **You don't need to add waits or sleeps to your test.** 
  Protractor can communicate with your AngularJS app automatically and execute the next step in your test the moment the webpage finishes pending tasks, so you don't have to worry about waiting for your test and webpage to sync.
- **It supports Angular-specific locator strategies.** 
  (e.g., binding, model, repeater) but also native WebDriver locator strategies (e.g., ID, CSS selector, XPath).
- **It is easy to set up page objects.**
  Protractor dose not execute WebDriver commands until an action is needed(e.g., get, sendKeys, click). This way you can set up page objects so that tests can manipulate page elements without touching the HTML.
- **It uses Jasmine.**
  Protractor allows tests to be organized based on [Jasmine](http://jasmine.github.io/), thus allowing you to write both unit and functional tests on Jasmine.
- **Integrated to CI.**
  It runs on real browsers and headless browsers.


### Setup
```sequence
npm install -g protractor
```
This will install two command line tools, `protractor` and `webdriver-manager`. Try running `protractor --version` to make sure it's working.
The `webdriver-manager` is a helper tool to easily get an instance of a Selenium Server running. Use it to download the necessary binaries with:

```sequence
webdriver-manager update
```
Start up a server with:

```sequence
webdriver-manager start 
```
This will start up a Selenium Server and will output a bunch of info logs. Your Protractor test will send requests to this server to control a local browser. You can see informantion about the status fo the server at http://localhost:4444/wd/hub.

### Write a test
[Protractor-Demo](http://juliemr.github.io/protractor-demo/)

- **Spec file**

```javascript
// spec.js

describe('Protractor Demo App', function() {

    it('should have a title', function() {
 
        browser.get('http://julimer.github.io/protractor-demo/');
 
        expect(browser.getTitle()).toEqual('Super Calculator');
 
    });
});
```
The `describe` and `it` syntax is from the Jasmine framework.

- **Configuration file**

``` javascript
// conf.js

exports.config = {

    framework: 'jasmine',

    seleniumAddress: 'http://localhost:4444/wd/hub',

    specs: ['spec.js']
}
```

Run the test with

```sequence
protractor conf.js
```



### Interacting with elements
```javascript
// spec.js

describe('Protractor Demo App', function() {

    it('should add one and two', function() {

        browser.get('http://julimer.github.io/protractor-demo/');

        element(by.model('first')).sendKeys(1);        # <input type=text ng-model="first">

        element(by.model('second')).sendKeys(2);

        element(by.id('gobutton')).click();            # <button id="gobutton">

        expect(element(by.binding('latest')).getText()).toEqual('3');       // {{latest}}
    });
});
```
The globals `element` and `by` are created by Protractor. The `element` function is used for finding HTML elements and returning an ElementFinder object. The `by` object creates Locators.

- **Locators**

```javascript
by.css('.myclass')    # find an element using a css selector

by.id('myid')         # find an element with the given id

by.model('name')      # find an element with a certain ng-model

by.binding('bindingname')         # find an element bound to the given variable
```
For Protractor-specific locators, see the [Protractor API: ProtractorBy](http://www.protractortest.org/#/api?view=ProtractorBy)

- **Actions**

```javascript
var el = element(locator)

el.click();           # Click on the element

el.sendKeys('my text');        # Send keys to the element (usually an input)

el.clear();           # Clear the text in an element (usually an input)              

el.getAttribute('value');         # Get the value of an attribute, for example, get the value of an input
```

```javascript
var el = element(locator);

el.getText().then(function(text) {

    console.log(text);
});
```

See a full list [here](http://www.protractortest.org/#/api?view=webdriver.WebElement).

- **Finding Multiple Elements**

```javascript
element.all(by.css('.selector')).then(fucniton(elements) {

    //elements is an array of ElementFinders.

});
```

`element.all()` has several helper functions:

```javascript
element.all(locator).count();         # Number of elements.

element.all(locator).get(index);        # Get by index (starting at 0).

# First and last.
element.all(locator).first();
element.all(locator).last();
```

- **Finding Sub-Elements**

To find sub-elements, simply chain element and element.al function together.
Using a single locator to find:

```javascript
# an element:
element(by.css('some-css'));

# a list fo elements:
element.all(by.css('some-css'));
```
Using chained locators to find:

```javascript
# a sub-element:
element(by.css('some-css')).element(by.tagName('tag-within-css'));

# to find a list of sub-elements:
element(by.css('some-css')).all(by.tagName('tag-within-css'));
```

You can chain get/first/last as well like so:

```javascript
element.all(by.css('some-css')).first().element(by.tagName('tag-within-css'));

element.all(by.css('some-css')).get(index).element(by.tagName('tag-within-css'));

element.all(by.css('some-css')).first().all(by.tagName('tag-within-css'));
```

- **Lists of elements**

```javascript
// spec.js
describe('Protractor Demo App', function() {

    var firstNumber = element(by.model('first'));
    var secondNumber = element(by.model('second'));
    var goButton = element(by.id('gobutton'));
    var lastestResult = element(by.binding('latest'));
    var history = element.all(by.repeater('result in memeory'));
    
    function add(a,b) {
        firstNumber.sendKeys(a);
        secondNumber.sendKeys(b);
        goButton.click();
    }
    
    beforeEach(function() {
        browser.get('http://juliemr.github.io/protractor-demo/')
    });
    
    it('should have a history', function() {
        add(1, 2);
        add(3, 4);
        expect(history.count()).toEqual(2);
        expect(history.last().getText()).toContain('1 + 2');
    });
});
```

`element.all` returns an ElementArrayFinder. ElementArrayFinder also has methods `each`, `map`, `filter`, and `reduce` which are analogous to JavaScript Array methods. [Read API for more details](http://www.protractortest.org/#/api?view=ElementArrayFinder).

### Writing multiple scenarios

See detail code in github: 
`beforeEach` is run before every `it` block.

### Changing the configuration

Run tests on more than one browser at once. See more details [here](https://github.com/angular/protractor/blob/master/docs/referenceConf.js).

```javascript
// conf.js
exports.config = {
  
    framework: 'jasmine',
    seleniumAddress: 'http://localhost:4444/wd/hub',
    specs: ['spec.js']
    multiCapabilities: [{
        browserName: 'firefox'
    },{
        browserName: 'chrome'   
    }]
}
```



#### [Table of Content](http://www.protractortest.org/#/toc)



### Connect to browser drivers

You have *4 different built-in options/ways to connect to browser drivers*:

[See more details](http://stackoverflow.com/questions/30600738/difference-running-protractor-with-without-selenium)

1. specify `seleniumServerJar` to start selenium standalone server locally
2. specify `seleniumAddress` to connect to a running selenium server(local or remote)
3. set `sauceUser` and `sauceKey` to connect to Sauce Labs remote selenium server
4. use `directConnect` to connect to Chrome of Firefox directly. There are additional `chromeDriver` and `firfoxPath` setting that you can use to define custome Chrome driver and Firefox application binary locations.

If you have a specific configuration(s) you want your tests to turn on. Say, you have a Windows machine with IE 11 on board and Ubuntu with Firefox 35. In this case, use `seleniumAddress`.

If you are running your tests locally and your target browsers are Chrome or/and Firefox, use `directConnect`, your tests would run faster.

If you are running your tests locally and need to test against IE, Safari or other browsers, you'd go with options 1-3 (usually 1), since these browsers cannot work in "direct connect" mode.

[Demo here](https://github.com/lemon123456/Protractor)














