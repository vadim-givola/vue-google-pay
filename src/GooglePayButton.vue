<template>
  <div :id="id" />
</template>

<script>
import props from './props'

export default {
  props: {
    id: {
      type: String,
      required: false,
      default: 'google-pay-button'
    },
    options: {
      type: Object,
      required: true,
      default: function () {
        return props
      }
    }
  },
  data () {
    return {
      ...props,
      cardPaymentMethod: null,
      paymentsClient: null
    }
  },
  async mounted () {
    const varsSet = await this.assignVars()
    if (varsSet) this.injectGooglePayScript()
  },
  methods: {
    async assignVars () {
      Object.assign(this, this.options)
      return await this.$nextTick()
    },
    injectGooglePayScript () {
      if (!this.paymentsClient) {
        // load google pay script
        const googlePayScript = document.createElement('script')
        googlePayScript.setAttribute('src', 'https://pay.google.com/gp/p/js/pay.js')
        googlePayScript.setAttribute('async', true)
        googlePayScript.setAttribute('defer', true)
        googlePayScript.onload = () => this.onGooglePayLoaded()
        document.head.appendChild(googlePayScript)
      }
    },
    async initPaymentsVars () {
      this.cardPaymentMethod = Object.assign(
        {},
        this.baseCardPaymentMethod,
        {
          tokenizationSpecification: this.tokenizationSpecification
        }
      )
      return await this.$nextTick()
    },
    getGoogleIsReadyToPayRequest () {
      return Object.assign({}, this.baseRequest, {
        allowedPaymentMethods: [this.baseCardPaymentMethod]
      })
    },
    getGooglePaymentsClient () {
      if (this.paymentsClient === null) {
        // eslint-disable-next-line
        this.paymentsClient = new google.payments.api.PaymentsClient({
          environment: this.environment
        })
      }
      return this.paymentsClient
    },
    addGooglePayButton () {
      const button = this.paymentsClient.createButton({
        onClick: () => this.googlePayButtonClick(),
        buttonColor: this.buttonColor
      })
      document.getElementById(this.id).appendChild(button)
    },
    async onGooglePayLoaded () {
      const varsSet = await this.initPaymentsVars()
      if (varsSet) {
        const paymentsClient = this.getGooglePaymentsClient()
        paymentsClient
          .isReadyToPay(this.getGoogleIsReadyToPayRequest())
          .then(response => {
            if (response.result) {
              this.addGooglePayButton()
              // @todo prefetch payment data to improve performance after confirming site functionality
              // prefetchGooglePaymentData();
            }
          })
          .catch(err => {
            // show error in developer console for debugging
            console.error(err)
          })
      }
    },
    getGooglePaymentDataRequest () {
      const paymentDataRequest = Object.assign({}, this.baseRequest)
      paymentDataRequest.allowedPaymentMethods = [
        this.cardPaymentMethod
      ]
      paymentDataRequest.transactionInfo = this.transactionInfo
      paymentDataRequest.merchantInfo = {
        // @todo a merchant ID is available for a production environment after approval by Google
        // See {@link https://developers.google.com/pay/api/web/guides/test-and-deploy/integration-checklist|Integration checklist}
        merchantId: this.merchantInfo.merchantId,
        merchantName: this.merchantInfo.merchantName
      }
      return paymentDataRequest
    },
    googlePayButtonClick () {
      const paymentDataRequest = this.getGooglePaymentDataRequest()
      paymentDataRequest.transactionInfo = this.transactionInfo

      const paymentsClient = this.getGooglePaymentsClient()
      paymentsClient
        .loadPaymentData(paymentDataRequest)
        .then(paymentData => {
          // handle the response
          this.$emit('payed', paymentData)
        })
        .catch(err => {
          // show error in developer console for debugging
          console.error(err)
          if (err.statusCode === 'CANCELED') this.$emit('cancel')
        })
    }
  }
}
</script>
