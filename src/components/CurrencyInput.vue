<template>
  <div>
    <h3>Enter Conversion Details</h3>
    <div class="block-wrapper">
        <label for="amount">Amount in GBP:</label>
        <input v-model="amount" type="number" id="amount" />
    </div>

    <div class="block-wrapper">
        <label for="minRate">Minimum Conversion Rate (GBP to EUR):</label>
        <input v-model="minRate" type="number" id="minRate" step="0.01" />
    </div>

    <div class="block-wrapper">
        <label for="maxRate">Maximum Conversion Rate (GBP to EUR):</label>
        <input v-model="maxRate" type="number" id="maxRate" step="0.01" />
    </div>

    <button @click="handleConvert">Convert</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      amount: 0,
      minRate: 1.15,
      maxRate: 1.20
    };
  },
  methods: {
    handleConvert() {
      this.$emit('convert', this.amount, this.minRate, this.maxRate);
    }
  }
};
</script>

<style scoped>
    .block-wrapper{
        margin-bottom: 10px;
    }
    button{
        border: 1px solid;
        padding: 10px;
        background: #3FB31A;
        color: white;
        font-weight: bold;
    }
</style>
