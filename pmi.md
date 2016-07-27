# Payment Method Identifiers

## Intro

This is a proposal for how Payment Method Identifers could work with the PaymentRequest API. This document is the result of numerous conversations and attempts to address many of the use cases expressed by various members of the Working Group and others. 

## Proposal

### Payment Methods

Broadly, there are two types of payment methods that we care about: 1.) General, "open" systems that can be integrated and/or supported by a variety of players without special permission (e.g. credit cards, SEPA, etc); 2.) What can be called "proprietary" systems (e.g. PayPal.com). It is oftentimes the case, though not always, that there is a 1:1 mapping between a proprietary payment method and its corresponding application (e.g. only the PayPal application can process a PayPal payment).

More details on each can be found below.

### Open Systems

**Requirements:**

* Are standardized by the WG and have a corresponding specification that details how they are identified and their expected response
* Are identified by a URN (i.e. it is purely an identifier, it does not resolve to anything)
* Can be supported by any third party payment app
* Identifier should have the format of: `urn:payment-method:{name}` [1]

A more detailed example of how credit cards will work in this world can be found below, but as a short example taking our existing specifications for [Basic Card Payment](https://w3c.github.io/webpayments-methods-card/) and [SEPA Credit Transfer](http://w3c.github.io/webpayments-methods-credit-transfer-direct-debit/), we would have:

```js
var supportedMethods = [{
  supportedMethods: ['urn:payment-method:basic-card-payment'],
  data: {
    supportedNetworks: ['visa']
  }
}, {
  supportedMethods: ['urn:payment-method:push-sct'],
  data: {
    somethingAboutSepa: '12345'
  }
}];
```

Any changes made to these payment methods over time must be approved by the WG.

[1] We can formalize what we think this naming structure should look like. This is just a straw man to get started.


### Proprietary Systems

**Requirements:**

* Are not standardized by the WG and thus do not have specifications located inside WG-controlled repos
* Are identified by an absolute URL (e.g. https://bobpay.xyz/pay)
* Documentation is the responsbility of the payment method owner
* By default, third party payment apps other than those associated with the identifying origin cannot claim to offer support for these payment methods

This proposal does not necessarily assume that anything should exist at the identifying URL, but it is probably *really useful* to do so. The WG should consider standardizing what this optional thing could look like (e.g. a manifest file containing information about the payment method).

The benefit of the above approach is that it provides a mechanism for the identifying origin to assert what other origins could potentially also support that payment method.

For example, let's say that BobPay is a proprietary payment method, but BobPay has struck a deal with AlicePay to also allow AlicePay to return back valid BobPay payment responses. We could define a mechanism to support this within the manifest file located at bobpay.xyz/pay:

```js
{
  name: 'BobPay - Payments of the Future',
  supportedOrigins: ['https://alicepay.xyz']
}
``` 

In addition to the above, a manifest file (or similar) allows the Payment Mediators to pull data about unknown payment methods for a variety of improved user experiences.

## Credit Cards

Because credit cards are such an important part to bootstrapping the ecoystem for Payment Request and have a number of complex requirements, I want to better lay out how they will work under this proposal.

A few changes:

1. Basic credit card support is expressed as a single `basic-card-payment` identifier instead of individual network names.
2. Individual network support is passed into the `data` object as `supportedNetworks`. By default, all networks are supported.
3. Additional support for filtering by `debit`, `credit`, and `prepaid` card types under a `supportedTypes` key. By default, all types are supported.

Example:

```js
var methodData = [{
  supportedMethods: ['urn:payment-method:basic-card-payment'],
  data: {
    supportedNetworks: ['visa', 'mastercard'],
    supportedTypes: ['debit']
  }
}];
```

> Note: It's important to note that a number of third party payment apps might not know whether or not a given card in their system is credit or debit. If we require the payment app to report this and aggressively filter in cases where they don't know, we potentially prevent users from using cards they might otherwise have been able to pay with. Should we enforce this aggressive filtering, or fall back on the user to decide?

For more complex requirements (e.g. limiting to corporate cards), we would expect a more advanced card payment option to be available under a different identifier, `urn:payment-method:advanced-card-payment`, that can evolve independently of basic card payments.

## Changes to PaymentRequest

To enable some of the more complex matching requirements for payment methods, a couple of changes need to be made to PaymentRequest:

* PaymentRequest should be able to support multiple payment methods of the same identifier. This allows situations like the following:

```js
var methodData = [{
  supportedMethods: ['urn:payment-method:basic-card-payment'],
  data: {
    supportedNetworks: ['visa'],
    supportedTypes: ['debit']
  }
}, {
  supportedMethods: ['urn:payment-method:basic-card-payment'],
  data: {
    supportedNetworks: ['mastercard'],
    supportedTypes: ['credit']
  }
}];
```

* We need to extend [PaymentDetailsModifier](https://w3c.github.io/browser-payment-api/#paymentdetailsmodifier-dictionary) to also support the filtering criteria outlined in a given payment method specification. For example, some site might want to provide a price difference between paying with a Visa credit card as opposed to a Visa debit card. It is expected that the Mediator will match on the **first** matching set of conditions. So more specific conditions should be accounted for first to avoid confusion.
