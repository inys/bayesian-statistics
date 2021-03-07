<template>
  <div class="container">
    <div class="field">
      <label class="label">Number of samples</label>
      <div class="control">
        <input v-model="number"
          class="input is-small" type="text"/>
      </div>
    </div>
    <div class="field">
      <label class="label">Real mu</label>
      {{ realMu}}
      <div class="control">
          <input
          v-model="realMu"
          type="range"
          min="0.01"
          max="0.99"
          step="0.01"
          class="slider"/>
      </div>
    </div>
    <div class="field">
      <label class="label">Range</label>
      {{ range }}
      <div class="control">
        <input 
          v-model="range"
          type="range"
          min="1"
          :max="number"
          class="slider"
          id="myRange"/>
      </div>
    </div>
  </div>
  <hr>
  <p>RandomValues: {{ randomValues }}</p>
  <hr>
  <div class="container">
    <div id="plot"></div>
  </div>
</template>

<script>
import { create, all } from 'mathjs'

// create a mathjs instance with configuration
const config = {
  epsilon: 1e-12,
  matrix: 'Matrix',
  number: 'number',
  precision: 64,
  predictable: false,
  randomSeed: 2021
}
const math = create(all, config);

import Plotly from 'plotly.js';

export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      number: 50,
      range: 10,
      randomValues: [],
      realMu: 0.3,
    }
  },
  computed: {
    m: function() {
      return math.sum(this.randomValues.slice(0,this.range));
    }
  },
  watch: {
    number: function() {
      this.generateRandomValues();
      this.draw();
    },
    range: function() {
      this.draw();
    },
    realMu: function() {
      this.generateRandomValues();
      this.draw();
    }
  },
  methods: {
    draw() {
      try {
        const beta_mu_p_q = '(mu^(p-1))*((1-mu)^(q-1))*factorial(p+q-1)/(factorial(p-1)*factorial(q-1))';
        // compile the expression once
        const expr = math.compile(beta_mu_p_q)
        const p = 1 + this.m;
        const q = 1 + (this.range - this.m);

        // evaluate the expression repeatedly for different values of x
        const xValues = math.range(0, 1, 0.01).toArray()
        const yValues = xValues.map(function (x) {
          return expr.evaluate({
            mu: x,
            p: p,
            q: q,
          })
        });

        // render the plot using plotly
        const trace = {
          x: xValues,
          y: yValues,
          name: 'density',
          mode: 'lines',
          type: 'scatter'
        }
        
        const beta_mean = {
          x: [p/(p+q)],
          y: [0],
          name: 'beta mean',
          mode: 'markers',
          type: 'scatter'
        }

        const real_mean = {
          x: [this.realMu],
          y: [0],
          name: 'real mean',
          mode: 'markers',
          type: 'scatter'
        }

        const data = [trace, beta_mean, real_mean]
        Plotly.newPlot('plot', data)
      }
      catch (err) {
        console.error(err)
        alert(err)
      }
    },
    generateRandomValues() {
      const randomArray = math.random([this.number]);
      this.randomValues = randomArray.map(el => (el < this.realMu ? 1 : 0));
    }
  },
  created() {
    this.generateRandomValues();
  },
  mounted() {
    this.draw();
  }
}
</script>

<style>
.slider {
  -webkit-appearance: none;
  width: 100%;
  height: 15px;
  border-radius: 5px;  
  background: #d3d3d3;
  outline: none;
  opacity: 0.7;
  -webkit-transition: .2s;
  transition: opacity .2s;
}
</style>
