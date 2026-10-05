# Secular-ly

This is an exploit to disable the Securly filtering extension.

It was made by [@akabutnicer](https://github.com/akabutnicer) and [@ading2210](https://github.com/ading2210).

## How do I use this?

Visit one of the following links, and click the "Disable Securly" button:
- https://secular-ly.pages.dev/
- https://secular-ly.vercel.app/

Or, you can download the HTML file in the [Github releases](https://github.com/ading2210/secular-ly/releases), and open it in Chrome. If file:// urls are blocked, you may import the same file as a saved bookmark into chrome://bookmarks. 

## How does this work?

This exploit relies on two key facts. First of all, any 404 error page on securly.com contains the following Javascript code:

```js
var utmKeys = ["utm_source", "utm_medium", "utm_campaign", "utm_term", "utm_content"];

var urlParams = new URLSearchParams(window.location.search);
var cookieExpire = new Date();
cookieExpire.setMonth(cookieExpire.getMonth() + 1);
var expDate = cookieExpire.toUTCString();

utmKeys.forEach(function (key) {
  var value = urlParams.get(key);
  if (value != null) {
    document.cookie = key + "=" + encodeURIComponent(value) + "; expires=" + expDate + "; path=/; SameSite=Lax" + (window.location.protocol === "https:" ? "; Secure" : "");
  }
});
```

Basically, if you include one of the `utmKeys` in a query parameter on the URL, a cookie will be set with the value of the query param, with an expiration date of one month in the future. For instance, the page https://www.securly.com/not_a_real_page?utm_source=123, will create the cookie `utm_source=123` when visited. You can create 5 of these cookies, using the 5 different allowed `utmKeys`, and they can be up to about 4KB long (due to URL length limits).

Secondly, Securly's web server will reject any requests with headers that are too long, and it'll return a 400 error code. The limit is about 16KB of data. 

Thus, this exploit simply uses Securly's 404 error page to set 5 different cookies with the longest allowed length, totaling 20KB in data. Because cookies are included in the header of any HTTP request, this means that any request to www.securly.com will include our 20KB of cookie data. This causes every request to exceed the header length limits and fail, including any requests from the Securly Chrome extension. If the extension is unable to check whether or not the page you are visiting is blocked, it will just allow everything through. 

## I want to make my own deployment.

Clone this repository, and install the dependencies.

```
pip3 install jinja2-cli
```

Run the following command to generate the proper HTML file:

```
jinja2 index.html > out.html
```

You can now upload this file to your desired web hosting service.