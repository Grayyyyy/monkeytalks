<template>
  <div id="home-section">
    <!-- NAVBAR -->
    <Navbar class="mb-1"
      :showHomeButton="false"
    ></Navbar>
    <!-- NAVBAR END -->
    <!-- FAUCET END -->
    <transition-expand>
      <FaucetSection v-show="$store.state.showFaucet"/>
    </transition-expand>
    <!-- FAUCET END -->
    <!-- ENTER MESSAGE SECTION -->
    <div class="section pt-4 pt-md-5 pb-4 px-1 my-1" id="enter-message-section">
      <div class="container pt-4 pt-md-5">
        <div class="row align-items-center d-flex justify-content-around">
          <div class="col-12 col-lg-10">
            <form>
              <div class="input-group">
                <EmojiPicker
                  v-show="showEmojiMenu"
                  :visible="showEmojiMenu"
                  :searchText="emojiSearchText"
                  :itemClicked="emojiClicked"
                  v-closable="{handler: 'hideEmojiMenu'}"
                />
                <input
                  type="text"
                  class="font-weight-bold form-control form-control-lg rounded-100 bg-transparent border-2 px-4 px-lg-5 col-12 col-md-9 mx-0 mx-md-2"
                  id="messageInput"
                  @input="onMessageChanged"
                  placeholder="Write a message"
                  ref="messageInputValue"
                  autocomplete="off"
                  v-bind:class="[$store.state.showSendCard ? ['textfield-secondary', 'text-secondary', 'border-secondary'] : ['textfield-primary', 'text-primary', 'border-primary'], 'text-lowercase']"
                >
                <span class="input-group-btn col-12 col-md-3 mt-3 mt-md-0 px-0 mx-0 mx-md-2">
                  <button
                    type="submit"
                    class="btn btn-lg btn-block mx-0"
                    @click="toggleSendCard"
                    :disabled="inputDisabled"
                    v-bind:class="[$store.state.showSendCard ? ['btn-secondary', 'text-light', 'glow-purple', 'grow-3'] 
                    : (messageContent.length == 0 ? ['btn-primary', 'text-dark'] : ['btn-primary', 'text-dark', 'glow-green', 'grow-3']) ]"
                  >{{$store.state.showSendCard ? "Close" : "Send"}}</button>
                </span>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
    <!-- ENTER MESSAGE SECTION END -->
    <!-- SEND CARD SECTION END -->
    <transition-expand>
      <SendCardSection v-show="$store.state.showSendCard" :messageContent="messageContent"/>
    </transition-expand>
    <!-- SEND CARD SECTION END -->
    <!-- CHAT SECTION -->
    <div class="section mb-5 mt-4 mt-md-5 pb-0 pb-md-3" id="chat-section">
      <div class="container mb-5 mt-0 md-mt-5">
        <div class="row align-items-center d-flex justify-content-between">
          <div class="col-12 col-md-11 col-lg-9 mx-auto">
            <transition-group name="list-item" v-if="messages">
              <ChatListItem v-for="message in messages" :message="message" :key="message.id"/>
            </transition-group>
            <div v-else>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
              <ChatListItemDummy/>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- CHAT SECTION END -->

    <!-- FOOTER -->
    <Footer/>
    <!-- FOOTER END -->
  </div>
</template>

<script>
import Vue from "vue"
import Navbar from "./Navbar.vue"
import Footer from "./Footer.vue"
import ChatListItem from "./ChatListItem.vue"
import ChatListItemDummy from "./ChatListItemDummy.vue"
import FaucetSection from "./FaucetSection.vue"
import TransitionExpand from "./TransitionExpand.vue"
import EmojiPicker from "./EmojiPicker.vue"
import SendCardSection from "./SendCardSection.vue"
import Stenography from "../util/stenography.ts"
import Closable from "../directives/closable"
import API from "../util/api.ts"
import Big from "big.js"
import bigInt from "big-integer"

