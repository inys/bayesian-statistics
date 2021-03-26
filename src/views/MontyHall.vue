<template>
  <header class="header">
    <div class="container">
      <h1 class="is-size-1 title">Monty Hall Problem</h1>
      <div class="buttons">
        <button class="button is-warning"
          @click="resetDoors">Reset doors</button>
      </div>
    </div>
  </header>
  <section class="section">
    <div class="container">
      <div class="columns is-multiline">
        <div class="column is-12">
          <plotly :data="dataGraph" :layout="layout"></plotly>
        </div>
        <div class="column is-12">
          <div class="field">
            <label class="label">Number of open doors:</label>
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
            <button class="button" @click="incRange(-1)"><i>-1</i></button>
            <button class="button" @click="incRange(1)"><i>+1</i></button>
          </div>
          <hr>
          <p>Doors: {{ randomValues.slice(0,range) }}</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
// Plotly component
//import Plotly from 'plotly.js';
import Plotly from '@/components/Plotly.vue'

// some constants
const minRange = 0;
const maxRange = 8;
const defaultMu = 0.05;
const defaultAlpha = 2;
const defaultBeta = 100;

export default {
  name: 'MontyHall',
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
      layout: {
        xaxis: {
          title: "doors"
        },
        yaxis: {
          range: [0.0, 1.0],
          title: "probability"
        },
      }
    }
  },
  computed: {
    dataGraph() {
      var data = null;
      try {
        const xValues = [1];
        const yValues = [1/(this.number+2)];

        const openDoors = this.randomValues.slice(0, this.range);
        const closedDoors = this.randomValues.slice(this.range);

        openDoors.forEach(el => {
          xValues.push(el);
          yValues.push(0.0);
        })

        const propability = (this.number+1)/((this.number + 2)*(closedDoors.length))

        closedDoors.forEach(el => {
          xValues.push(el);
          yValues.push(propability);
        })

        // render the plot using plotly
        const trace = {
          x: xValues,
          y: yValues,
          name: 'probilities',
          type: 'bar'
        }

        data = [trace];
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
      const tris = Array.from({length: this.number + 1}, (v, i) => i + 2);
      for (let i = tris.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [tris[i], tris[j]] = [tris[j], tris[i]];
      }
      this.randomValues = tris;
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
    resetDoors() {
      this.generateRandomValues();
    }
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
