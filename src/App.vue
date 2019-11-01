<template>
  <div id="app">
    <credit-card-input mask="#### #### #### #### ##"></credit-card-input>
  </div>
</template>

<script>
  import CreditCardInput from './components/credit-input/credit-input';

  export default {
    components: {CreditCardInput},
    data() {
      return {}
    }
  }
</script>

<style lang="scss">

</style>
