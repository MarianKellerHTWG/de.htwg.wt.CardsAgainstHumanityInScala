<template>
  <div class="d-flex align-items-center">
    <div class="container justify-centerContainer-center">
      <div class="container text-center">
        <div class="container align-items-center"><h1 id="current-question">Question Card goes here ____</h1></div>
        <div class="row">
          <div class="col align-items-center">
            <div class='played-cardstack'>
            </div>
            <p class="game-subtitle" id="player">player 1</p>
            <div class='cardstack'>
            </div>
          </div>
          <div class="col align-items-center">
            <div class='played-cardstack'>
            </div>
            <p class="game-subtitle" id="player">player 2</p>
            <div class='cardstack'>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from 'jquery'

let websocket = null;
let isLast = false;
let callOnMessage = null;

function connectWebSocket() {
  websocket = new WebSocket("ws://localhost:9000/websocket");
  websocket.onopen = function() {
    console.log("Connected to Websocket");
    getGamePage(updateGamePage, false);
  }

  websocket.onclose = function () {
    console.log('Connection with Websocket Closed!');
  };

  websocket.onerror = function (error) {
    console.log('Error in Websocket Occured: ' + error);
  };

  websocket.onmessage = function (e) {
    console.log(e.data)
    if (typeof e.data === "string") {
      if (callOnMessage != null) {
        callOnMessage(e.data, isLast)
      }
    }

  };
}

$(document).ready(function() {
  connectWebSocket()
});

function getGamePage(cFunction, last) {
  let query = {
    "action": "state",
    "message": ""
  }
  console.log(query);
  if(websocket.readyState == WebSocket.OPEN) {
    callOnMessage = cFunction;
    isLast = last;
    websocket.send(JSON.stringify(query));
  } else {
    console.log("Could not send data. Websocket is not open.");
    setTimeout(connectWebSocket, 500)
    setTimeout(function() {getGamePage(cFunction, last)}, 700)
  }
}

function evaluate(input, cFunction) {
  let query = {
    "action": "eval",
    "message": String(input)
  };
  console.log(query);
  if(websocket.readyState == WebSocket.OPEN) {
    callOnMessage = cFunction;
    websocket.send(JSON.stringify(query));
  } else {
    console.log("Could not send data. Websocket is not open.");
  }
}

function updateGamePage(data, last) {
  console.log(last.toString())
  let json_data = $.parseJSON(data);
  document.getElementById('current-question').innerHTML = json_data.game.roundQuestion;

  let playerTitles = document.querySelectorAll('[id^=player]');
  let i = 0;
  for (let player in playerTitles) {
    playerTitles[player].innerHTML = json_data.game.player[i].name;
    if (i === json_data.game.activePlayer) {
      playerTitles[player].classList.add("game-subtitle");
      playerTitles[player].classList.remove("game-subtitle-dark");
    } else {
      playerTitles[player].classList.add("game-subtitle-dark");
      playerTitles[player].classList.remove("game-subtitle");
    }
    i++;
    if (i >= json_data.game.numberOfPlayers) {
      break;
    }
  }

  let cardstacks = document.querySelectorAll('[class^=cardstack]');
  i = 0;
  for (let cardstack in cardstacks) {
    cardstacks[cardstack].innerHTML = "";
    for (let cardIndex = 0; cardIndex < json_data.game.player[i].playerCards.length; cardIndex++) {
      let newCard = document.createElement("div");
      newCard.id = cardIndex.toString();
      if (i === json_data.game.activePlayer) {
        if (i === json_data.game.numberOfPlayers-1) {
          newCard.addEventListener("click", function () {lastCardClicked(cardIndex)});
        } else {
          newCard.addEventListener("click", function () {cardClicked(cardIndex)});
        }
      }
      newCard.classList.add("light");
      newCard.classList.add("stackcard");
      let newCardContent = document.createElement("ul");
      newCardContent.classList.add("cardtext");
      newCardContent.classList.add("menu");
      let newCardContentLi = document.createElement("li");
      newCardContentLi.innerHTML = json_data.game.player[i].playerCards[cardIndex];
      newCardContent.append(newCardContentLi);
      newCard.append(newCardContent);
      cardstacks[cardstack].append(newCard);
    }
    i++;
    if (i >= json_data.game.numberOfPlayers) {
      break;
    }
  }
}

function cardClicked(card) {
  evaluate(card, function () {getPlayedCard(false)});
}

function lastCardClicked(card) {
  evaluate(card, function () {getPlayedCard(true)});
}

function getPlayedCard(last) {
  getGamePage(showPlayedCard, last);
}

