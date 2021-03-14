<template>
  <nav class="navbar is-transparent">
    <div class="navbar-brand">
      <router-link class="navbar-item" to="/">Bayesian statistics</router-link>
      <div class="navbar-burger" data-target="navbarExampleTransparentExample">
        <span></span>
        <span></span>
        <span></span>
      </div>
    </div>

    <div id="navbarExampleTransparentExample" class="navbar-menu">
      <div class="navbar-start">
        <div class="navbar-item">
          <router-link class="navbar-item" to="/cointoss">Coin toss</router-link>
        </div>
        <div class="navbar-item">
          <router-link class="navbar-item" to="/golfer">Golfer</router-link>
        </div>
      </div>
    </div>
  </nav>
  
  <router-view/>
</template>

<style>
</style>
