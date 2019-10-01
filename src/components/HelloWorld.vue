<template>
  <v-container>
    <div id="app">
      <v-btn @click="disconnect" v-if="status === 'connected'" color="error">Disconnect</v-btn>
      <v-btn @click="connect" v-if="status === 'disconnected'" color="primary">Connect</v-btn>

      <v-card class="mx-auto" max-width="800" tile>
        <div v-if="status === 'connected'">
          <v-list-item v-for="(message, i) in messages" :key="i">
            <v-list-item-content>
              <v-list-item-title v-text="message.sended_at + ' : '+ message.body" />
            </v-list-item-content>
          </v-list-item>
        </div>
      </v-card>
    </div>
  </v-container>
</template>

<script>
import { Socket } from "phoenix";
export default {
  name: "HelloWorld",

  methods: {
    connect() {
      let socket = new Socket("ws://localhost:4000/socket");
      socket.connect();
      // Join in the links channel
      this.channel = socket.channel("room:1", {});
      this.channel
        .join()
        .receive("ok", resp => {
          this.status = "connected";
          console.log("Joined successfully", resp);
        })
        .receive("error", resp => {
          console.log("Unable to join", resp);
        });

      this.channel.on("messages", payload => {
        this.messages = payload.messages;
      });
      this.channel.on("new_message", payload => {
        console.log(payload);
        this.messages = this.messages.concat(payload.message);
      });
    },
    disconnect() {
      this.channel.leave();
      this.status = "disconnected";
    }
  },
  data() {
    return {
      message: "",
      messages: [],
      status: "disconnected"
    };
  }
};
</script>
