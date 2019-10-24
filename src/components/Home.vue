<template>
  <v-container>
    <div id="app">
      <v-form v-model="valid" ref="form">
        <v-text-field v-model="jwt_token" :rules="jwt_token_rules" label="JWT Token" required></v-text-field>
        <v-text-field v-model="room_id" :rules="room_id_rules" label="Room ID" reqired></v-text-field>
        <v-btn @click="disconnect" v-if="status === 'connected'" color="error">Disconnect</v-btn>
        <v-btn @click="connect" v-if="status === 'disconnected'" color="primary">Connect</v-btn>
        <v-btn @click="send_message" v-if="status === 'connected'" class="mx-2" fab dark color="indigo">
            <v-icon dark>mdi-plus</v-icon>
          </v-btn>
        <v-card class="mx-auto" max-width="800">
          <v-textarea
            v-if="status === 'connected'"
            outlined
            v-model="body_message"
            :rules="body_message_rules"
            label="Envie sua mensagem"
            value
          ></v-textarea>
        </v-card>

        <v-card class="mx-auto" max-width="800" tile>
          <div v-if="status === 'connected'">
            <v-list-item v-for="(message, i) in messages" :key="i">
              <v-list-item-content>
                <v-list-item-title
                  v-text="'user: ' + message.sender_user_id + ' says: '+ message.body"
                />
              </v-list-item-content>
            </v-list-item>
          </div>
        </v-card>
      </v-form>
    </div>
  </v-container>
</template>

<script>
import { Socket } from "phoenix";
export default {
  name: "Home",

  methods: {
    connect() {
      if (this.$refs.form.validate()) {
        let socket = new Socket("ws://localhost:4000/socket", {
          params: { token: this.jwt_token }
        });
        socket.connect();
        // Join in the links channel
        this.channel = socket.channel("room:" + this.room_id, {});
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
      }
    },
    disconnect() {
      this.channel.leave();
      this.status = "disconnected";
    },

    send_message() {
      this.channel
        .push("new_message", { body: this.body_message })
        .receive("ok", msg => console.log("created message", msg))
        .receive("error", reasons => console.log("create failed", reasons))
        .receive("timeout", () => console.log("Networking issue..."));

      this.body_message = ""
    }
  },
  data() {
    return {
      message: "",
      messages: [],
      status: "disconnected",
      valid: false,
      jwt_token: "",
      jwt_token_rules: [v => !!v || "JWT is required"],
      room_id: "",
      room_id_rules: [v => !!v || "Room ID is required"],
      body_message: "",
      body_message_rules: [v => !!v || "Not be empty"],
    };
  }
};
</script>
