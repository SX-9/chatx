<script setup>
import { createApp, h, ref } from "vue";
import Mess from "./components/Mess.vue";
import swears from "./swears.json";
import { initializeApp } from "firebase/app";
import {
  getAuth,
  signInWithRedirect,
  GithubAuthProvider,
  GoogleAuthProvider,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut,
  updateProfile,
} from "firebase/auth";
import {
  getFirestore,
  addDoc,
  updateDoc,
  onSnapshot,
  collection,
  orderBy,
  query,
  limit,
  doc,
  disableNetwork,
} from "firebase/firestore";
import { v5 as uuid } from "uuid";

// TODO: Make API

initializeApp({
  apiKey: "AIzaSyC17Jru5AC4145DIcoOa5W-cxTm7Phj0CY",
  authDomain: "vf-chat-x.firebaseapp.com",
  projectId: "vf-chat-x",
  storageBucket: "vf-chat-x.appspot.com",
  messagingSenderId: "81255262067",
  appId: "1:81255262067:web:05a65355ac8bfd641bc706",
});

const db = getFirestore();
const msgRefs = collection(db, "messages");
const likesRef = collection(db, "likes");
const typingRef = doc(collection(db, "vars"), "typing");
const lockedRef = doc(collection(db, "vars"), "locked");

const auth = getAuth();
const ownerUid = "BRzxfCztjaQN6J2CKgYdp62ggnF2";
const actionCodeSettings = {
  url: "https://vf-chat-x.web.app",
  dynamicLinkDomain: "vf-chat-x.web.app",
};

const isAuth = ref(false);
const showForm = ref(false);
const username = ref("???");
const uid = ref("???");
const authToggle = ref(() => {
  if (isAuth.value) {
    signOut(auth);
  } else {
    let method = prompt(
      `
Select A Sign In Method:

1. Github
2. Google
3. Email
    `,
      "3"
    );
    let provider;
    if (method == 1) {
      provider = new GithubAuthProvider();
    } else if (method == 2) {
      provider = new GoogleAuthProvider();
    } else if (method == 3) {
      showForm.value = true;
      return;
    } else {
      alert("Invalid");
      return;
    }
    signInWithRedirect(auth, provider);
  }
});
const emailAuth = ref((e) => {
  let email = document.querySelector("input[name=email]").value;
  let pass = document.querySelector("input[name=pass]").value;

  e.preventDefault();
  signInWithEmailAndPassword(auth, email, pass)
    .then((userCredential) => {
      const user = userCredential.user;
    })
    .catch((error) => {
      if (error.code === "auth/user-not-found") {
        if (confirm("Account Not Found, Create A New One?")) {
          createUserWithEmailAndPassword(auth, email, pass);
        }
      }
      console.error(`
Error: ${error.code}

${error.message}
      `);
    });
  showForm.value = false;
});

auth.onAuthStateChanged((user) => {
  if (user) {
    if (!user.displayName && !user.reloadUserInfo.screenInfo) {
      let name = prompt("Username:");
      updateProfile(auth.currentUser, {
        displayName: name !== "SX-9" || name !== "You" || !isASCII(name) || !name ? name : randomUsername(5),
      })
        .then(() => window.location.reload())
        .catch(console.error);
    }
    username.value = user.reloadUserInfo.screenName || user.displayName;
    isAuth.value = true;
    uid.value = user.uid;
    if (localStorage.getItem("new")) return;
    alert(`
Welcome ${username.value}, Looks Like You Are New!

We Have A Ton Of Features:
1. Image Support
2. Markdown Support
3. Random Name Color
4. Typing Indicators
5. Message Likes
6. Admin Dashboard
7. API (Soon)
    `);
    localStorage.setItem("new", "true");
  } else {
    username.value = "???";
    isAuth.value = false;
    uid.value = "???";
  }
});

