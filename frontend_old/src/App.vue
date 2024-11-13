<template>
    <HelloWorld msg="Welcome to Your Vue.js App"/>
</template>

<script>
import HelloWorld from './components/HomeMap.vue'


export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
}
#app {
  height: 100vh;
  width: 100vw;
}
</style>
