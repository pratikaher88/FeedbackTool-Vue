<template>
  <div id="app">

    <FeedbackForm/>
    
  </div>
</template>

<script>

import FeedbackForm from "./components/FeedbackForm";


export default {
  name: 'App',
  components: {
    FeedbackForm
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  display: flex;
  align-items: center; 
  justify-content: center;
  flex-direction: column;

}
</style>
