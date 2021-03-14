<template>
  <header class="header">
    <div class="container">
      <h1 class="is-size-1 title">Inference of the success probability of a golfer</h1>
      <div class="buttons">
        <button class="button is-info"
          @click="newExperiment">New experiment</button>
        <button class="button is-danger"
          @click="resetExperiment">Reset experiment</button>
      </div>
    </div>
  </header>
  <section class="section">
    <div class="container">
      <div class="columns is-multiline">
        <div class="column is-2">
          <h3 class="is-size-5">The golfer</h3>
          <div class="field">
            <label class="label">Success probability:</label>
            {{ realMu * 100 }} %
            <div class="control">
              <input
                v-model="realMu"
                type="range"
                min="0.01"
                max="0.1"
                step="0.001"
                class="slider"/>
            </div>
          </div>
        </div>
        <div class="column is-8">
          <plotly :data="dataGraph"></plotly>
        </div>
        <div class="column is-2">
          <h3 class="is-size-5">The prior distribution</h3>
          <p>Beta distribution with parameters</p>
          <div class="field">
            <label class="label">alpha:</label>
            <div class="control">
              <input
                v-model="alpha"
                class="input"
                type="text">
            </div>
          </div>
          <div class="field">
            <label class="label">beta:</label>
            <div class="control">
              <input
                v-model="beta"
                class="input"
                type="text">
            </div>
          </div>
        </div>
        <div class="column is-12">
          <div class="field">
            <label class="label">Number of samples:</label>
            {{ range }}
            <div class="control">
              <input 
                v-model="range"
                type="range"
                min="0"
                :max="number"
                class="slider"
                id="myRange"/>
            </div>
          </div>
          <div class="buttons">
            <button class="button" @click="incRange(-10)"><i>-10</i></button>
            <button class="button" @click="incRange(-1)"><i>-1</i></button>
            <button class="button" @click="incRange(1)"><i>+1</i></button>
            <button class="button" @click="incRange(10)"><i>+10</i></button>
          </div>
          <hr>
          <p>Outcome: {{ randomValues.slice(0,range) }}</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
// mathjs
import { create, all } from 'mathjs'

// specify mathjs configuration
const config = {
  epsilon: 1e-12,
  matrix: 'Matrix',
  number: 'number',
  precision: 64,
  predictable: false,
  randomSeed: 2021
}

// create just the functions we need
const math = create(all, config);

// Plotly component
//import Plotly from 'plotly.js';
import Plotly from '@/components/Plotly.vue'

const d3 = require("d3-random");

// some constants
const minRange = 0;
const maxRange = 50;
const defaultMu = 0.05;
const defaultAlpha = 2;
const defaultBeta = 100;

export default {
  name: 'Golfer',
  components: {
    Plotly
  },
  data() {
    return {
      number: maxRange,
      range: minRange,
      randomValues: [],
      realMu: defaultMu,
      alpha: defaultAlpha,
      beta: defaultBeta,
    }
  },
  computed: {
    dataGraph() {
      var data = null;
      try {
        // beta density
        const beta_mu_p_q = '(mu^(p-1))*((1-mu)^(q-1))*combinations(p+q,p)*p*q/(p+q)';
        // compile the expression once
        const expr = math.compile(beta_mu_p_q)

        const subRange = this.randomValues.slice(0, this.range);
        const p = parseInt(this.alpha) + subRange.length;
        const q = parseInt(this.beta) + math.sum(subRange);

        // evaluate the expression repeatedly for different values of x
        const xValues = math.range(0.0, 0.1, 0.0005).toArray()
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
          name: 'posterior density',
          mode: 'lines',
          type: 'scatter'
        }

        const beta_mean = {
          x: [p/(p+q)],
          y: [0],
          name: 'posterior mean',
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

        data = [trace, beta_mean, real_mean];
      } catch (err) {
        console.error(err)
        alert(err)
      }

      return data
    }
  },
  watch: {
    number: function(value) {
      const intValue = parseInt(value);

      if (isNaN(intValue)) {
        return
            }

            if (intValue <= 0) {
                return
            }

      this.generateRandomValues();
    },
    realMu: function() {
      this.generateRandomValues();
    },
  },
  methods: {
    generateRandomValues() {
      const tris = Array.from({length: this.number}, d3.randomGeometric(this.realMu))
      this.randomValues = tris.map(el => (el - 1))
    },
    newExperiment() {
      this.generateRandomValues();
      this.draw();
    },
    resetExperiment() {
      this.range = minRange;
      this.realMu = defaultMu;
      this.alpha = defaultAlpha;
      this.beta = defaultBeta;
      this.generateRandomValues();
      this.draw();
    },
    incRange(step) {
      if (this.range + step >= maxRange) {
        this.range = maxRange;
      } else if (this.range + step <= minRange) {
        this.range = minRange;
      } else {
        this.range = this.range + step;
      }
    },
  },
  created() {
    this.generateRandomValues();
  },
  mounted() {
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
