<template>
  <div id="gameClient">
    <Header></Header>
    <playersview ref="view" v-on:action="logout"></playersview>
    <Chat ref="chat" :name="name" v-on:submit="sendMessage" />
  </div>
</template>

<script>
/* eslint-disable */
import Chat from "./chat.vue";
import Header from "./header.vue";
import Playersview from "./playersView.vue";
var es;
export default {
  name: "gameClient",
  components: {
    Header,
    Playersview,
    Chat
  },
  data() {
    return {
      name: "player"
    };
  },
  methods: {
    displayMessage(message) {
      this.$refs.chat.addMessage(message);
    },
    sendMessage(message) {
      const name = encodeURIComponent(message.name);
      const text = encodeURIComponent(message.message);
      var result = this.sendActivity(
        `name=${name}&message=${text}&category=message`
      );
      return result;
    },
    receiveEvent(message) {
      if (message.Event == "Enter") {
        name = message.Name;
        message.Message = `${name}さんが入室しました`;
        message.Name = "system";
        this.displayMessage(message);
        this.$refs.view.appendPlayer(name);
      } else if (message.Event == "Quit") {
        name = message.Name;
        message.Message = `${name}さんが退室しました`;
        message.Name = "system";
        this.displayMessage(message);
        this.$refs.view.removePlayer(name);
      }
    },
    sendActivity(requestBody) {
      const method = "POST";
      const body = requestBody;
      const headers = {
        Accept: "application/json",
        "Content-Type": "application/x-www-form-urlencoded; charset=utf-8"
      };
      return fetch("JinrouResponcer.php", { method, headers, body }).then(
        res => {
          return res;
        }
      );
    },
    logout(playerName) {
      if (this.name == playerName) {
        if (window.confirm("ログアウトしますか？")) {
          this.sendActivity("category=quit").then(res => {
              if (res.ok == true) {
                $.ajax({
                  url: "./login.html",
                  type: "GET"
                }).done(data => {
                  $("#gameClient").html(data);
                });
              }
            });
        }
      }
    }
  },
  beforeMount() {
    var vue = this;
    var res = this.sendActivity("category=me");
    res.then(data => {
      if (data.status == "401") {
        $("#gameClient").html("");
        alert("このゲームはログインする必要があります。");
        $.ajax({
          url: "./login.html",
          type: "GET"
        }).done(data => {
          $("#gameClient").html(data);
        });
      } else {
        data.text().then(function(text) {
          vue.name = text;
        });
      }
    });
  },
  created() {
    es = new EventSource("Broadcast.php");
    let vue = this;
    es.onmessage = function(event) {
      let jdata = JSON.parse(event.data);
      if (jdata.Event == "message") {
        vue.displayMessage(jdata);
      } else {
        vue.receiveEvent(jdata);
      }
    };
  }
};
</script>

<style>
#gameClient {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
</style>