function randomUsername(length) {
  let chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  let randomString;
  for (let i = 0; i < length; i++) {
    randomString += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return "User" + randomString.replace("undefined", "");
}

function isASCII(str, extended) {
  return (extended ? /^[\x00-\xFF]*$/ : /^[\x00-\x7F]*$/).test(str);
}

function msgComponentCreate(msg, user, system, time, doc, img) {
  let msgComponent = createApp({
    setup() {
      return () => h(Mess, { msg, user, system, time, doc, img });
    },
  });

  let wrapper = document.createElement("div");
  msgComponent.mount(wrapper);
  document.querySelector("#messages").appendChild(wrapper);
}

function getRandomColor() {
  var letters = "0123456789ABCDEF";
  var color = "#";
  for (var i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

const locked = ref(false);
onSnapshot(lockedRef, async (e) => {
  locked.value = e.data().active
  if (!e.data().active || uid.value === ownerUid) return;
  alert('Error: Unable To Reach ChatX Servers, This Is Usually Due To It Being Locked.');
  await disableNetwork(db);
});

const lockChat = ref(() => {
  if (confirm('Lock The Chat? This Will Disconnect All Clients!'))
  updateDoc(lockedRef, { active: !locked.value });
});

const updateMsg = (snapshot) => {
  document.querySelector("#messages").innerHTML = "";
  console.log(snapshot)
  snapshot.docs.reverse().forEach((doc) => {
    msgComponentCreate(
      doc.data().msg,
      doc.data().author === username.value ? "You" : doc.data().author,
      uid.value === ownerUid,
      doc.data().created,
      doc.id,
      doc.data().img
    );
  });
  document
    .querySelectorAll("h2")
    .forEach((el) => (el.style.color = getRandomColor()));
  window.scrollTo(0, document.body.scrollHeight || document.documentElement.scrollHeight);
};
onSnapshot(query(msgRefs, orderBy("created", "desc"), limit(15)), updateMsg);

let timeout = false;
const msgCreate = ref(() => {
  let message = document.querySelector("#msg-input").value;

  if (uid.value !== ownerUid) {
    if (timeout) return alert("Slowdown, Theres a 5 Second Timeout!");
    timeout = true;
    setTimeout(() => (timeout = false), 5000);

    if (message.length === 0) return alert("Enter A Message");
    if (!isASCII(message, true))
      return alert("Message Contains Non ASCII Characters");

    let stop = false;
    swears.forEach((banned) => {
      if (stop) return;
      if (message.toLowerCase().includes(banned.toLowerCase())) stop = true;
    });
    if (stop) return alert("Banned Word Detected!");
  }

  addDoc(msgRefs, {
    msg: message,
    author: username.value,
    created: Date.now(),
    img: "",
  });
  addDoc(likesRef, { likes: 1 });

  document.querySelector("#msg-input").value = "";
});
const imgCreate = ref(() => {
  let message = document.querySelector("#msg-input").value;
  let imgUrl = prompt(
    "Image Url:",
    "https://static3.makeuseofimages.com/wordpress/wp-content/uploads/2010/10/HTML-Code-Examples-Featured.jpg"
  );

  if (uid.value !== ownerUid) {
    if (timeout) return alert("Slowdown, Theres a 5 Second Timeout!");
    timeout = true;
    setTimeout(() => (timeout = false), 5000);

    if (!isASCII(message, true))
      return alert("Message Contains Non ASCII Characters");

    let stop = false;
    swears.forEach((banned) => {
      if (stop) return;
      if (message.toLowerCase().includes(banned.toLowerCase())) stop = true;
    });
    if (stop) return alert("Banned Word Detected!");

    if (imgUrl.length === 0) return alert("Enter An Image Url");
    if (!isASCII(imgUrl, true))
      return alert("Image Url Contains Non ASCII Characters");
    if (!imgUrl.startsWith("http")) return alert("Invalid Image Url");
  }

  addDoc(msgRefs, {
    msg: message,
    author: username.value,
    created: Date.now(),
    img: imgUrl,
  });
  addDoc(likesRef, { likes: 1 });

  document.querySelector("#msg-input").value = "";
});

window.onkeypress = (e) => {
  if (e.key === "Enter") {
    msgCreate.value();
  }
};

const typing = ref(false);
onSnapshot(typingRef, (t) => {
  typing.value = t.data().active;
});

let typingTimeout;
const typingStart = ref((e) => {
  if (e.key === "Enter")
    return updateDoc(typingRef, {
      active: false,
    });
  updateDoc(typingRef, {
    active: true,
  });
  if (typingTimeout != undefined) clearTimeout(typingTimeout);
  typingTimeout = setTimeout(
    () =>
      updateDoc(typingRef, {
        active: false,
      }),
    3000
  );
});
</script>

<template>
  <div id="bg"></div>
  <div id="topbar" class="fadeTop">
    <h1><a href="https://github.com/SX-9/vf-chat">ChatX</a></h1>
    <button @click="authToggle">
      <span v-if="isAuth">Log Out</span>
      <span v-else>Log In</span>
    </button>
    <h1 v-if="isAuth" id="username">{{ username }}</h1>
  </div>
  <div v-if="typing && isAuth" class="typing fadeTop">People Are Typing...</div>
  <div v-if="typing && isAuth" class="typing fadeBottom">
    People Are Typing...
  </div>
  <p id="end" class="fadeLeft">
    Chat Is Limited To 15 Messages<br />
    Database Location: Asia/Jakarta<br />
    <a v-if="uid === ownerUid" @click="lockChat"><strong><br />Click To <span
    v-if="locked">Un</span>Lock
    Chat</strong><br /></a>
  </p>
  <div id="messages" class="fadeLeft"></div>
  <div id="inputs" v-if="(isAuth && !locked) || uid === ownerUid" class="fadeBottom">
    <button @click="imgCreate">📷</button>
    <input
      autofocus
      id="msg-input"
      type="text"
      placeholder="Hello World, Type Here..."
      @keydown="typingStart"
    />
    <button @click="msgCreate">✈️</button>
  </div>
  <div v-if="showForm && !isAuth" class="fadeBottom" id="form">
    <h1>ChatX Account</h1>
    <form @submit="emailAuth">
      <label for="email">Email:</label>
      <input
        required
        type="email"
        name="email"
        placeholder="user@example.com"
      />
      <label for="pass">Password:</label>
      <input required type="password" name="pass" placeholder="P4$$W0RD!" />
      <button>Sign In / Make Account</button>
    </form>
    <button @click="showForm = false">Cancel</button>
  </div>
</template>

<style>
h2,
p {
  margin-top: 0.1rem;
  margin-bottom: 0.1rem;
}
h1 {
  margin: 0.1em;
}
a {
  text-decoration: none;
}

#form {
  background: #0f0f0fef;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
#form label,
#form input,
#form button {
  display: block;
  margin: 0.1em;
}
#form button,
#form h1 {
  margin-top: 1em;
}

#messages {
  padding-bottom: 4em;
}
#end {
  text-align: center;
  padding-top: 5em;
}

