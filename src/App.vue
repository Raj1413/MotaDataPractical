<template>
  <HomeMain msg="Welcome to Your Vue.js App"/>
</template>

<script>
import HomeMain from './components/HomeMain.vue';
// import Home from './components/HomeMain.vue'

export default {
  name: 'App',
  components: {
    HomeMain
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
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
