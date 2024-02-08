<template>
  <qrcodeShow/>
</template>

<script>
import qrcodeShow from './components/qrcodeShow.vue'

export default {
  name: 'App',
  components: {
    qrcodeShow
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
