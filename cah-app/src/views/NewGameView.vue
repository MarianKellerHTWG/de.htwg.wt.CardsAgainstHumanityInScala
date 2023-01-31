<template>
  <div class="d-flex align-items-center vh-90" id="new-game-screen">
    <div class="container justify-centerContainer-center">
      <div class="d-flex justify-content-center">
        <h1 class="game-title">Cards Against Humanity</h1>
      </div>
      <div class="d-flex justify-content-center">
        <p class="game-subtitle">Please enter your names:</p>
      </div>

      <name-list v-on:update="onNameListUpdate"></name-list>

      <action-button @click="startGame" buttonText="Start Game" />
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import ActionButton from '@/components/ActionButton.vue'
import NameList from '@/components/NameList.vue'

let websocket = null;
let playerNameList = [];
let transmittedNames = 0;

export default {
  name: 'NewGameView',
  components: {
    ActionButton,
    NameList
  },
  mounted() {
    this.connectWebSocket();
  },
  methods: {
    onNameListUpdate (name) {
      playerNameList.push(name);
    },
    startGame() {
      if (playerNameList.length <= 0 && transmittedNames == 0) {
        alert("You seem to be quite alone, please add some players.");
        return;
      } else {
        this.evaluate(playerNameList.length.toString(), this.startGame2);
      }
    },
    connectWebSocket() {
      websocket = new WebSocket("ws://localhost:9000/websocket");
      websocket.onopen = function() {
        console.log("Connected to Websocket");
      }

      websocket.onclose = function () {
        console.log('Connection with Websocket Closed!');
      };

      websocket.onerror = function (error) {
        console.log('Error in Websocket Occured: ' + error);
      };

      websocket.onmessage = function (e) {
        if (typeof e.data == "string") {
          //getGamePage(e.data)
        }
      };
    },
    evaluate(input, cFunction) {
      let query = {
        "action": "eval",
        "message": input
      }
      console.log(query)
      if(websocket.readyState == WebSocket.OPEN) {
        websocket.send(JSON.stringify(query));
        cFunction();
      } else {
        console.log("Could not send data. Websocket is not open.");
      }
    },
    startGame2() {
      this.evaluate("weiter", this.startGame3);
    },
    startGame3() {
      if (playerNameList.length > 0) {
        transmittedNames += 1;
        this.evaluate(playerNameList.pop(), this.startGame3);
      } else {
        this.$router.push('gameScreen');
      }
    }
  }
}
</script>

<style scoped>
body {
  background-color: #000000;
  font-family: Arial, Helvetica, sans-serif;
}

h1 {
  color: #FFFFFF;
  font-family: Arial, Helvetica, sans-serif;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.game-title {
  font-size: 3em;
  font-weight: bold;
  animation: fadeIn ease-out 3s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}

.game-subtitle {
  color: #FFFFFF;
  font-size: 2em;
  animation: fadeIn ease-out 2s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-delay: 1s;
  opacity: 0;
}

.vh-90 {
  min-height: 90vh;
}
</style>