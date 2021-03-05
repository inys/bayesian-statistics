<template>
  <input v-model="number"/>
  <p>Number of samples: {{ number }}</p>
  <div class="slidecontainer">
  <input 
    v-model="range"
    type="range"
    min="1"
    max="20"
    class="slider"
    id="myRange">
  </div>
  <p>Range: {{ range }}</p>
</template>

<script>
export default {
  name: 'App',
  components: {
  },
  data() {
    return {
      number: 20,
      range: 10,
    }
  }
}
</script>

<style>
</style>
