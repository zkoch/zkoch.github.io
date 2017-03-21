# Payment Manifest Proposal V2
zkoch@google.com

## Intro

This proposal is an extension of the [Payment Method Identifier](https://github.com/zkoch/zkoch.github.io/blob/master/pmi.md) proposal and an updated version of the [original Payment Method Manifest proposal](https://github.com/zkoch/zkoch.github.io/blob/master/payment-manifest.md).

## Goals of this Proposal

The goal of this proposal is to explain the following:

* Given an identifying URL, how to access a machine-readable manifest file about that payment method in a standard, well-defined way
* Define the structure for this machine-readable manifest file
* Define how we extend the web app manifest spec to enable a variety of use cases

## Proposal

### Payment-Manifest

For every payment method, there must be a corresponding JSON manifest file describing how that method participates in the PaymentRequest ecosystem.

This manifest contains two key pieces of information: 1.) Any default applications (if any) that are associated with this payment method, referenced as an absolute URL to a web app manifest; 2.) Any other origins (if any) that are permitted to return back payment credentials for a given payment method.

## Use Cases

There are three primary use cases that this manifest file seeks to address:

1. The ability to confirm the authenticity of payment apps associated with payment methods. Put another way, if a user installs a payment application associated with a proprietary payment method on their platform of choice, we want to verify that the app is who it claims to be.
2. The ability for proprietary payment methods to control what other payment apps in the system are able to support their payment method.
3. The ability for payment mediators to offer improved UX in cases where a users wants to use a payment method but doesn't have a necessary app installed (i.e. run-time or dynamic payment app registration/installation).

### Accessing the Payment-Manifest

We will use HTTP Link Headers defined in [RFC5988](http://www.rfc-editor.org/rfc/rfc5988.txt). This allows you to go from an identifier (an absolute URL) to the machine-readable manifest.

The format of the HTTP link header for manifest is as follows:

`Link: <https://alicepay.com/pay/>; rel="payment-method-manifest"`

A new relationship type of "payment-method-manifest" will need to be coordinated with the IANA.

## Example Structure of a Payment-Manifest

```js
// payment-manifest.json
{
  "applications": ["https://bobpay.xyz/path/to/webappmanifest.json"],
  "supported_origins": ["https://alicepay.xyz"] // or "*" to indicate open
}
```

## Example Web App Manifest (simplified)

Below is an example of a web app manifest for a payment app. Web app manifests already have affordances for native apps and service workers, so the only additional bits necessary are platform-specific for the validation of payment apps. We've added two to the "play" platform: `sha256_cert_fingerprints` and `version`.

```js
{
  "name": "BobPay - World's Greatest Payment Method",
  "description": "This payment method changes lives",
  "short_name": "BobPay",
  "icons": [{
    "src": "icon/lowres.webp",
    "sizes": "64x64",
    "type": "image/webp"
  },{
    "src": "icon/lowres.png",
    "sizes": "64x64"
  }, {
    "src": "icon/hd_hi",
    "sizes": "128x128"
  }],
  "serviceworker": {
    "src": "payment-sw.js",
    "scope": "/pay",
    "use_cache": false
  }
  "related_applications": [
    {
      "platform": "play",
      "url": "https://play.google.com/store/apps/details?id=com.bobpay",
      "sha256_cert_fingerprints": ["59:5C:88:65:FF:C4:E8:20:CF:F7:3E:C8..."], //new
      "version": 1, // new
      "id": "com.example.app1"
    }, {
      "platform": "itunes",
      "url": "https://itunes.apple.com/app/example-app1/id123456789",
    }
  ]
}
```

## Anticipated FAQ

**Why such heavy reliance on web app manifests?**

Turns out, they're really useful. They already have almost all the data we need (icons, labels, service workers, related_applications), so it didn't seem to make sense to try and duplicate this information inside of the payment method manifest.

It also allows us to cleanly separate the idea of "payment method" and "payment app". Payment methods always have a payment method manifest and payment apps always have a web app manifest. So clean. Amaze.

**Isn't there a security concern with run-time installation of payment apps?**

Maybe, but I don't the risk is any greater than what's already present on the web. Origin is still going to play a big role in helping users determine what to install, and user agents will continue to invest in blocking nefarious actors.

**I don't understand what supported_origins is for. Help?**

In most cases today, there is a 1:1 correspondance between proprietary payment method and supported payment app. PayPal is a good example. If a merchant supports PayPal payments, only PayPal can successfully authorize that transaction (right now). 

But this isn't *always* the case. Let's take a made up example. We have the greatest payment method in the world, BobPay, with an identifying URL of "https://bobpay.xyz/pay". We then have a Payment App called "Alice Pay" (located at https://alicepay.xyz), which is a really popular P2P payment app. BobPay and AlicePay make a deal that says wherever BobPay is accepted, users can use their AlicePay application to pay. GeorgePay, however, a competing P2P payment app has not entered into the same agreement, so unfortuantely GeorgePay cannot be used to pay when a merchant accepts BobPay.

We need some way for BobPay to easily express this relationship structure. This is what "supported_origins" is for. By putting "https://alicepay.xyz" as a supported origin, if the user agent ever encounters a Payment App for AlicePay claiming support for BobPay, we can verify that with BobPay.

Note that this *doesn't* enable run-time installation of the "AlicePay" payment app.

**Why supported origins and not supported absolute URLs? Or why not link to the supported web app manifest directly?**

This would just make a much tighter coupling between entities than what is necessary. In the case of the above BobPay and AlicePay scenario, AlicePay has to keep BobPay informed any time their payment app location changes. This is incredibly difficult to scale. Our primary purpose is validating the relationship, nothing more. 
