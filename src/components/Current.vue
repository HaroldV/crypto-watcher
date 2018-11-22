<template>
  <div>
    <h2>Precios actuales de las Cryptomonedas</h2>
    <div id="btc" class="currency">
      <label > 1 BTC</label>
      <p>${{currentCurrency.BTC}}</p>
    </div>
    <div id="dgb" class="currency">
      <label > 1 DGB</label>
      <p>${{currentCurrency.DGB}}</p>
    </div>
    <div id="sc" class="currency">
      <label > 1 SC</label>
      <p>${{currentCurrency.SC}}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  props: {
    currentCurrency: {  type: Object  }    
  },
  data () {
    return {}
  }
}
</script>

<style scoped>
div p { font-size: 2rem; }
h2 { font-size: 1.5em; }
.currency {
  border: 1px solid #F5CE00;
  border-radius: 15px;
  padding: 2em 1em;
  display: inline-block; 
  margin: 5px; 
}
</style>

