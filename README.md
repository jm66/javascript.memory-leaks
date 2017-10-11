Javascript Memory Leaks Examples
================================

Example of Javascript Memory leaks with explanations and fixes

There is an presentation about [Memory leaks in Javascript](https://slides.com/xufocoder/memory-leaks-in-the-javascript-4)

# Tables of contents

* [Global variables](#global-variables)
  * [Cache service](#cache-service)
* [Callbacks](#callbacks)
  * [Endpoint status](#endpoint-status)
* [Not killed timers](#not-killed-timers)
  * [Gonzalo Ruiz de Villa Example](#gonzalo-ruiz-de-villa-example)

## Global variables

Scripts create allocate objects intoz global scope 

### Cache service

**What is a memory leak:** after removing `CacheService` instance memory must be released 

```js
(function(window) {
  cache = []

  window.CacheService = function() {
    return {
      cache: function(value) {
        cache.push(new Array(1000).join('*'))
        cache.push(value)
      }
    }
  }
})(window)

var service = new window.CacheService()

for (let i=0; i < 99999; i++) {
  service.cache(i)
}

service = null
```

**How to fix:** move `cache` variable to `CacheService` scope like the followings:

```js
(function(window) {
  window.CacheService = function() {
    var cache = []

    return {
      cache: function(value) {
        cache.push(new Array(1000).join('*'))
        cache.push(value)
      }
    }
  }
})(window)

var service = new window.CacheService()

for (let i=0; i < 99999; i++) {
  service.cache(i)
}

service = null
```


## Callbacks

### Endpoint status

**What is a memory leak:** every interval tick create event listeners

```js
function checkStatus() {
  fetch('/endpoint').then(function(response) {
    var container = document.getElementById("container"); 
    container.innerHTML = response.status;

    container.addEventListener("mouseenter", function mouseenter() { 
      container.innerHTML = response.statusText;
    });
    container.addEventListener("mouseout", function mouseout() { 
      container.innerHTML = response.status;
    });
  })
}

setInterval(checkStatus, 100);
```

**How to fix:** extract all callbacks for event listeners and 

```js
var container = document.getElementById("container");
var status = {
  code: null,
  text: null
}

container.addEventListener("mouseenter", function() { 
  container.innerHTML = status.code;
});

container.addEventListener("mouseout", function() { 
  container.innerHTML = status.text;
});

function processResponse(response) {
   status.code = response.status
   status.text = response.statusText
}

function checkStatus() {
  fetch('/endpoint').then(processResponse)
}

setInterval(checkStatus, 100)
```

## Not killed timers

### Gonzalo Ruiz de Villa Example

```js

var strangeObject = { 
  callAgain: function () {
    var ref = this 
    var val = setTimeout(function () {
      ref.callAgain() 
    }, 50) 
  } 
} 

strangeObject.callAgain() 
strangeObject = null
```