export default Vue.extend({
  name: "Home",
  data() {
    return {
      messages: null,
      loadingMessages: [],
      messageContent: "",
      showEmojiMenu: false,
      emojiIndexStart: -1,
      emojiIndexEnd: -1,
      emojiSearchText: ""
    };
  },
  methods: {
    sendMessage(event) {
      event.preventDefault();
      messages.unshift({
        id: messages[0].id + 1,
        content: this.$refs.messageInputValue.value,
        date: new Date().toString()
      });
      this.$refs.messageInputValue.value = "";
    },
    toggleSendCard(event) {
      event.preventDefault();
      this.$store.state.showSendCard = !this.$store.state.showSendCard;
    },
    emojiClicked(key, event) {
      if (this.showEmojiMenu && key != null) {
        let newPotentialMessage =
          this.messageContent.slice(
            0,
            -1 * (this.emojiIndexEnd - this.emojiIndexStart)
          ) + key;
        let encodedValue = Stenography.encodeMessage(newPotentialMessage);
        if (
          encodedValue == false ||
          Big(Stenography.encodeMessage(newPotentialMessage)).gt(
            Big(10).pow(29)
          )
        ) {
          this.showEmojiMenu = false;
          this.emojiIndexStart = -1;
          this.emojiIndexEnd = -1;
          this.emojiSearchText = "";
        } else {
          this.messageContent = newPotentialMessage;
          this.$refs.messageInputValue.value = this.messageContent;
          this.showEmojiMenu = false;
          this.emojiIndexStart = -1;
          this.emojiIndexEnd = -1;
          this.emojiSearchText = "";
        }
        this.$refs.messageInputValue.focus();
      }
    },
    hideEmojiMenu() {
      this.showEmojiMenu = false;
      this.emojiIndexStart = -1;
      this.emojiIndexEnd = -1;
      this.emojiSearchText = "";
    },
    onMessageChanged(event) {
      this.messageContent = event.target.value;
      if (
        !this.showEmojiMenu &&
        this.$refs.messageInputValue.selectionStart > 0 &&
        this.messageContent.charAt(
          this.$refs.messageInputValue.selectionStart - 1
        ) == ":"
      ) {
        this.showEmojiMenu = true;
        this.emojiIndexStart = this.$refs.messageInputValue.selectionStart - 1;
        this.emojiIndexEnd = this.$refs.messageInputValue.selectionStart;
        this.emojiSearchText = ":";
      } else {
        if (this.showEmojiMenu) {
          if (
            this.messageContent.charAt(
              this.$refs.messageInputValue.selectionStart - 1
            ) == " " ||
            this.messageContent.length == 0
          ) {
            this.hideEmojiMenu();
          } else {
            this.emojiIndexEnd = this.$refs.messageInputValue.selectionStart;
            this.emojiSearchText = this.messageContent.substring(
              this.emojiIndexStart,
              this.emojiIndexEnd
            );
          }
        }
      }
      // Replace characters not in the ascii range 32-96
      this.messageContent = this.messageContent.replace(/[^\x20-\x7A]+/g, "");
      this.messageContent = this.messageContent.toLowerCase();
      Big.DP = 29;
      while (
        Big(Stenography.encodeMessage(this.messageContent)).gt(Big(10).pow(29))
      ) {
        this.messageContent = this.messageContent.slice(0, -1);
      }
      event.target.value = this.messageContent;
    }
  },
  computed: {
    inputDisabled: function() {
      return this.messageContent.length == 0 && !this.$store.state.showSendCard;
    }
  },
  components: {
    Navbar,
    ChatListItem,
    ChatListItemDummy,
    FaucetSection,
    TransitionExpand,
    SendCardSection,
    EmojiPicker,
    Footer,
  },
  sockets: {
    connect: function() {
      //console.log("connected to websocket");
    },
    delete_message: function(data) {
      let message = JSON.parse(data)
      let toDelete = null
      for (let i = 0; i < this.messages.length; i++) {
        if (this.messages[i].id == parseInt(message.id)) {
          toDelete = i
        }
      }
      if (toDelete != null) {
        this.$delete(this.messages, toDelete)
      }
    },
    new_message: function(data) {
      let message = JSON.parse(data);
      if (message.test && this.messages != null) {
        message.id = this.messages[0].id + 1;
      }
      if (this.messages == null) {
        this.loadingMessages.unshift(message);
      } else {
        this.messages.forEach(m => {
          if (m.address == message.address) {
            m.count = message.count;
          }
        });
        this.messages.unshift(message);
        if (this.messages.length > 100) {
          this.$delete(this.messages, this.messages.length - 1)
        }
        // Auto-close
        if (bigInt(Stenography.encodeMessage(this.messageContent))
            .equals(bigInt(message.content.slice(-29)))) {
          this.messageContent = ""
          this.$refs.messageInputValue.value = ""
          this.$store.state.showSendCard = false
        }
      }
    }
  },
  mounted: function() {
    API.getMessages().then(response => {
      if (response != null) {
        this.messages = [];
        response.forEach(element => {
          if (Stenography.decodeMessage(element.content) !== false) {
            this.messages.push(element);
          }
        });
        this.loadingMessages.forEach(element => {
          if (Stenography.decodeMessage(element.content) !== false) {
            this.messages.push(element);
          }
        });
      }
    });
  }
});
</script>

<style lang="scss">
@import "../assets/css/atten_font.css";
@import "../assets/css/main.scss";
</style>

<style>
/* faucet expand transition */
.expand-enter-active,
.expand-leave-active {
  transition-property: opacity, height;
}

.expand-enter,
.expand-leave-to {
  opacity: 0;
}

/* list item animation */
.list-item-enter-active,
.list-item-leave-active {
  transition: opacity 0.3s, transform 0.3s;
  transform-origin: left center;
}
.list-item-enter,
.list-item-leave-to {
  opacity: 0;
  transform: scale(0.5);
}
</style>