#inputs,
#topbar {
  position: fixed;
  left: 0;
  height: 3em;
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1em;
}
#topbar {
  padding-bottom: 0.3em;
  padding-top: 0.3em;
  top: 1em;
  background: #000000a2;
}
#topbar h1#username {
  overflow-x: scroll;
  overflow-y: hidden;
  white-space: nowrap;
  max-width: 30%;
}
#topbar h1#username::-webkit-scrollbar {
  height: 0.1em;
  background-color: white;
  border-radius: 5em;
}
#topbar h1#username::-webkit-scrollbar-thumb {
  background-color: grey;
}

#inputs {
  bottom: 1em;
}
#inputs button {
  width: 5em;
}
#inputs input {
  width: 60%;
}

.typing {
  position: fixed;
  width: 100%;
  background: #000000a2;
  padding: 0.5em;
}
.fadeTop.typing {
  top: 4.5em;
}
.fadeBottom.typing {
  bottom: 4.5em;
}

#bg {
  background: linear-gradient(to top left, #31112f, #00393b);
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}

input,
button {
  color: white;
  background: #000000a2;
  border: 0.1em solid #ffffff;
  padding: 1em;
  border-top-left-radius: 0.5em;
  border-top-right-radius: 0.5em;
  border-bottom-left-radius: 0.5em;
  border-bottom-right-radius: 0.5em;
}

.fadeLeft {
  animation: fadeLeft 1s ease-out;
}
.fadeTop {
  animation: fadeTop 1s ease-out;
}
.fadeBottom {
  animation: fadeBottom 1s ease-out;
}

@keyframes fadeLeft {
  from {
    opacity: 0;
    transform: translateX(-100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes fadeTop {
  from {
    opacity: 0;
    transform: translateY(-100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes fadeBottom {
  from {
    opacity: 0;
    transform: translateY(100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}
</style>
