<template>
  <div class="chat">
    <div class="join-container" v-if="!joined">
      <form @submit.prevent="join">
        <label for="name">What's your name?</label>
        <input type="text" v-model="name" />

        <button type="submit">Send</button>
      </form>
    </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">[{{ message.name }}]: {{ message.text }}</div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label for="message">Message:</label>
          <input type="text" v-model="messageText" @input="emitTyping" />

          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onBeforeMount, ref } from "vue";
import { io } from "socket.io-client";

const socket = io("http://localhost:3030");

const messages = ref([]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  // Get all messages
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });

  // Listen event name "message" by server and push to messages array
  socket.on("message", (response) => {
    messages.value.push(response);
  });

  // Listen event name "typeing" by server and display typing message
  socket.on("typeing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, (response) => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, (response) => {
    messageText.value = "";
  });
};

let timeout = null;
const emitTyping = () => {
  socket.emit("typeing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typeing", { isTyping: false });
  }, 2000);
};
</script>

<style scoped>
.chat {
  padding: 20px;
  height: 60vh;
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.messages-container {
  flex: 1;
}
</style>
