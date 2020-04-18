<template>
  <div id="app">
    <RSA/>
  </div>
</template>

<script>
import RSA from './components/RSA.vue'

export default {
  name: 'App',
  components: {
    RSA
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
