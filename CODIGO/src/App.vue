<template>
  <div id="app">
    <SqliteViewer />
  </div>
</template>

<script>
import SqliteViewer from './components/SqliteViewer.vue'; 

export default {
  name: 'App',
  components: {
    SqliteViewer 
  }
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  font-family: Arial, Helvetica, sans-serif;
  background-color: hsl(230, 70%, 25%);
  text-align: center;
}
</style>
