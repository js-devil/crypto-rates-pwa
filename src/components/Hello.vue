<template>
  <div class="hello">

    <header>
      <span>Crypto Rates</span>
      <span class="curr" v-if="currency" @click="openModal">
        {{currency}} 
        <i class="fas fa-caret-down down"></i>
      </span>
    </header>

    <div class="container" ref="layout">      
      <table class="table table-striped" style="position: relative; top: 15px;">

        <!--Table head-->
        <thead>
          <tr class="table-head">
            <th style="text-align: left">#</th>
            <th>Coin</th>
            <th>Price ({{currency}})</th>
            <th style="text-align: center">Change</th>
          </tr>
        </thead>
      </table>
      <!--Table-->

      <div style="overflow-y: scroll; height: 510px;">
        <table class="table table-striped"> 
        <tbody>
          <tr class="table-info" v-for="(coin, i) in coinRates" :key="i">
            <th scope="row">{{i+1}}</th>
            <td>
              <img :src="coinImage(coinTypes[i].toLowerCase())" class="coin-image"/>
              {{coinTypes[i]}}</td>
            <td>{{parseFloat(format(coin)).toLocaleString('en')}}</td>
            <td v-if="prevRates.length>0">{{changeInRates(coin, prevRates[i])}}
              <span v-if="coin > prevRates[i]">
                <i class="fas fa-caret-up" style="color: green"></i>
              </span>
              <span v-else-if="coin < prevRates[i]">
                <i class="fas fa-caret-down" style="color: red"></i>
              </span>
              <span v-else>
                <i class="fa fa-minus"></i>
              </span>
            </td>
          </tr>
        </tbody>
      </table>
      </div>
      
    </div>

    
    <div class="coin-name" v-show="modal" style="top: calc(25% - 160px);">
      <div style="height:100vh; background: #fff;">
        <div class="selected-coin">
          <p>Select Currency</p>
        </div>
        
        <div style="overflow-y: scroll; height: 550px;">
          <div v-for="(data, index) in currencies" :key="index">
            <p class="coin-selected" @click="currency=data.id; closeModal();">{{data.name}} ({{data.id}})</p>
          </div>
        </div>
      </div>
          
      <footer class="footer-details" @click="closeModal">
      <p>Cancel</p>
      </footer>
    </div>

  </div>
</template>

<script>
var axios = require('axios')
const Endpoint = 'https://api.coinbase.com/v2'
export default {
  name: 'hello',
  data () {
    return {
      currency: '',
      coinTypes: ["BTC","LTC","BCH","ETH", "XRP", "ETC", "BAT", "BCH", "EOS", "ZEC"],
      coinRates: [],
      prevRates: [],
      currencies: [],
      currency: null,
      modal: false,
    }
  },
  methods:{
    openModal(){
      let layout = this.$refs.layout;
      layout.style.opacity = '0.2'
      this.modal = true;
    },
    closeModal(){
      let layout = this.$refs.layout;
      layout.style.opacity = '1';
      this.modal = false;
    },
    getUserCountry: async function(){
      let response = await axios.get("https://ipapi.co/json")
      this.currency = response.data.currency
      this.getExchange(this.currency)

      response = await axios.get(`${Endpoint}/currencies`);
      this.currencies = response.data.data
    },
    getExchange: async function(currency){
      let rates = []
      
      if(this.coinRates.length>0){
        this.prevRates = this.coinRates
      }

      for (var i = 0; i < this.coinTypes.length; i++){
        const response = await axios.get(`${Endpoint}/exchange-rates?ran=1&currency=${this.coinTypes[i]}`);
        
        if(response.data.data.rates.hasOwnProperty(currency)){
          rates.push((response.data.data.rates[currency]))
        }
      }
      this.coinRates = rates
      let layout = this.$refs.layout;
      layout.style.opacity = '1'
      $('.loader').css('display', 'none')
    },
    format(coin){
      return parseFloat(coin).toFixed(2)
    },
    changeInRates(new_value, old_value){
      if(new_value && old_value){
        let diff =((new_value - old_value)/new_value) * 100
        return (diff).toFixed(2)+'%'
      }else{
        return ''
      }
    },
    coinNames(coin){
      if(coin=='BTC'){
        return 'Bitcoin'
      }
      else if(coin=='ETH'){
        return ''
      }
    },
    coinImage(url){
      return require('../assets/'+url+'.png')
    }
  },
  mounted(){
    let layout = this.$refs.layout;
    layout.style.opacity = '0'
    this.getUserCountry()

    setInterval(() => {
      if(this.currency){
        this.getExchange(this.currency)
      } 
    }, 5000);
  },
  watch:{
    currency(){
      $('.loader').css('display', 'block')
      let layout = this.$refs.layout;
      layout.style.opacity = '0'
      this.getExchange(this.currency)
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
header{
  display: flex;
  margin-top: -11%;
  justify-content: space-between;
  position: fixed;
  width: 100%
}

table{
  color: #35495E;
}
.table-head th{
  font-weight: 600;
  text-align: right;
}
.table-info>td, .table-info>th {
  background-color: none;
  padding-top: 15px;
  text-align: left;
}

.table-info:nth-child(even)>td, .table-info:nth-child(even)>th {
  background-color: #fff;
}

.selected-coin{
  background: #35495E;
  color: white;
  height: 40px;
  padding-top: 10px;
}
  .selected-coin p{
    text-align: left;
    padding-left: 10px;
    font-weight: 500;
    font-size: 15px;
  }
  .coin-selected{
  color: #35495E;
    text-align: left;
    padding-left: 9px;
    padding-top: 10px;
    font-size: 15px;
  }
  .coin-name{
    background: white;
    height: 75vh;
    position: fixed;
    top: calc(25% - 5px);
    width: 100%;
    z-index: 222222;
    transition: 2s ease-in-out;
  }
   .footer-details{
    position: fixed;
    bottom: 0;
    width: 100%;
    background: #f0f0f0;
    height: 50px;
    display: flex;
    align-items: center;
    padding-left: 10px;
    color: red;
  }
  .footer-details p{
    margin-bottom: 0;
  }
  .down{
    color: #bee5eb;
  }
  .curr{
    font-size: 90%;
    top: 3px;
  }
  .coin-image{
    height: 25px;
  }
</style>
