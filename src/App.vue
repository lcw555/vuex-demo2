<template>
  <div id="app">
    <OverViewHead></OverViewHead>
    <OverView></OverView>
  </div>
</template>

<script>
import OverViewHead from './components/OverViewHead.vue';
import OverView from './components/OverView.vue';

export default {
  name: 'app',
  components: {
    OverViewHead,
    OverView
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width:100%;
  height: 100%;
  position: absolute;
}
</style>
