# Passing data from InAppBrowser to Cordova

This article on Telerik's blog: [Cross Window Communication With Cordova's InAppBrowser](http://blogs.telerik.com/appbuilder/posts/13-12-23/cross-window-communication-with-cordova's-inappbrowser) represents a way to work with **InAppBrowser** in Cordova (PhoneGap) application development.

Passing data to InAppBrowser implements via JavaScript injection, using a standard **executeScript** function of instantiated window.

A problem lies in the fact that there is no proper way to pass data *from* InAppBrowser *to* Cordova. InAppBrowser does not support **window.postMessage** method to perform a cross-window Ajax request.

[AppBuilder](http://blogs.telerik.com/appbuilder) in aforenamed article proposes to use a tricky hack - a polling loop. In a nutshell:

1. inAppBrowser's side: a data which is needed to pass from a sub-window (aka InAppBrowser) stores in localStorage.
2. Cordova's side: a loop works continuously to check a precense of a data via sending a JavaScript injection with further getting it.

***

I propose one more **solution**: instead of *step 2* you should just make a web page refreshing and sending an injection on new refreshed page loading. InAppBrowser has a **loadstop** and **loadstart** events, which let you operate your sub-window on page loading without endless loops, which can have lots of disadvantages.

```javascript
// instantiating an InAppBrowser window:
app.subWindow = window.open('your url', '_blank', 'location=no,toolbar=no');
// adding a listener for page load handling:
app.subWindow.addEventListener("loadstop", app.loadStopHandler, false);
```

Tip: to make a full web page refresing in InAppBrowser with further *loadstop* event handling, you should add a random **GET** parameter to it's current url, because *loadstop* event will not raise. This script might be runned on some event raising, it's up to your and depends on your task.

```javascript
// current URL
var curentLocation = window.location.href;
// random string creating for further operations
var randomStr = Math.random().toString(36).substring(7);
// adding an unmeaning GET param for FULL page refreshing
var newLocation = currentLocation + "&yourparamname={" + randomStr + "}";
// redirecting to a "new" web page
window.location.href = newLocation;
```


On *loadstop* (or *loadstart*, maybe) need to run a JS injection to fetch needed data from webStorage.

```javascript
// on inAppBrowser load (a snippet: function in 'app' namespace)
loadStopHandler: function() {
    // sending a JS injection string to fetch a sessionStorage element
    app.subWindow.executeScript(
        { code: "sessionStorage.getItem('message')" },
        function(data) {  // callback function
            if (data) {
                // you've passed a data from InAppBrowser to Cordova app
            }
        }
    );
}
```
***
P.S. An article is on editing stage. Later I will add more detailed information, maybe even with source examples.
Theese snippets are not structured in one source, just scraps of my thoughts :), but I have created this article anyway, because some people in comments on Telerik blog *need* a solution. I hope this will help them.
Sorry for my poor English, I can't express my thoughts like a native speaker.

