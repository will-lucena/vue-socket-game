<script setup>
import { computed, ref } from 'vue'

/*global io*/
const socket = io('ws://localhost:3500')
// const socket = io('https://node-chat-260r.onrender.com')
const name = ref('')
const room = ref('')
const message = ref('')
const messageInput = ref(null)
const activity = ref('')
const messages = ref([])
const activeUsers = ref([])
const activeRooms = ref([])

function sendMessage(e) {
  e.preventDefault()

  if (name.value && message.value && room.value) {
    socket.emit('message', {
      text: message.value,
      name: name.value
      // room: chatRoom.value,
    })
    message.value = ''
  }

  messageInput.value.focus()
}

function enterRoom(e) {
  e.preventDefault()
  if (name.value && room.value) {
    socket.emit('enterRoom', {
      name: name.value,
      room: room.value
    })
  }
}

function onKeypress() {
  socket.emit('activity', name.value)
}

socket.on('message', (data) => {
  activity.value = ''
  messages.value.push(data)
  // chatDisplay.scrollTop = chatDisplay.scrollHeight
})

function isMyMessage(chatMessage) {
  return chatMessage.name === name.value
}

function isFromNotAdminUser(chatMessage) {
  return chatMessage.name !== 'admin'
}

let activityTimer

socket.on('activity', (name) => {
  activity.value = `${name} is typing...`
  clearTimeout(activityTimer)
  activityTimer = setTimeout(() => {
    activity.value = ''
  }, 3000)
})

socket.on('userList', ({ users }) => {
  activeUsers.value = users
})

socket.on('roomList', ({ rooms }) => {
  console.log(rooms)
  activeRooms.value = rooms
})

const activeUsersList = computed(() => {
  console.log(activeUsers.value)
  let list = ''
  if (activeUsers.value.length) {
    list = `Users in ${room.value}:`
    activeUsers.value.forEach((user, index) => {
      list += ` ${user.name}`
      if (activeUsers.value.length > 1 && index != activeUsers.value.length - 1) {
        list += ','
      }
    })
  }
  return list
})

const activeRoomsList = computed(() => {
  let rooms = ''
  if (activeRooms.value.length) {
    rooms = 'Active rooms:'
    activeRooms.value.forEach((room, index) => {
      rooms += ` ${room}`
      if (activeRooms.value.length > 1 && index != activeRooms.value.length - 1) {
        rooms += ','
      }
    })
  }
  return rooms
})
</script>

<template>
  <main>
    <form class="form-join">
      <input type="text" v-model="name" maxlength="8" placeholder="Your name" size="5" required />
      <input type="text" v-model="room" maxlength="8" placeholder="Chat room" size="5" required />
      <button @click="enterRoom">Join</button>
    </form>

    <ul class="chat-display">
      <li
        v-for="chatMessage in messages"
        :key="chatMessage.id"
        class="post"
        :class="{
          'post--left': isMyMessage(chatMessage),
          'post--right': !isMyMessage(chatMessage) && isFromNotAdminUser(chatMessage)
        }"
      >
        <div
          v-if="isFromNotAdminUser(chatMessage)"
          class="post__header"
          :class="{
            'post__header--user': isMyMessage(chatMessage),
            'post__header--reply': !isMyMessage(chatMessage)
          }"
        >
          <span class="post__header--name">{{ chatMessage.name }}</span>
          <span class="post__header--time">{{ chatMessage.time }}</span>
        </div>
        <div class="post__text">{{ chatMessage.text }}</div>
      </li>
    </ul>
    <p class="user-list">
      <em>{{ activeUsersList }}</em>
    </p>
    <p class="room-list">
      <em>{{ activeRoomsList }}</em>
    </p>
    <p class="activity">{{ activity }}</p>

    <form class="form-message">
      <input
        type="text"
        ref="messageInput"
        @keypress="onKeypress"
        v-model="message"
        placeholder="your message"
        required
      />
      <button @click="sendMessage">Send</button>
    </form>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
