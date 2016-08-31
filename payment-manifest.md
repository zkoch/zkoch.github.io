# Payment Manifest Proposal
zkoch@

## Intro

This proposal is an extension of the [Payment Method Identifier](https://github.com/w3c/webpayments/blob/gh-pages/proposals/zach-pmi.md) proposal. In that proposal I state:

> This proposal does not necessarily assume that anything should exist at the identifying URL, but it is probably really useful to do so.

It emerged in subsequent discussions that leaving this open-ended was insufficient (potentially blocked work on payment apps, called into question the value of using URLs as identifiers, etc).

The purpose of this document is to clearly define what should live at the end of that identifier and why that is necessary.

## Goals of this Proposal

The goal of this proposal is to gain consensus on the following:

* Given an identifying URL, there must be a way to access a machine-readable manifest file about that payment method in a standard, well-defined way
* Every proprietary payment method in the PaymentRequest ecosystem must have a corresponding machine-readable manifest file

The *exact* contents of this manifest file can be worked out over time in addition to the *exact* mechanism for what that "standard, well-defined way" is. This proposal contains various strawman proposals to jumpstart this discussion.

For now, our goal is to agree on the above bullets in order to unblock current work and drive forward progress.

> Note that this proposal is limited to Payment Methods, but I believe most all of it would hold true and be valuable for Payment Apps as well. That said, such discussion is out of scope for this proposal. 

## Proposal

### Payment-Manifest

For every payment method, there must be a corresponding `payment-manifest.json` file describing how that method participates in the PaymentRequest ecosystem.

This manifest contains information about how that payment method can be used. This includes, but is certainly not limited to: the default payment app associated with it (if any), other supported applications (expressed as URLs), platform-specific details required for verifying authenticity of an app, etc.

### Accessing the Payment-Manifest (strawman)

`payment-manifest.json` must live at the directory of the identifying URL. It must be fetched with a MIME type of `application/json`.

As a concrete example, let's assume I define a payment method with the following identifier: `https://bobpay.xyz/pay`. The corresponding payment manifest file must live at `https://bobpay.xyz/pay/payment-manifest.json`.

There are a few benefits to this approach:

1. It allows for the method identifier itself to reatain simple, semantic meaning.
1. The identifier by itself can still resolve to a standard, human-readable HTML page. So a curious developer can simply copy and paste it into a URL bar and discover information about that method (assuming, of course, the method provider supports this).
1. It allows for a machine-readable version of the payment method to be reached in a single round trip request (i.e. no need to parse an HTML page looking for a manifest link).

> Note: This proposal was inspired by Android's [Digital Asset Links](https://developers.google.com/digital-asset-links/v1/getting-started)

## Use Cases

There are three primary use cases that this manifest file seeks to address right now, though I suspect more will emerge as the specs progress. These key three are:

1. The ability to confirm the authenticity of payment apps associated with payment methods. Put another way, if a user installs a payment application associated with a proprietary payment method on their platform of choice, we want to verify that the app is who is claims to be.
2. The ability for proprietary payment methods to control what other payment apps in the system (in addition to themselves) are able to support their payment method.
3. The ability for payment mediators to offer improved UX in cases where users want to use a payment method but don't have a necessary app installed.

## Example Structure of a Payment-Manifest

```js
// payment-manifest.json
{
  "payment_app": {
    "android": {
      "package": "xyz.bobay.app",
      "sha256_cert_fingerprints": ["14:6D:E9:83:C5:73:06:50"],
      "platforms": {
        "play": "https://play.google.com/store/apps/details?id=com.bobspayments.app1"
      }
    },
    "web": {
      "name": "Bob's Payments",
      "icons": [{
        "src": "icon/lowres.webp",
        "sizes": "48x48",
        "type": "image/webp"
      }, {
        "src": "icon/lowres",
        "sizes": "48x48"
      }, {
        "src": "icon/hd_hi.ico",
        "sizes": "72x72 96x96 128x128 256x256"
      }, {
        "src": "icon/hd_hi.svg",
        "sizes": "257x257"
      }]
    },
    "ios": {...}
  },
  "externally_supported_apps": ["https://alicepay.com/pay"]
}
```

## Alterntive Proposals for Accessing Payment-Manifest

There are a few other ways that we could go about doing this. This is open for discussion. A couple other ideas to get started:

* `<link rel="payment-manifest" href="/payment-manifest.json">` at the identifying URL.
* Something in the request header of the identifying URL indicating you're looking for the machine-readable version
