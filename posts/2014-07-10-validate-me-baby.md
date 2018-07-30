title: VALIDATE me baby
categories:
  - JavaScript
  - ideas
  - HTML
  - development
data:
  updated: "2015-03-13 00:00"
  comments: true
---
There are many times that I have been writing sites with the desire for the client side to call the server in a [safe](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.1) and [idempotent](http://en.wikipedia.org/wiki/Idempotence) way with a request body for sending more data than a `GET` can, or data that can't be recorded via referral logs.

Even when or if the new [Form HTTP extensions specification](http://www.w3.org/TR/form-http-extensions/) gets approval and adoption for the basic HTTP methods, we still lack a clear way to send over requests with server side impact.

<!-- more -->

## Proposal
This proposal is for browsers and servers to support a new HTTP verb `VALIDATE` and adding a new HTML form parameter `methodfor` so that browsers can safely send validation requests to the server.

## The problem
All HTTP requests besides `GET` and `HEAD` are specified to have side effects on the server. Which means that behind the scenes of my client side applications, I can only safely send over `GET` requests to request for updates on the validation of messages.

So for example if I wanted to server side validate a credit card form I would have to send over:

    https://mysite.com/checkout/pay?creditcard=4111111111111111&fname=jonathan…

Although technically, this will be a secure request at the network layer, it could be cached and/or logged at the web server.

## Advantages
The `VALIDATE` verb has two clear advantages: clear semantic reasoning and automatic validation with safe client side methods.

### Semantic reasoning
It gives the client and server the ability to know when a request is just a validation, and enables both sides to provide a better overall experience to the user.

User agents can indicate to the user when the form will be just a validation and will be technologically safe for them to send, potentially by giving a different appearance or accent to buttons. By doing so, users will have more confidence in trying out changes, for example payment buttons in a checkout flow.

#### Example:
A dynamic page could easily serve up the following form:

```
<form method="VALIDATE" methodfor="POST" >
  <fieldset>
    <legend>Purchase item</legend>
    <div class="form-row">
      <label for="my-item">Item</label>
      <input name="item-sku" placeholder="Input SKU code" id="my-item" />
    </div>
    <button>Submit</button>
  </fieldset>
</form>
```

<img src="/images/validateme/first-form.png" title="Example of how a browser may render a VALIDATE form" />

Once the user submits the form, the dynamic page could then come back with validation issues:

```
<form method="VALIDATE" methodfor="POST" >
  <fieldset>
    <legend>Purchase item</legend>
    <div class="form-row">
      <label for="my-item">Item</label>
      <input name="item-sku" value="thing" placeholder="Input SKU code" id="my-item" />
      <div class="inline-error">Sorry our SKU codes are numeric, you might find these in our catalog EG: SKU #121212;</div>
    </div>
    <button>Submit</button>
  </fieldset>
</form>
```
<img src="/images/validateme/second-form.png" title="Example of how a browser may render a VALIDATE form with errors" />

Once validation issues are resolved then the following would display:

```
<form method="POST" >
  <fieldset>
    <legend>Confirm purchase of item 1212</legend>
    <div class="form-row">
      <label for="my-item">Item</label>
      <input name="item-sku" value="1212" id="my-item" readonly />
    </div>
    <!-- This goes out to all you "law abiding citizens" out there -->
    <button>Order with obligation to pay</button>
  </fieldset>
</form>
```

<img src="/images/validateme/final-form.png" title="Example of how a browser may render a validated form before submission" />

###  Automatic validation
With `VALIDATE` forms the JavaScript could automatically send over requests for updates and new error validations via AJAX and safely know that the user will not be impacted by side effects.

Having the ability for the JavaScript to safely make requests to the server means this feature could be turned on globally; even if the client library doesn't understand the response, the error code could be used to indicate to the user of when the form is valid or not.

### Example:

Client sends:
```
VALIDATE /checkout/pay HTTP/1.0
Method-For: POST
Content-Type: application/javascript; charset=UTF-8
Content-Length: 32

{creditcard: '4111111111111111', fname: 'jonathan'…}
```

Server response:
```
HTTP/1.0 412 Precondition Failed
Date: Fri, 31 Jun 2014 23:59:59 GMT
Content-Type: text/json
Content-Length: 121212

{status: 'additional-information', form: [{type: 'string', name: 'creditcard'}, {status: 'new-additional', type: 'string', name: 'cv2'}, {type: 'string', name: 'fname'}...]}
```
When precondition failed returns, it would be able to return validation issues per field, errors for the whole form and additional fields that are required.

Wait though, what is wrong with POST?
Natively the server doesn’t expose an interface for sending body data over in a request, which means that swapping from one system to another could have side effects on the users data.


So for example lets take a site with different validate systems:

- Your payment gateway implements the parameter: `?test=1`
- Your order process uses http headers: `X-Validate-Request: 1`
- Your blogging platform uses the parameter: `?verify-request='yes'`

Client side applications can’t quickly share the same code without further adapters to each message; POSTing the data raw couldn’t be used safely as it might actually run the request that you are trying to check without the intention of the user to do so.

However, if both the payment gateway and order process implemented VALIDATE as a verb, the JavaScript and/or browser could automatically verify forms as and when it sees fit. Additionally, the blog could have the VALIDATE functionality turned off, and the server would be used in the traditional round trip manner, passing back more field when it knows the client needs them. This has the added bonus of working even with VALIDATE not being supported, so in the rare case of it being fired it would be completely safe to do so.

## Methodfor reasoning
If you have read this far, it is likely you are as pedantic as I am, and you are likely to be asking ‘Ok, but how does the server know the corresponding verb’ as different verbs will require different validation.

Well I propose two things, a new HTML attribute to cater for the following verb and also a new HTTP header for the same purpose.

```
<form method="VALIDATE" methodfor="DELETE" action="/checkout/pay" >
```

Which in turn would send:
```
VALIDATE /checkout/pay HTTP/1.1
Method-For: DELETE
…
```
Where the server would respond:
```
HTTP/1.1 412 Precondition Failed
Date: Fri, 31 Jun 2014 22:21:59 GMT
Content-Type: text/json
Vary: Method, Method-For
```

This means that:

- We don't end up with `VALIDATE-DELETE`, `VALIDATE-PUT`, `VALIDATE-POST`, `VALIDATE-HEAD`, `VALIDATE-NEW-SHINY` methods
- Until full adoption by browsers server side support can retrofit functionality.
  - Older user agents can easily not worry about what the new verbs and parameters so long as the server assumes no support if no `Method-For` header is not sent
  - Server should treat request as a normal `POST` interaction with no side effects 
  - Server can then send back the corresponding approval / validation issues mentioned above
- User agents on sites with `VALIDATE` support can automatically send the requests safely
- Forms could display differently based upon the `methodfor` and `method=”VALIDATE”`


## Further discussions into integration

### Method discovery
Frameworks and user agents would also be able to quickly resolve if the form could support the method by running an OPTIONS request which could likely be cached for a significant amount of time.

### Messaging

Deciding upon a standard for negotiation between the client and server for how validation responses would be formed is key for JavaScript to really pick up and get use out of the VALIDATE verb.

Specifications like JSON API could be adapted and expanded to provide the same kind of validations as specified here: [JSON API review](http://discourse.specifiction.org/t/json-api-v1-0-review/207/5) or serialised versions of [Robin Berjon's Web Schema](https://github.com/darobin/web-schema).

