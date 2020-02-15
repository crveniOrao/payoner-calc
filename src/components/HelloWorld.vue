<template>
  <div class="main-div">
    <v-card class="mx-auto" max-width="500px" shaped style="background: #d0e1f9">

      <v-card-title primary-title class="title">Payoneer Calc</v-card-title>

      <v-card-text>
        <div style="display: flex; justify-content: center; align-items: center; position: relative; width: 150px; margin: 0 auto;">
          <input type="number" name="amount" id="amount-input" placeholder="Amount in RSD" v-model="amount" @click="$event.target.select()" @keypress.enter="getResult()">
          <transition name="fadeRight">
            <v-icon @click.prevent="reset()" v-if="amount" style="margin-left: 1rem; color: #ee4035; position: absolute; right: 10px;">fas fa-times</v-icon>
          </transition>
        </div>

        <div class="currency">
          <img :class="currency == 'EUR' ? 'selected' : ''" @click="reset(); currency = 'EUR';" src="../assets/eu.webp" alt="eu-flag">
          <img :class="currency == 'USD' ? 'selected' : ''" @click="reset(); currency = 'USD';" src="../assets/usa.webp" alt="usa-flag">
        </div>

        <h3 class="result-text" color="secondary">{{ result }} {{ currency }}</h3>

        <v-btn style="background: #283655; color: white" @click="getResult()">Get Result</v-btn>

        <transition-group name="fadeDown">
          <p v-show="rate" class="rate" key="rate"><strong>Rate:</strong> 1 {{ currency }} = {{ rate }}</p>
          <p v-show="rate" key="formula"><strong>Formula:</strong> <span style="font-style: italic;">(Amount / (currency rate - 3.5%)) + {{currency == 'USD' ? '$3.5' : '€2.5'}}</span></p>
          <p v-show="rate" key="source">Currency rate from https://fcsapi.com/</p>
        </transition-group>
      </v-card-text>
    </v-card>

    <v-snackbar
      v-model="snackbar"
      :timeout="2000"
    >
      Please add amount first
      <v-btn
        color="info"
        text
        @click="snackbar = false"
      >
        Close
      </v-btn>
    </v-snackbar>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      amount: '',
      result: 0,
      currency: 'USD',
      rate: null,
      snackbar: false
    };
  },
  methods: {
    getResult () {
      if(!this.amount) return this.snackbar = true

      axios.get(`https://fcsapi.com/api/forex/converter?symbol=${this.currency}/RSD&amount=${this.amount}&access_key=${process.env.VUE_APP_API_URL}`)
      .then(resp => {
        let rate = parseFloat(resp.data.response.price_1x_RSD)
        let atm_fee = this.currency == 'USD' ? 3.5 : 2.5

        this.result = (this.amount / (rate - (rate * 3.5 / 100)) + atm_fee).toFixed(2)
        this.rate = rate - (rate * 3.5 / 100)
      })
      .catch(e => {
        console.log(e)
      })
    },
    reset () {
      this.amount = ''
      this.result = 0
      this.currency = 'USD'
      this.rate = null
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">

.v-card {
  width: 95vw;
  height: 95vh;
  display: flex;
  text-align: center;
  flex-direction: column;
}


.main-div {
  display: flex;
  justify-content: center;
  align-items: center;
}

.title {
  display: flex;
  justify-content: center;
  background: #1e1f26;
  color: white;
}

#amount-input {
  text-align: center;
  margin: 3rem 0rem;
  font-size: 1.5rem;
  width: 150px;
  height: 50px; 
  border-radius: 15px;
  background: #4d648d;
  color: white;
  border: none;
  outline: none;
  &::placeholder {
    color: #ccc;
    font-size: 1.3rem;
  }
}


.result-text {
  min-height: 30px;
  min-width: 150px;
  text-align: center;
  margin-bottom: 3rem;
  font-size: 2.5rem;
}

.currency {
  display: flex;
  justify-content: space-evenly;
  img {
    width: 75px;
    height: 50px;
    object-fit: cover;
    border-radius: 5px;
    margin-bottom: 3rem;
    padding: 0.3rem;
  }
  .selected {
    border: 2px solid #1e1f26;
  }
}

.rate {
  margin-top: 3rem;
}


</style>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Exo+2&display=swap');

* {
  font-family: 'Exo 2', sans-serif;
}
  /* Chrome, Safari, Edge, Opera */
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Firefox */
input[type=number] {
  -moz-appearance:textfield;
}
</style>