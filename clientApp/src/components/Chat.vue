<script setup>
import * as signalR from "@microsoft/signalr";
import Room from "../components/Room.vue";
import { onMounted, ref } from "vue";
import Splitter from "primevue/splitter";
import SplitterPanel from "primevue/splitterpanel";
import Button from "primevue/button";

const connection = ref(null);
const roomName = ref("");
const roomPassword = ref("");
const joinedRoomName = ref("");
const joinedRoomPassword = ref("");
const roomCreated = ref(false);
const messages = ref([]);
const roomId = ref(0);
const playerCount = ref(0);
const numberOfPlayers = ref(0);
const role = ref("");
const local = "https://localhost:7090/chatHub";
const azureApi = "https://kingdom-api.azurewebsites.net/chatHub";
onMounted(() => {
  connection.value = new signalR.HubConnectionBuilder()
    .withUrl(azureApi)
    .build();

  connection.value
    .start()
    .then(() => {
      console.log("Connection started");
    })
    .catch((err) => {
      console.error("Error while starting connection: " + err);
    });

  connection.value.on("ReceiveMessage", (user, message) => {
    messages.value.push(`${user}: ${message}`);
  });

  connection.value.on("RoomCreated", (room) => {
    console.log(room, "RoomCreated");
    roomCreated.value = true;
    roomName.value = room.name;
    roomId.value = room.roomId;
    numberOfPlayers.value = room.numberOfPlayers;
    playerCount.value = room.roomParticipants;
  });

  connection.value.on("JoinedRoom", (room) => {
    console.log(room, "JoinedRoom");
    roomCreated.value = true;
    roomName.value = room.name;
    numberOfPlayers.value = room.numberOfPlayers;
    roomId.value = room.roomId;
    playerCount.value = room.roomParticipants;
    console.log(roomName.value);
  });

  connection.value.on("ReceiveRole", (assignedRole) => {
    role.value = assignedRole;
    console.log("Assigned role:", assignedRole);
  });

  connection.value.on("Error", (error) => {
    alert(error);
  });
});

const createRoom = () => {
  console.log(roomName.value);
  console.log(roomPassword.value);
  console.log(numberOfPlayers.value);
  connection.value.invoke("CreateRoom", roomName.value, roomPassword.value, 5);
};

const joinRoom = () => {
  connection.value.invoke(
    "JoinRoom",
    joinedRoomName.value,
    joinedRoomPassword.value
  );
};
</script>

<template>
  <div v-if="!roomCreated" class="">
    <Splitter style="height: 100vh" class="mb-8">
      <SplitterPanel
        class="flex items-center justify-center bg-blue-300 rounded-xl"
      >
        <div class="room flex flex-col">
          <h1 class="createTitle">
            Create a Kingdom room session for others to join
          </h1>
          <p>Session room are designed for 5 people</p>
          <input
            class="border-2 rounded-xl text-center py-2"
            placeholder="Enter room name"
            v-model="roomName"
          />
          <input
            class="border-2 rounded-xl text-center py-2"
            type="password"
            placeholder="Enter room password"
            v-model="roomPassword"
          />
          <button
            class="button bg-blue-700 rounded-xl text-center py-2"
            @click="createRoom"
          >
            Create Room
          </button>
        </div>
      </SplitterPanel>
      <SplitterPanel
        class="flex items-center justify-center bg-green-300 rounded-xl"
      >
        <div class="room">
          <h1 class="createTitle">Join a Kingdom room session</h1>
          <input
            class="border-2 rounded-xl text-center py-2"
            type="password"
            placeholder="Enter room name"
            v-model="joinedRoomName"
          />
          <input
            class="border-2 rounded-xl text-center py-2"
            type="password"
            placeholder="Enter room password"
            v-model="joinedRoomPassword"
          />
          <button class="button bg-green-700" @click="joinRoom">
            Join Room
          </button>
        </div>
      </SplitterPanel>
    </Splitter>
  </div>
  <div v-if="roomCreated" class="joinedRoom">
    <Room
      :roomName="roomName"
      :playerCount="playerCount"
      :numberOfPlayers="numberOfPlayers"
      :role="role"
    />
  </div>
</template>
