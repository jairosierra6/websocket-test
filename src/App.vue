<template>
  <div id="app">
    <h2>Lambda + Vue.js WebSocket test</h2> 
    <button v-on:click="sendMessage('hello')">Send Message</button>
  </div>
</template>

<script>
export default {
  name: 'App',
  data: function() {
    return {
      connection: null,
      socketUrl: 'wss://tnpbcownl8.execute-api.us-east-2.amazonaws.com/dev'
    }
    
  },
  methods: {
    sendMessage: function(message) {
      console.log(this.connection);
      this.connection.send(message);
    }
  },
  created: function() {
    console.log("Starting connection to WebSocket Server")
    this.connection = new WebSocket(this.socketUrl);

    this.connection.onmessage = function(event) {
      console.log(event);
    }
    this.connection.onopen = function(event) {
      console.log(event)
      console.log("Successfully connected to the echo websocket server...")
    }
    this.connection.onclose = function(event) {
      console.log('closed connection: ', event);
    }
    this.connection.onerror = function(event) {
      console.log('error: ', event);
    }

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
