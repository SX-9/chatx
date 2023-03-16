<script setup>
import { ref } from "vue";
import { getFirestore, updateDoc, deleteDoc, collection, doc as docId, } from "firebase/firestore";
import Markdown from 'vue3-markdown-it';
const props = defineProps({
  img: String,
  msg: String,
  doc: String,
  user: String,
  time: Number,
  system: Boolean,
});

function getRandomColor() {
  var letters = "0123456789ABCDEF";
  var color = "#";
  for (var i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

document
  .querySelectorAll(".title")
  .forEach((el) => (el.style.color = getRandomColor()));

const msgs = collection(getFirestore(), 'messages');
const edit = ref(() => {
  let { msg, doc, user, time } = props;
  let newMsg = prompt('Edit Message: ', msg);
  if (!newMsg) return;
  
  updateDoc(docId(msgs, doc), {
    msg: newMsg, author: user, created: time
  });
});
const rm = ref(() => {
    let { doc } = props;
  let yn = confirm('Delete Message?');
  if (yn) deleteDoc(docId(msgs, doc));
});
</script>

<template>
  <div class="container">
    <div v-if="user === 'SX-9'" class="shape owner">|</div>
    <div v-else class="shape">|</div>
    <h2 class="title">
      {{ user }}
      <span class="time"
        >{{ new Date(time).getMonth() }}/{{ new Date(time).getDate() }}/{{
          new Date(time).getFullYear()
        }}</span
      >
    </h2>
    <Markdown class="msg" v-if="msg" :source="msg"/>
    <img class="img" v-if="img" :src="img" alt="Photo Message" loading="lazy">
    <p v-if="system" class="controls">
      <a id="edit" @click="edit()">Edit</a>
      /
      <a id="rm" @click="rm()">Delete</a>
    </p>
  </div>
</template>

<style scoped>
.container {
  display: grid;
  margin: 1em;
  grid-template-areas:
    "shape title"
    "shape msg"
    "shape img"
    "actions actions";
  grid-template-columns: 1.5em 1fr;
}

.shape {
  grid-area: shape;
  font-size: 2.7em;
  color: grey;
}
.shape.owner {
  color: yellow;
}
.title {
  grid-area: title;
}
.time {
  color: grey;
  font-size: 0.7em;
}
.msg {
  grid-area: msg;
}

.img {
  grid-area: img;
  width: 95%;
  max-width: 40rem;
  border: .1em solid grey;
  margin: .5em;
}

.controls {
  grid-area: actions;
}

#edit { color: lightblue; }
#rm { color: pink; }
#edit, #rm { cursor: pointer; }

</style>
