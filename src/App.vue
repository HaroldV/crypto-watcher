<template>
  <div>
    <intro></intro>
    <div id="body">
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
import Intro from '@/components/Intro'
import Current from '@/components/Current'
import Previous from '@/components/Previous'
import moment from 'moment'
import axios from 'axios'
import Pusher from 'pusher-js'

export default {  
  name: 'app',  
  components: { Intro, Current, Previous },
  
  data () {
    return {
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
  
  methods: {
    fetchDataFor: (key, daysAgo,previousCurrency) => {              
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
    
    fetchDataForToday: (currentCurrency) => {
      let url = 'https://min-api.cryptocompare.com/data/pricemulti?fsyms=BTC,DGB,SC&tsyms=USD'                  
      axios.get(url).then(res => {        
        localStorage.setItem('BTC', currentCurrency.BTC = res.data.BTC.USD)
        localStorage.setItem('DGB', currentCurrency.DGB = res.data.DGB.USD)
        localStorage.setItem('SC', currentCurrency.SC = res.data.SC.USD)
      })
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
        yesterday: JSON.parse(localStorage.getItem('yesterdayPrices')),
        twoDays: JSON.parse(localStorage.getItem('twoDaysPrices')),
        threeDays: JSON.parse(localStorage.getItem('threeDaysPrices')),
        fourDays: JSON.parse(localStorage.getItem('fourDaysPrices')),
        fiveDays: JSON.parse(localStorage.getItem('fiveDaysPrices')),
      }
    } else {      
      this.fetchDataFor('yesterday', 1,this.previousCurrency)
      this.fetchDataFor('twoDays', 2,this.previousCurrency)
      this.fetchDataFor('threeDays', 3,this.previousCurrency)
      this.fetchDataFor('fourDays', 4,this.previousCurrency)
      this.fetchDataFor('fiveDays', 5,this.previousCurrency)
      this.fetchDataForToday(this.currentCurrency)
    }
  }
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
</style>