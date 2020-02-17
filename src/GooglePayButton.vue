<template>
  <div :id="id" />
</template>

<script>
export default {
  props: {
    id: {
      type: String,
      required: true,
      default: 'google-pay-button'
    }
  }
}
</script>
