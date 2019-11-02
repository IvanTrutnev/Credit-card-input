<template>
  <div>
    <div class="params-group">
      <div class="param-name">Bank:</div>
      <div>Name: {{data.bank && data.bank.name}}</div>
      <div>Phone: {{data.bank && data.bank.phone}}</div>
      <div>Url: {{data.bank && data.bank.url}}</div>
    </div>
    <div class="params-group">
      <div class="param-name">Brand:</div>
      {{data.brand}}
    </div>
    <div class="params-group">
      <div class="param-name">Country:</div>
      <div>Currency: {{data.country && data.country.currency}}</div>
      <div>Name: {{data.country && data.country.name}}</div>
      <div>Tatitude: {{data.country && data.country.latitude}}</div>
      <div>Longtitude: {{data.country && data.country.longitude}}</div>
    </div>
    <div class="params-group">
      <div class="param-name">Sheme:</div>
      {{data.scheme}}
    </div>
    <div class="params-group">
      <div class="param-name">Type:</div>
      {{data.type}}
    </div>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Object,
      default: () => ({
        bank: {},
        country: {}
      })
    }
  },
  mounted() {
    console.log(this.data);
  }
};
</script>

<style scoped lang="scss">
.params-group {
  margin-top: 10px;
  .param-name {
    color: #a09a9a;
    text-decoration: underline;
  }
}
</style>
