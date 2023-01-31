<template>
  <div class="d-flex align-items-center vh-90">
    <div class="container justify-centerContainer-center">
      <div class="d-flex justify-content-center">
        <h1 class="index-title">Cards Against Humanity</h1>
      </div>
      <div class="d-flex justify-content-center">
        <p class="index-subtitle">A game for horrible people...</p>
      </div>

      <action-button @click="this.$router.push('newGame')" buttonText="Start Game" />
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import ActionButton from '@/components/ActionButton.vue'
import {msg} from "@babel/core/lib/config/validation/option-assertions";

export default {
  name: 'HomeView',
  methods: {msg},
  components: {
    ActionButton
  }
}
</script>

<style scoped>
@keyframes dropIn {
  from {
    opacity: 0;
    transform: translate3d(0, -200%, 0) rotateX(1170deg) rotateY(10deg);
  }
  30% {
    opacity: 1;
    transform: translate3d(0, +30%, 0) rotateX(80deg);
  }
  50% {
    transform: translate3d(0, +30%, 0) rotateX(88deg);
  }
  60% {
    transform: translate3d(0, +30%, 0) rotateX(90deg);
  }
  to {
    transform: translate3d(0, 0, 0) rotateX(0deg);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.index-title {
  font-size: 3em;
  font-weight: bold;
  animation: dropIn ease-out 3s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
}

.index-subtitle {
  font-size: 2em;
  animation: fadeIn ease-out 2s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-delay: 2s;
  opacity: 0;
}

.vh-90 {
  min-height: 90vh;
}
</style>