<template>
  <header class="header">
    <div class="container">
      <h1 class="is-size-1 title">The choice of the prior</h1>
      <div class="buttons">
        <button class="button is-danger"
          @click="resetParameters">Reset parameters</button>
      </div>
    </div>
  </header>
  <section class="section">
    <div class="container">
      <div class="columns is-multiline">
        <div class="column is-9">
          <plotly :data="dataGraph"></plotly>
        </div>
        <div class="column is-3">
          <h3 class="is-size-3">
            Prior 1:
          </h3>
          <p>Normal distribution with mean -2 and standard deviation 0.15.</p>
          <h3 class="is-size-3">
            Prior 2:
          </h3>
          <p>Student-t distribution with mean -2 and standard deviation 0.15.</p>
        </div>
        <div class="column is-12">
          <div class="field">
            <label class="label">Number of persons tested:</label>
            {{ numPersons }}
            <div class="control">
              <input 
                v-model="numPersons"
                type="range"
                min="0"
                :max="100"
                class="slider"
                id="myRange"/>
            </div>
          </div>
          <div class="field">
            <label class="label">Number of persons who are positive:</label>
            {{ posPersons }} of {{ numPersons }}
            <div class="control">
              <input 
                v-model="posPersons"
                type="range"
                min="0"
                :max="numPersons"
                class="slider"
                id="myRange"/>
            </div>
          </div>
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

// some constants
const dx = 0.005;

export default {
  name: 'PriorChoice',
  components: {
    Plotly
  },
  data() {
    return {
      numPersons: 20,
      posPersons: 18,
    }
  },
  computed: {
    dataGraph() {
      var data = null;
      try {
        // beta density
        const logit_expr = 'log10(pi/(1-pi))';
        const first_prior_expr = '10^(p*lambda)*exp(-(lambda-mu)^2/(2*sigma^2))/(1+10^lambda)^N';
        const second_prior_expr = '10^(p*lambda)(1+0.1*sigma^(-2)*(lambda-mu)^2)^(-11/2)/(1+10^lambda)^N';
        // compile the expression once
        const logit_fun = math.compile(logit_expr);
        const first_prior_fun = math.compile(first_prior_expr);
        const second_prior_fun = math.compile(second_prior_expr);

        const p = this.posPersons;
        const N = this.numPersons;

        // evaluate the expression repeatedly for different values of x
        const xValues = math.range(0, 1, dx).toArray()
        
        var yValues1 = xValues.map(function (x) {
          return first_prior_fun.evaluate({
            mu: -2.0,
            sigma: 0.15,
            lambda: logit_fun.evaluate({
              pi: x,
            }),
            p: p,
            N: N,
          })
        });
        const auc1 = math.sum(yValues1)*dx;
        yValues1 = yValues1.map(el => el/auc1);

        var yValues2 = xValues.map(function (x) {
          return second_prior_fun.evaluate({
            mu: -2.0,
            sigma: 0.15,
            lambda: logit_fun.evaluate({
              pi: x,
            }),
            p: p,
            N: N,
          })
        });
        const auc2 = math.sum(yValues2)*dx;
        yValues2 = yValues2.map(el => el/auc2);

        // render the plot using plotly
        const trace1 = {
          x: xValues,
          y: yValues1,
          name: 'posterior1 density',
          mode: 'lines',
          type: 'scatter'
        }

        const trace2 = {
          x: xValues,
          y: yValues2,
          name: 'posterior2 density',
          mode: 'lines',
          type: 'scatter'
        }

        data = [trace1, trace2];
      } catch (err) {
        console.error(err)
        alert(err)
      }

      return data
    }
  },
  watch: {
  },
  methods: {
    resetParameters() {
      this.numPersons = 20;
      this.posPersons = 18;
    },
  },
  created() {
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
