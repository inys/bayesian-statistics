<template>
  <header class="header">
    <div class="container">
      <h1 class="is-size-1 title">Inference of the real probability by tossing a coin</h1>
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
          <h3 class="is-size-5">The coin</h3>
          <div class="field">
            <label class="label">Real mean:</label>
            {{ realMu }}
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
        </div>
        <div class="column is-8">
          <div id="plot"></div>
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

const minRange = 0;
const maxRange = 200;
const defaultMu = 0.3;
const defaultAlpha = 1;
const defaultBeta = 1;

export default {
  name: 'App',
  components: {
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
            this.draw();
        },
        range: function() {
            this.draw();
        },
        realMu: function() {
            this.generateRandomValues();
            this.draw();
        },
        alpha: function() {
            this.draw();
        },
        beta: function() {
            this.draw();
        }
    },
    methods: {
        draw() {
            try {
                const beta_mu_p_q = '(mu^(p-1))*((1-mu)^(q-1))*combinations(p+q,p)*p*q/(p+q)';
                // compile the expression once
                const expr = math.compile(beta_mu_p_q)
                const m = math.sum(this.randomValues.slice(0, this.range));
                const p = parseInt(this.alpha) + m;
                const q = parseInt(this.beta) + (this.range - m);

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
