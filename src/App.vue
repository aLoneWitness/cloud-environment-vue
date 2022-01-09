<template>
  <div id="app">
    <head>
      <link rel="stylesheet" href="node_modules/xterm/css/xterm.css" />
    </head>
    <button @click="sendMessage()">Test WS</button>
    <div id="terminal"></div>
  </div>
</template>

<script>
import { io } from 'socket.io-client'
import { Terminal } from 'xterm'

export default {
  name: 'App',
  created() {
    this.socket = io('http://localhost:3000')
    this.socket.on("connect", () => {
      console.log(this.socket.connected); // true
      window.setInterval(this.triggerHearthbeat, 5000);
    });
  },
  mounted() {
    var term = new Terminal();
    term.open(document.getElementById('terminal'))

    let data = 'DQ==';
    let buff = new Buffer(data, 'base64');
    let text = buff.toString('ascii');
    console.log('b64 decode: ' + text)

    this.socket.on('stdout', (raw) => {
      // TODO: bruh
      let stringyJson = JSON.parse(raw)
      let json = JSON.parse(stringyJson)
      let decodedStdout = atob(json.stdout.data);
      term.write(decodedStdout)
    })

    term.onData((data) => {
      let buff = new Buffer(data);
      let base64data = buff.toString('base64');
      this.socket.emit('stdin', { stdin: { data: base64data } })
    });

    // term.onLineFeed((data) => {
    //   console.log(data)
    // })
  },
  methods: {
    sendMessage() {
      console.log('sending')
      // const lol = {
      //   stdin: {
      //     data: "DQ=="
      //   }
      // }
      let buff = new Buffer('/r');
      let base64data = buff.toString('base64');
      const lol = {
        stdin: {
          data: base64data
        }
      }
      this.socket.emit('stdin', lol)
    },
    triggerHearthbeat() {
      this.socket.emit('heartbeat', {})
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
