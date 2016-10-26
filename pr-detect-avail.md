# Detecting Payment Method Availability

### Background

We’ve heard very strong feedback from merchants that they would like to know before calling `show()` on PaymentRequest if a user has an available way to pay.

Our initial response to this feedback was to call `.show()` and, if it throws, fall back to an existing checkout flow. This, however, has created challenges. Two are worth highlighting:

* Some merchants would like to use PaymentRequest as an abbreviated checkout of sorts, thus forgoing things like coupon codes, etc. Put in a more direct way: they would like to directly replace the Apple Pay button, whose appearance is determined by a `canMakePaymentsWithActiveCard` function. But this is only worth doing if a form of payment is already available inside of PaymentRequest (otherwise, overall benefit is reduced).
* Some merchants are interested in using PaymentRequest when a payment method is available, but otherwise would prefer their own flow.

### Proposal

We add a new method onto PaymentRequest, `canMakeActivePayment()`, that returns back a Boolean indicating whether or not the user has the ability to make a payment at the time `show()` is called.

It probably makes the most sense for this to be a `Promise`.

A straw man:

```js
pr.canMakeActivePayment()
  .then(() => { return pr.show(); })
  .catch(error => { console.log(error); });
```

We would still prefer that merchants not be able to simply iterate through the entire set of payment methods and determine availability for privacy reasons. As such, `canMakeActivePayment` will be rate limited to 1 call per origin per some window of time (where the duration of this is still TBD). 

We would, however, be accepting that payment requests that are instantiated with only a single form of payment available would leak whether or now that single form of payment was available. This does not strike me as overly problematic, even if not ideal.

### Scenarios

Here I have tried to outline the different scenarios that we've heard requested from merchants and indicate whether or not these use cases would be met by the proposal.

**1. Only leverage PaymentRequest when a user has a form of payment available**

Support: **Yes**

**2. Only call PaymentRequest if a *certain* payment method is available**

Support: **Partial**

This could be done in cases where you were only checking against a single payment method.

**3. Brand payment buttons directly (e.g. Pay With BobPay)**

Support: **Partial**

**4. Brand multiple buttons that each leverage PaymentRequest with a single payment method**

Support: **No**

Well, they could do it, but there would be no guarantee that it would work.
