<template>
  <div class="hello">
    <h1>{{ message }}</h1>
    <button @click="changeMessage">Üzenet változtatása</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const message = ref('Hello, Vue!')

function changeMessage() {
  message.value = 'Üzenet megváltozott!'
}
</script>

<style scoped>
.hello {
  text-align: center;
  margin-top: 50px;
}
button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}
</style>
