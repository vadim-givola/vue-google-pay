# Vue Google Pay
A simple Vue wrapper for <a href="https://developers.google.com/pay/api/web/overview" target="_blank">Google Pay</a>

<i>Please note that at the moment it doesn't support shipping and dynamic price update</i>

## Installation
```
npm install vue-google-pay
```
or
```
yarn add vue-google-pay
```

## Usage
1. Import the component
```javascript
import { GooglePayButton } from 'vue-google-pay'
```
2. Add the component
```javascript
<GooglePayButton
  id="google-pay-btn"
  :options="options"
  @payed="processPayment($event)"
  @cancel="handleCancel"
/>
```
3. Add options and methods
```javascript
data () {
   return {
      environment: 'TEST',
      buttonColor: 'white',
      allowedCardNetworks: [
         'AMEX',
         'DISCOVER',
         'INTERAC',
         'JCB',
         'MASTERCARD',
         'VISA'
      ],
      allowedCardAuthMethods: ['PAN_ONLY', 'CRYPTOGRAM_3DS'],
      merchantInfo: {
         merchantName: 'Example Merchant',
         merchantId: '0123456789'
      },
      transactionInfo: {
         totalPriceStatus: 'FINAL',
         totalPrice: '1.00',
         currencyCode: 'USD',
         countryCode: 'US'
      },
      tokenizationSpecification: {
         type: 'PAYMENT_GATEWAY',
         parameters: {
            gateway: 'example',
            gatewayMerchantId: 'exampleGatewayMerchantId'
         }
      }
   }
},
methods: {
   payed (paymentData) {
      // process payment
   },
   cancelled () {
      // handle cancel event
   }
}
```
