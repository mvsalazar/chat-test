<template>
  <div id="app">
    <chat-component
      firstName="John"
      lastName="Doe"
      emailAddress="jdoe@exampple.com"
    />
  </div>
</template>

<script>
import ChatComponent from './components/ChatComponent.vue';

export default {
  name: 'App',
  components: {
    ChatComponent
  }
};
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
