<!--
{
"name" : "unit-testing-angular-controller-jasmine",
"version" : "0.1",
"title" : "Unit Testing AngularJS Controller Using Jasmine",
"description": "See the power of Jasmine in action with unit testing in Angular.",
"freshnessDate" : 2013-04-06,
"homepage" : "http://sravi-kiran.blogspot.com/2013/04/UnitTestingAngularJsControllerUsingJasmine.html",
"canonicalSource" : "http://sravi-kiran.blogspot.com/2013/04/UnitTestingAngularJsControllerUsingJasmine.html",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

# Overview

Building and using single page JavaScript applications is fun. It brings richness on client side without having to install any external plugins. While building these applications, we write a lot of JavaScript code to run on client side. Releasing such large amount of code to client without testing will be a sin. We might leave the code with some unnoticed bugs if we do not test.

Right from the [first post on Angular JS](http://sravi-kiran.blogspot.com/2013/02/EasyTwoWayDataBindingInHtmlPagesWithAngularJS.html), I have mentioned several times that unit testability is one of its primary design goals. [Tutorial series on Angular’s website](http://docs.angularjs.org/tutorial) demonstrates [end-to-end testing](http://docs.angularjs.org/guide/dev_guide.e2e-testing) using Jasmine’s BDD style. I cannot have sound sleep unless I write some tests for the [AngularShoppingCart application](http://sravi-kiran.blogspot.com/2013/02/EasyTwoWayDataBindingInHtmlPagesWithAngularJS.html). In this post, we will see how a controller can be unit tested using [Jasmine](https://github.com/pivotal/jasmine).

_Note: If you are looking for a tutorial on unit testing using QUnit, I have a blog post on it: [Unit Testing Angular JS Controller Using QUnit and Sinon](http://sravi-kiran.blogspot.com/2013/06/UnitTestingAngularJsControllerUsingQUnitAndSinon.html)_

If you haven’t followed earlier posts, take a look at the code on [GitHub](https://github.com/pivotal/jasmine).

<!-- @section -->

## Jasmine and setting up

Jasmine is a [BDD (Behaviour Driven Development)](http://en.wikipedia.org/wiki/Behavior_driven_development) framework for testing JavaScript code. It doesn’t depend on any other JavaScript library. It has nice support to create spies (mocks), which helps to easily get rid of some concrete dependencies. Jasmine can be used to unit test anything written in JavaScript.

In order to run jasmine tests and view their results, we need to create an HTML page. Following references should be added to the page:

1. Jasmine core library
2. Jasmine HTML library
3. Jasmine style sheet
4. Source files of the script to be tested, which is also referred as System under test (SUT)
5. File containing unit tests for the above source
6. Any other library (like jQuery, jQuery UI, Angular) on which the source depends

As we will be testing code written using Angular JS, we should include angular-mocks.js library, which has mock services to replace some of the most commonly used services.

In body tag of the HTML page, a couple of statements are to be added to bootstrap Jasmine. Following is the template of a Jasmine spec runner page:

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
  "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
    <title>Jasmine Test Runner</title>

    <!-- Jasmine stylesheet -->
    <link rel="stylesheet" type="text/css" href="../../Styles/Jasmine.css">

    <!-- Jasmine core and HTML -->
    <script type="text/javascript" src="../../Scripts/jasmine.js"></script>
    <script type="text/javascript" src="../../Scripts/jasmine-html.js"></script>
    <script type="text/javascript" src="../../Scripts/jasmine.async.js"></script>

    <!-- JavaScript libraries on which source depends -->
    <script type="text/javascript" src="../../Scripts/angular.js"></script>
    <script type="text/javascript" src="../../Scripts/angular-mocks.js"></script>

    <!-- Script source to test and other files on which source depends -->
    <script type="text/javascript" src="../../Scripts/app/ShoppingModule.03.js"></script>
    <script type="text/javascript" src="../../Scripts/app/ShoppingCartController.03.js"></script>
    <script type="text/javascript" src="../../Scripts/app/CartCheckoutController.js"></script>

    <!-- Test script -->
    <script type="text/javascript" src="ShoppingCartControllerSpec.js"></script>
    <!--<script type="text/javascript" src="CartCheckoutTestSpec.js"></script>-->
</head>
<body>
    <script type="text/javascript">
        jasmine.getEnv().addReporter(new jasmine.TrivialReporter());
        jasmine.getEnv().execute();
    </script>
</body>
</html>
```

<!-- @task, "text" : "Add the template to bootstrap Jasmine to a project you are working on."-->

<!-- @section -->

## Specs and Suits

Jasmine has a set of functions defined to create suits, specs and assertions. Following listing explains each of them in brief:

1. [describe](http://pivotal.github.io/jasmine/#section-Suites:_<code>describe</code>_Your_Tests): Used to define a suite. Accepts two parameters, name of the suite and a function containing statements to be contained in a suite
2.   [it](http://pivotal.github.io/jasmine/#section-It&rsquo;s_Just_Functions): Used to define a spec. Accepts a name and function containing logic to be executed. The logic contains assertions to evaluate the behaviour
3.   [beforeEach](http://pivotal.github.io/jasmine/#section-Setup_and_Teardown): To set up dependencies before any spec under a suite runs
4.   [afterEach](http://pivotal.github.io/jasmine/#section-Setup_and_Teardown): To clear any dependency after being used in the test cases
5.   [expect](http://pivotal.github.io/jasmine/#section-Expectations): A function that accepts an object to be asserted
6.   [matcher](http://pivotal.github.io/jasmine/#section-Included_Matchers): Jasmine defines several matchers like toBe, toHaveBeenCalled, toEqual to perform a Boolean match between actual and expected values. We can write custom matchers too<

In-depth discussion about above topics is beyond the scope of this article. Refer to [Jasmine’s documentation](http://pivotal.github.com/jasmine/) for more details.

<!-- @section -->

## Unit testing ShoppingCartController

Let’s start testing the functions defined in ShoppingCartController.

Dependencies of the controller are clearly visible from the signature. As we need to inspect behaviour of the controller in isolation, we must mock these services. Following is the signature of ShoppingCartController:

```javascript
function ShoppingCartCtrl($scope, $window, shoppingData, shared) {
}
```

As these services will be used across specs, it is better to create them globally and initialize them in beforeEach block. Since we will not hit the actual service, we need to use some static data to make the job of testing AJAX calls easier.

```javascript
var shoppingCartStaticData = [
    { "ID": 1, "Name": "Item1", "Price": 100, "Quantity": 5 },
    { "ID": 2, "Name": "Item2", "Price": 55, "Quantity": 10 },
    { "ID": 3, "Name": "Item3", "Price": 60, "Quantity": 20 },
    {"ID": 4, "Name": "Item4", "Price": 65, "Quantity": 8 }
];

describe("ShoppingCartCtrl", function () {

    //Mocks
    var windowMock, httpBackend, _shoppingData, sharedMock;

    //Controller
    var ctrl;

    //Scope
    var ctrlScope;

    //Data
    var storedItems;

    //Loading shopping module
    beforeEach(function () {
        module("shopping");
    });

    beforeEach(inject(function ($rootScope, $httpBackend, $controller, shoppingData) {
        //Mock the services here
    }

});
```

The second beforeEach block will set-up all dependencies for the controller. So, it needs a number of services to perform its job. We will discuss their importance in shortly.

<!-- @section -->

## Resolving Dependencies

First and most important dependency of the controller is the $scope service. We need to create our own scope and pass it as a parameter while creating object of the controller. Using $rootScope, it is very easy to create our own scope:</span>

```javascript
ctrlScope = $rootScope.$new();
```

Second dependency is the $window service. As we are using href property of location alone, we can create a custom object with this property alone.

```javascript
windowMock = { location: { href: ""} };
```

Third dependency shoppingData service is a wrapper to call backend data services. It used another service, $http to send AJAX requests. Angular JS team has made our job easy by creating $httpBackend, a mock for $http. $httpBackend provides a nice interface to send our own response when an AJAX request is made.

shoppingData service has three functions: getAllItems, addAnItem and removeItem. Let’s create spies for these functions as follows:

```javascript
httpBackend = $httpBackend;
_shoppingData = shoppingData;
spyOn(shoppingData, 'getAllItems').andCallThrough();
spyOn(shoppingData, 'addAnItem').andCallThrough();
spyOn(shoppingData, 'removeItem').andCallThrough();
```

The function andCallThrough delegate calls to the actual implementations. The function getAllItems sends an HTTP GET request to the service. Following statement configures a custom response on $httpBackend when Angular detects any such request:

```javascript httpBackend.expectGET('/api/shoppingCart/').respond(storedItems);
```javascript

Fourth and final dependency is the shared service. Following snippet creates a mock shared service with a spy for setCartItems, the only function used in ShoppingCartCtrl:

```javascript
sharedMock = {
    setCartItems: jasmine.createSpy('setCartItems')
};
```

Now that we have all the mocks ready, let’s create an object of the controller.

```javascript
ctrl = $controller(ShoppingCartCtrl, { $scope: ctrlScope, $window: windowMock, shoppingData: _shoppingData, shared: sharedMock });
```

<!-- @task, "text" : "Create an object of the controller."-->

<!-- @section -->

## Testing behaviour of the controller

On creation of the controller, it calls getAllItems function of shoppingData service to fetch details of all items. The test for this behaviour should check if the right function is called and if it sets value to the items property. Following test shows this:

```javascript
it("Should call getAllItems function on creation of controller and set items property", function () {
    expect(_shoppingData.getAllItems).toHaveBeenCalled();
    httpBackend.flush();
    expect(ctrlScope.items.length).not.toBe(0);
});
```

Upon calling addItem function of the controller, it calls addAnItem function of shoppingData service. As it makes an HTTP post request to the service, $httpBackend should be configured to respond when it finds a post request. Test looks as follows:

```javascript
it("Should call addAnItem function of the shoppingData service", function () {
    httpBackend.expectPOST('/api/shoppingCart/', {}).respond(storedItems.push({ "Id": 5, "Name": "Item5", "Price": 70, "Quantity": 10 }));
    ctrlScope.addItem({});
    expect(_shoppingData.addAnItem).toHaveBeenCalled();
    httpBackend.flush();
    expect(storedItems.length).toBe(5);
});
```

The function removeItem can also be tested in similar way. But, what if a request fails? The $errorMessage property of scope should be assigned with a friendly error message. A request can be forced to fail by passing a JavaScript object literal with an error status code to $httpBackend. Let’s see this in action:

```javascript
it("Should assign an error message", function () {
    httpBackend.expectDELETE('/api/shoppingCart/1').respond({ status: 500 });
    ctrlScope.removeItem(1);
    expect(ctrlScope.errorMessage).not.toBe("");
});
```

<span style="font-family: &quot;Calibri&quot;,&quot;sans-serif&quot;; font-size: 11.0pt; line-height: 115%; mso-ansi-language: EN-IN; mso-ascii-theme-font: minor-latin; mso-bidi-font-family: &quot;Times New Roman&quot;; mso-bidi-language: AR-SA; mso-bidi-theme-font: minor-bidi; mso-fareast-font-family: Calibri; mso-fareast-language: EN-US; mso-fareast-theme-font: minor-latin; mso-hansi-theme-font: minor-latin;">mySortFunction is used to convert numeric value to number. We can test this function by passing a number in the form of a string and checking if it returned a number to us. We need to set the property sortExpression before calling the function.</span>

<pre class="brush: jscript">it("Should return a number when a number is passed in", function () {
    var item = { "Number": "123" };
    ctrlScope.sortExpression = "Number";
    var numVal = ctrlScope.mySortFunction(item);
    expect(typeof numVal).toBe("number");
});
</pre>

The totalPrice function is very easy to test, as we need to just check if it sets some value to the returned variable.

On click of Purchase Items link on the page, the user has to be navigated to CheckoutItems view and setCartItems function of shared service should be called to pass the items array to the second view. As we are setting navigation URL to window.location.href, for which we created a mock, the test has to check if this property is set to some value. Following test verifies these functionalities:

```javascript
it("Should set value in shared and value of href set", function () {
    ctrlScope.items = storedItems;
    ctrlScope.purchase();

    expect(sharedMock.setCartItems).toHaveBeenCalled();
    expect(windowMock.location.href).not.toBe("");
});
```

Now view the test runner page on browser. All the tests should be passed. I encourage you to play a bit with the code and tests and check the result of test after the changes. This way, we will get to know more about Jasmine's behavior as well as unit testing.

<!-- @task, "text" : "Change some of the code to break the tests and verify that they fail."-->

<!-- @task, "hasDeliverable" : true, "text" : "Write another test to use and submit it here."-->

You can download the code including unit tests from the following GitHub repo: [AngularShoppingCart](https://github.com/sravikiran/AngularShoppingCart)

Happy coding!
