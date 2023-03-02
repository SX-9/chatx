<script setup>
defineProps({
  msg: String,
  user: String,
  system: Boolean,
});

function getRandomColor() {
  var letters = '0123456789ABCDEF';
  var color = '#';
  for (var i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

document.querySelectorAll('.title').forEach(
  el => el.style.color = getRandomColor()
);
</script>

<script>
function getRandomColor() {
  var letters = '0123456789ABCDEF';
  var color = '#';
  for (var i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

document.addEventListener('DOMContentLoaded', () => {
  document.querySelectorAll('.title').forEach(
    el => el.style.color = getRandomColor()
  );
});
</script>

<template>
  <div class="container">
    <div class="shape">|</div>
    <h2 class="title">{{ user }} {{ system ? '- System' : '' }}</h2>
    <p class="msg">{{ msg }}</p>
  </div>
</template>

<style scoped>
.container {
  display: grid;
  margin: 1em;
  grid-template-areas: 
    'shape title'
    'shape msg';
  grid-template-columns: 1.5em 1fr;
}

.shape {
  grid-area: shape;
  font-size: 2.7em;
  color: grey;
}
.title { grid-area: title; }
.msg { grid-area: msg; }
</style>