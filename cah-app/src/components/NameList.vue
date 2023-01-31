<template>
  <div class="d-flex justify-content-center input-group mb-3 game-name-list-width">
    <button @click="addName" class="btn btn-outline-secondary" type="button" id="add_name_button">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor"  class="bi bi-plus-lg" viewBox="0 0 16 16">
        <path fill-rule="evenodd" d="M8 2a.5.5 0 0 1 .5.5v5h5a.5.5 0 0 1 0 1h-5v5a.5.5 0 0 1-1 0v-5h-5a.5.5 0 0 1 0-1h5v-5A.5.5 0 0 1 8 2Z"/>
      </svg>
    </button>
    <input type="text" class="form-control" placeholder="" id="name-textfield">
  </div>
  <div class="d-flex justify-content-center">
    <ul class="list-group game-name-list-width game-name-list" id="game-name-list"></ul>
  </div>
</template>

<script>
export default {
  name: 'NameList',
  methods: {
    addName() {
        let nameField = document.getElementById("name-textfield")
        let list = document.getElementById("game-name-list")
        let li = document.createElement("li");
        let name = nameField.value;
        if (name !== "") {
          this.$emit('update', name)
          nameField.value = "";

          li.appendChild(document.createTextNode(name));
          li.setAttribute("class", "list-group-item");
          list.appendChild(li);
          setTimeout(function() {
            li.className = li.className + " show";
          }, 10);
        }
    }
  }
}
</script>

<style scoped>
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

</style>