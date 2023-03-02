<script setup>
import { createApp, h, ref } from 'vue';
import Mess from './components/Mess.vue';
import swears from './swears.json';

// TODO: Firebase Realtime Database

let bannedWords = swears.concat('http', 'www', 'com');
document.querySelector('title').innerText = 'Vue Firebase Chat';

function isASCII(str, extended) {
  return (extended ? /^[\x00-\xFF]*$/ : /^[\x00-\x7F]*$/).test(str);
}

function msgComponentCreate(msg, user, system) {
  let msgComponent = createApp({ 
    setup () {
      return () => h(Mess, {msg, user, system});
    }
  });

  let wrapper = document.createElement("div");
  msgComponent.mount(wrapper);
  document.querySelector('#messages').appendChild(wrapper);
}

const isAuth = ref(true);
const username = ref('???');
const authToggle = ref(() => {
  if (isAuth.value) {
    isAuth.value = false;
  } else {
    isAuth.value = true;
  }
});

let timeout = false;
const msgCreate = ref(() => {
  if (timeout) return alert('Slowdown, Theres a 3 Second Timeout!');
  timeout = true;
  setTimeout(() => timeout = false, 3000);

  let message = document.querySelector('#msg-input').value;
  if (message.length === 0) return alert('Enter A Message');
  if (!isASCII(message, true)) return alert('Message Contains Non ASCII Characters');
  
  let stop = false;
  bannedWords.forEach(banned => {
    if (stop) return;
    if (message.toLowerCase().includes(banned.toLowerCase())) stop = true;
  });
  if (stop) return alert('Swear Word Detected!');
  
  msgComponentCreate(message, username.value, false);
});

window.onkeypress = e => {
  if (e.key === 'Enter') {
    msgCreate.value();
  }
}
window.addEventListener('DOMContentLoaded', () => msgComponentCreate('Sign In To Chat!', 'Warning', true));
</script>

<template>
<div id="topbar">
  <h1><a href="https://github.com/SX-9/vf-chat">VF-Chat</a></h1>
  <button @click="authToggle">
    <span v-if="isAuth">Log Out</span>
    <span v-else>Log In</span>
  </button>
  <h1 v-if="isAuth">{{ username }}</h1>
</div>
<div id="messages"></div>
<p id="end">The End!</p>
<div id="inputs">
  <button><a href="#end">👇</a></button>
  <input id="msg-input" type="text" placeholder="Hello World, Type Here..."/>
  <button @click="msgCreate">✈️</button>
</div>
</template>

<style>
h2, p {
  margin-top: .1rem;
  margin-bottom: .1rem;
}
h1 { margin: .1em; }
a { text-decoration: none; }

#messages { margin-top: 5em; }
#end { text-align: center; padding-bottom: 4em; }

#inputs, #topbar {
  position: fixed;
  left: 0;
  height: 3em;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1em;
}
#topbar { top: 1em; background: #000000a2; }

#inputs { bottom: 1em; }
#inputs button { width: 5em; }
#inputs input { width: 60%; }


input, button {
  color: white;
  background: #000000a2;
  border: .1em solid #ffffff;
  padding: 1em;
  border-top-left-radius: .5em;
  border-top-right-radius: .5em;
  border-bottom-left-radius: .5em;
  border-bottom-right-radius: .5em;
}
</style>
