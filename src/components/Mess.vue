<script setup>
import { ref } from "vue";
import {
  getFirestore,
  updateDoc,
  deleteDoc,
  getDoc,
  setDoc,
  collection,
  doc as docId,
} from "firebase/firestore";
import Markdown from "vue3-markdown-it";
const props = defineProps({
  img: String,
  msg: String,
  doc: String,
  user: String,
  time: Number,
  system: Boolean,
});

const msgs = collection(getFirestore(), "messages");
const likesRef = collection(getFirestore(), "likes");
const likes = ref(0);

if (props.doc)
  getDoc(docId(likesRef, props.doc))
    .then((d) => {
      if (d.exists()) return (likes.value = d.data().likes);
      setDoc(docId(likesRef, props.doc), {
        likes: 0,
      });
      likes.value = 0;
    })
    .catch(console.error);

const edit = ref(() => {
  let { msg, doc, user, time } = props;
  let newMsg = prompt("Edit Message: ", msg);
  if (!newMsg) return;

  updateDoc(docId(msgs, doc), {
    msg: newMsg,
    author: user,
    created: time,
  });
});
const rm = ref(() => {
  let { doc } = props;
  let yn = confirm("Delete Message?");
  if (yn) {
    deleteDoc(docId(msgs, doc));
    deleteDoc(docId(likesRef, doc));
  }
});
const like = ref(() => {
  let { doc } = props;
  likes.value++;
  updateDoc(docId(likesRef, doc), {
    likes: likes.value,
  }).catch(() => likes.value--);
});
</script>

<template>
  <div class="container">
    <div v-if="user === 'SX-9'" class="shape owner">|</div>
    <div v-else-if="user === 'You'" class="shape you">|</div>
    <div v-else class="shape">|</div>
    <h2 class="title">
      {{ user }}
      <span class="time"
        >{{ new Date(time).getMonth() }}/{{ new Date(time).getDate() }}/{{
          new Date(time).getFullYear()
        }}
        {{ new Date(time).getHours() }}:{{ new Date(time).getMinutes() }}</span
      >
    </h2>
    <Markdown class="msg" v-if="msg" :source="msg" />
    <img class="img" v-if="img" :src="img" alt="Photo Message" loading="lazy" />
    <p class="controls">
      <span v-if="system">
        <a id="edit" @click="edit()">🖊 Edit</a>
        <a id="rm" @click="rm()">🗑 Delete</a>
      </span>
      <a @click="like()" id="likes"
        >👍
        {{
          new Intl.NumberFormat("en", { notation: "compact" }).format(likes) ||
          0
        }}</a
      >
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
.shape.you {
  color: lime;
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
  border: 0.1em solid grey;
  margin: 0.5em;
}

.controls {
  grid-area: actions;
}

#edit {
  color: lightblue;
}
#rm {
  color: pink;
}
#likes {
  color: yellow;
}
a {
  cursor: pointer;
  margin: 0.5em;
}
</style>
