<template>
  <div id="app">
    <HelloWorld />
  </div>
</template>

<script>
import HelloWorld from './index.vue'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
  *{
    margin: 0;
    padding: 0;
  }
  #app{
    height: 100%;
    position: absolute;
    width: 100%;
  }
</style>
