<template>
  <div id="app">
    <VoteChain />
  </div>
</template>

<script>
import VoteChain from './components/VoteChain.vue'

export default {
  name: 'App',
  components: {
    VoteChain
  }
}
</script>

<style>

@import 'https://www.w3schools.com/w3css/4/w3.css';
@import 'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.3.0/css/font-awesome.min.css';
/*
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
body {
  background-color: #ddd;
}
.container {
  background-color: #eee;
  width: 560px;
  margin: auto;
  padding: 10px;
  border-radius: 10px;
  box-shadow: #9a9a9a 2px 2px 3px;
} */
</style>
