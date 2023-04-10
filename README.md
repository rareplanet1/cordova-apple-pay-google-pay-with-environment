# Cordova Apple Pay and Google Pay integration

> This is a fork of https://www.npmjs.com/package/cordova-plugin-apple-pay-google-pay with the addition of an Android environment preference to allow for testing and producing screenshots of your buyflow for [submission to Google](https://developers.google.com/pay/api/android/guides/brand-guidelines#put-it-all-together).

This plugin is built as unified method for obtaining payment tokens to forward it to your [supported payment processor](https://developers.google.com/pay/api#participating-processors) (eg Fat Zebra,
Stripe, etc).

Plugin supports iOS 11-14. Tested properly with cordova 10, iOS 14.6 and Android 11.

## Installation

```
cordova plugin add cordova-plugin-apple-pay-google-pay-with-environment
```

or for Cordova build service, e.g. [VoltBuilder](https://volt.build/), add the following to config.xml:

```
<plugin name="cordova-plugin-apple-pay-google-pay-with-environment" source="npm" spec="1.1.5" />
```

For Android, register and fill all required forms at https://pay.google.com/business/console. Add the following to
config.xml:

```
<config-file parent="/manifest/application" target="AndroidManifest.xml">
    <meta-data
            android:name="com.google.android.gms.wallet.api.enabled"
            android:value="true" />
</config-file>
```

For testing and producing screenshots of your buyflow for submission to Google:
```
<preference name="GooglePayEnvironment" value="test" />
```

For iOS, you have to have valid developer account with merchant set up and ApplePay capability and a merchant id
configured in your Xcode project. Merchant id can be obtained
from https://developer.apple.com/account/resources/identifiers/list/merchant. Do configuration manually or using
config.xml:

```
<platform name="ios">

  <config-file target="*-Debug.plist" parent="com.apple.developer.in-app-payments">
    <array>
      <string>developer merchant ID here</string>
    </array>
  </config-file>

  <config-file target="*-Release.plist" parent="com.apple.developer.in-app-payments">
    <array>
      <string>production merchant ID here</string>
    </array>
  </config-file>
</platform>
```

## Usage

`canMakePayments()` checks whether device is capable to make payments via Apple Pay or Google Pay.

```
// use as plain Promise
async function checkForApplePayOrGooglePay(){
    let isAvailable = await cordova.plugins.ApplePayGooglePay.canMakePayments()
}

// OR
let available;

cordova.plugins.ApplePayGooglePay.canMakePayments((r) => {
  available = r
})
```

`makePaymentRequest()` initiates pay session.

```
let request = {
    merchantId: 'merchant.com.example', // obtain it from https://developer.apple.com/account/resources/identifiers/list/merchant
    purpose: `Payment for your order #1`,
    amount: 100,
    countryCode: "US",
    currencyCode: "USD"
}

cordova.plugins.ApplePayGooglePay.makePaymentRequest(request, r => {
        // in success callback, raw response as encoded JSON is returned. Pass it to your payment processor as is.
      let responseString = r

      },
      r => {
        // in error callback, error message is returned.
        // it will be "Payment cancelled" if used pressed Cancel button.
      }
   )
```

All parameters in request object are required.

## For iOS

Set `purpose` parameter of payment request to the Merchant Name to ensure that the business receiving the payment is [specified on the payment sheet](https://developer.apple.com/design/human-interface-guidelines/apple-pay/overview/checkout-and-payment/), otherwise it is likely your app submission will be rejected.

## For Android

You will have to provide few extra parameters:

```
request.gateway = 'stripe'; // or any another processor you are using: https://developers.google.com/pay/api#participating-processors
request.merchantId = 'XXXXXXX'; // merchant id provided by your processor
request.gpMerchantName = 'Your Company Name'; // will be displayed in transaction info
request.gpMerchantId = 'XXXXXXXXXXXX'; // obtain it at https://pay.google.com/business/console
```

For Stripe, Braintree and Vantiv, you will have to provide more extra parameters, all the parameter information can be found at [google pay for payments](https://developers.google.com/pay/api/android/reference/object#PaymentMethodTokenizationSpecification):

**Stripe**
```
request.version = 'your stripe api version'
request.publishableKey = 'your stripe payment processing publishable key'
```

**Braintree**
```
request.apiVersion = 'your braintree api version'
request.sdkVersion = 'your braintree sdk version'
request.clientKey = 'your btraintree client key'
```

**Vantiv**
```
request.merchantPayPageId = 'your vantiv page id'
request.merchantOrderId = 'order id'
request.merchantTransactionId = 'transactionID'
```

Also, on Android checking payment availability calling `canMakePayments()` always returns false even if user has a valid card attached to GooglePay.
