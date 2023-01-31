<template>
  <div class="d-flex justify-content-center">
    <button @click="$emit('click')" type="button" class="start-btn" id="start_btn">{{buttonText}}</button>
  </div>
</template>

<script>
export default {
  name: 'ActionButton',
  props: {
    buttonText: String
  }
}
</script>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

.start-btn {
  width: 150px;
  height: 50px;
  border-radius: 25px;
  background-color: #FFFFFF;
  color: #000000;
  font-size: 1em;
  margin: 20px;
  opacity: 0;
  animation: fadeIn ease-out 2s;
  animation-iteration-count: 1;
  animation-fill-mode: forwards;
  animation-delay: 1s;
  transition: .4s ease-in-out;
}

.start-btn:hover {
  cursor: pointer;
  background-color: #000000;
  color: #FFFFFF;
  border-style: solid;
  border-color: #FFFFFF;
  border-width: 5px;
}
</style>
