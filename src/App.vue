<template>
  <div>    
    <header class="hero">
      <div class="bar logo">
        <h3>CryptoWatcher</h3>
        <span class="monitor"><span class="monitorText" @click="cryptoToday(currentCurrency)">Actualizar</span></span>
      </div>
      <h1>PWA en Tiempo Real que muestra actualizaciones en Criptomonedas</h1>
      <h2>Bitcoin, Digibyte, Siacoin</h2>
    </header>      
    <div id="body">      
      <input type="text" placeholder="Search and press enter" v-model="searchCurrency" @keypress.enter="searchByCrypto(upper($event))">      
      <div id="current">
        <current :currentCurrency="currentCurrency"> </current>
      </div>
      <div id="previous">
        <previous :previousCurrency="previousCurrency"> </previous>
      </div>
    </div>
  </div>
</template>

<script>
import Current from '@/components/Current'
import Previous from '@/components/Previous'
import moment from 'moment'
import axios from 'axios'
import Pusher from 'pusher-js'

export default {  
  name: 'app',  
  components: { Current, Previous },
  
  data () {
    return {
      searchCurrency: '',
      currentCurrency: {
          BTC: '',
          DGB: '',
          SC: '',
      },      
      previousCurrency: {
          yesterday: {},
          twoDays:  {},
          threeDays: {},
          fourDays: {},
          fiveDays: {},
      }
    }
  },
  
  created() {            
    let pusher = new Pusher('f36887d03f799d430834', {
      cluster: 'us2',
      forceTLS: true
    })

    let channel = pusher.subscribe('price-updates');
    channel.bind('coin-updates', data => {
      this.currentCurrency = {
        BTC: data.coin.BTC.USD, 
        DGB: data.coin.DGB.USD, 
        SC: data.coin.SC.USD
      }
    });

    if (! navigator.onLine) {
      this.currentCurrency = {
        BTC: localStorage.getItem('BTC'),
        DGB: localStorage.getItem('DGB'),
        SC: localStorage.getItem('SC'),
      }            
      this.previousCurrency = {
        yesterday:  JSON.parse(localStorage.getItem('yesterdayPrices')),
        twoDays:    JSON.parse(localStorage.getItem('twoDaysPrices')),
        threeDays:  JSON.parse(localStorage.getItem('threeDaysPrices')),
        fourDays:   JSON.parse(localStorage.getItem('fourDaysPrices')),
        fiveDays:   JSON.parse(localStorage.getItem('fiveDaysPrices')),
      }
    } else {      
      this.cryptoFor('yesterday', 1,this.previousCurrency)
      this.cryptoFor('twoDays', 2,this.previousCurrency)
      this.cryptoFor('threeDays', 3,this.previousCurrency)
      this.cryptoFor('fourDays', 4,this.previousCurrency)
      this.cryptoFor('fiveDays', 5,this.previousCurrency)
      this.cryptoToday(this.currentCurrency)
    }
  },

  methods: {
    cryptoFor: (key, daysAgo,previousCurrency) => {              
      var date = moment().subtract(daysAgo,'days').unix()              
      let fetch = (curr, date) => axios.get(`https://min-api.cryptocompare.com/data/pricehistorical?fsym=${curr}&tsyms=USD&ts=${date}`)
    
      axios
        .all([fetch('BTC', date), fetch('DGB', date), fetch('SC', date)])
        .then(axios.spread((BTC, DGB, SC) => {
          previousCurrency[key] = {
            BTC: BTC.data.BTC.USD,
            SC: SC.data.SC.USD,
            DGB: DGB.data.DGB.USD,
            DATE: moment.unix(date).format("MMMM Do YYYY"),
          }
          localStorage.setItem(`${key} Prices`, JSON.stringify(previousCurrency[key]))
        }))
    },
    
    cryptoToday: (currentCurrency) => {    
      let url = 'https://min-api.cryptocompare.com/data/pricemulti?fsyms=BTC,DGB,SC&tsyms=USD'                  
      axios.get(url).then(res => {        
        localStorage.setItem('BTC', currentCurrency.BTC = res.data.BTC.USD)
        localStorage.setItem('DGB', currentCurrency.DGB = res.data.DGB.USD)
        localStorage.setItem('SC', currentCurrency.SC = res.data.SC.USD)
      })      
    },

    searchByCrypto: (searchCurrency) => {           
      let url = `https://min-api.cryptocompare.com/data/pricemulti?fsyms=${searchCurrency}&tsyms=USD`      
      console.log(url)
      axios.get(url).then(res => {                                              
        // document.createElement('div#'+ searchCurrency +' > label').innerHTML = '1 ' + searchCurrency                
        // document.createElement('#'+ searchCurrency +' > p').innerHTML = '$'+res.data[searchCurrency].USD
        alert(res.data[searchCurrency].USD)
      })
      .catch(err => {        
        alert('Crypto Not Found')
      })
    },

    upper: (e) => {      
      return e.target.value.toUpperCase()
    }
  },

}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Lato');
* {
  margin : 0px;
  padding : 0px;
  font-family: 'Lato', sans-serif;
}
body { height: 100%; width: 100%; }
.row { display: flex; flex-wrap: wrap; }
h1 { font-size: 38px; }
a { color: #FFFFFF; text-decoration: none; }
a:hover { color: #FFFFFF; }
a:visited { color: #000000; }
.button {
  margin: auto;
  width: 200px;
  height: 60px;
  border: 2px solid #E36F55;
  box-sizing: border-box;
  border-radius: 30px;
}
#body {
  max-width: 90%;
  margin: 0 auto;
  padding: 1.5em;
  text-align: center;
  color:rgb(0, 193, 131);
}
#current { padding: 2em 0em; }
#previous { padding: 2em 0em; }
header {
  background: linear-gradient(to bottom right, rgb(0, 193, 131),rgb(50, 72, 95));
  padding: 1em;
  margin-bottom: 1em;
  text-align: center;
  height: 300px;
  color: #fff;
}
header h3 {
    color: white;
    font-weight: bold;
    text-transform: uppercase;
    float: left;
}
bar { padding: 20px; height: 48px; }
.monitor{
  text-transform: uppercase;
  float:right;
  background-color: rgba(255, 255, 255, 0.2);
  line-height: 23px;
  border-radius: 25px;
  width: 175px;
  height: 48px;
  margin: auto;
}
.monitor:hover, monitorText:hover { cursor:pointer; }
.monitorText{
  width: 104px;
  height: 23px;
  font-weight: bold;
  line-height: 50px;
  font-size: 14px;
}
header h1 { padding-top: 80px;  margin: auto; }
header h2{ padding-top:20px; }
input {
  border-radius: 15px;
  border: 1px solid #F5CE00;
  width: 60%;
  text-align: center;
  color: rgb(0, 193, 131);
  font-size: 18px;
  text-transform: uppercase;
}
::placeholder { /* Chrome, Firefox, Opera, Safari 10.1+ */
    color: rgb(0, 193, 131);
    opacity: 1; /* Firefox */
}
</style>