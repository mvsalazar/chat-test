<template>
  <div id="app">
    <chat-component
      firstName="John"
      lastName="Doe"
      emailAddress="jdoe@exampple.com"
      businessLine="Billing"
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
}
</style>