function showPlayedCard(data, last) {
  let json_data = $.parseJSON(data);
  //alert(json_data.game.roundAnswerCards[json_data.game.roundAnswerCards.length-1].placedCard);

  let cardstacks = document.querySelectorAll('[class^=played-cardstack]');
  let current_card_index = json_data.game.roundAnswerCards.length-1;
  let i = 0;
  for (let cardstack in cardstacks) {
    if (i === current_card_index) {
      let newCard = document.createElement("div");
      newCard.classList.add("dark");
      newCard.classList.add("played-stackcard");
      let newCardContent = document.createElement("ul");
      newCardContent.classList.add("cardtext");
      newCardContent.classList.add("menu");
      let newCardContentLi = document.createElement("li");
      newCardContentLi.innerHTML = json_data.game.roundAnswerCards[i].placedCard;
      newCardContent.append(newCardContentLi);
      newCard.append(newCardContent);
      cardstacks[cardstack].append(newCard);
    }

    i++;
    if (i >= json_data.game.numberOfPlayers) {
      break;
    }
  }

  if (last) {
    nextCard();
  } else {
    updateGamePage(data, last);
  }
}

function nextCard() {
  evaluate("next", function () {getGamePage(updateGamePage,false)});
}

export default {
  name: 'GameScreenView',
  mounted() {
    connectWebSocket()
  }
}
</script>

<style>
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

.game-subtitle {
  color: #FFFFFF;
  font-size: 2em;
  animation: fadeIn ease-out 2s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-delay: 1s;
  opacity: 0;
}

.game-subtitle-dark {
  color: #555555 !important;
  font-size: 2em;
}

.game-name-list-width {
  min-width: 50%;
  max-width: 400px;
  margin: auto;
}

.game-name-list {
  min-width: 50%;
  max-width: 400px;
}

.game-name-list li {
  transition: all 0.3s ease-out;
  opacity: 0;
  height: 0;
}

.game-name-list li.show {
  opacity: 1;
  height: auto;
}

.vh-90{
  min-height:90vh;
}

.cardContainer{
  overflow:hidden;
  margin-left:10%;
}

.card{
  /* Card Sizing */
  width: 2.5in;
  height: 4in;
  border-radius: .25in;

  /* Display Properties */
  float: left;
  margin: 10px;
}

.dark{
  /* Color information */
  border: 3px solid white;
  color: white;
  background: black;
}

.light{
  /* Color information */
  border: 3px solid black;
  color: black;
  background: white;
}

.cardtext{
  /*font information*/
  font-family: Helvetica,sans-serif;
  font-weight: bolder;
  font-size: 26px;
  padding-left:7%;
  padding-right:7%;
  word-wrap: break-word;
}

.subtext{
  /*font information*/
  font-family: Helvetica,sans-serif;
  font-weight: bold;
  font-size: 16px;
  padding-left:7%;
  padding-right:7%;
  word-wrap: break-word;
}

.menu{
  list-style-type: none;
}

.played-cardstack{
  width:2.5in;
  float: top;
  margin: auto;
}

.played-stackcard{
  /* Card Sizing */
  width: 2.5in;
  height: 4in;
  border-radius: .25in;

  /* Display Properties */
  position:relative;
  margin: 10px;
}

.played-stackcard:hover{
  transition: all 0.8s;
  transform-style: preserve-3d;
  transform: rotateZ(10deg);
  translate: 50px;
  z-index: 1000;
}

.played-stackcard:first-child{
  margin: 10px;
  margin-top: 0in !important;
}

.played-stackcard:nth-child(n){
  margin: 10px;
  margin-top: -3in;
}

.cardstack{
  width:2.5in;
  float: top;
  margin: auto;
}

.stackcard{
  /* Card Sizing */
  width: 2.5in;
  height: 4in;
  border-radius: .25in;

  /* Display Properties */
  position:absolute;
  margin: 10px;
}

.stackcard:hover{
  transition: all 0.8s;
  transform-style: preserve-3d;
  transform: rotateZ(10deg);
  translate: 50px;
  z-index: 1000;
}

.stackcard:nth-child(2){
  margin: 10px;
  margin-top: 80px;
}

.stackcard:nth-child(3){
  margin: 10px;
  margin-top: 150px;
}

.stackcard:nth-child(4){
  margin: 10px;
  margin-top: 220px;
}

.stackcard:nth-child(5){
  margin: 10px;
  margin-top: 290px;
}

.stackcard:nth-child(6){
  margin: 10px;
  margin-top: 360px;
}

.stackcard:nth-child(7){
  margin: 10px;
  margin-top: 430px;
}

.stackcard:nth-child(8){
  margin: 10px;
  margin-top: 500px;
}

.stackcard:nth-child(9){
  margin: 10px;
  margin-top: 570px;
}
</style>