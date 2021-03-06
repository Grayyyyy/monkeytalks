<template>
  <div class="row align-items-center d-flex justify-content-around my-4">
    <div
      class="col-10 py-2 py-md-3 px-4"
      v-bind:class="[message.premium ? ['bg-primary', 'speech-bubble-green'] : ['bg-secondary', 'speech-bubble-purple'] ]"
    >
      <div class="row align-items-center align-middle d-flex justify-content-between py-2">
        <div class="col-12 col-md-6">
          <a
            target="_blank"
            class="py-1"
            v-bind:class="[message.premium ? ['underline-dark'] : ['underline-light'] ]"
            :href="'https://creeper.banano.cc/explorer/block/'+message.block_hash"
          >
            <h3
              class="text-left font-weight-bold break-word inline-text"
              v-bind:class="[message.premium ? ['text-dark'] : ['text-light'] ]"
              v-html="decodeMessage(message.content)"
            ></h3>
          </a>
        </div>
        <h6
          class="col-12 text-left mt-1 col-md-6 text-md-right mt-md-0 font-weight-light"
          v-bind:class="[message.premium ? ['text-dark'] : ['text-light'] ]"
        >{{ formatDate(message.date) }}</h6>
      </div>
      <div
        v-bind:class="[message.premium ? ['bg-black-12', 'border-black-08'] : ['bg-black-15', 'border-black-10'] ]"
        class="row align-items-center d-flex justify-content-center px-3 px-md-4 mx-0 py-1 my-2 rounded-2"
      >
        <a
          target="_blank"
          class="py-2"
          v-bind:class="[message.premium ? ['underline-dark'] : ['underline-light'] ]"
          :href="'https://creeper.banano.cc/explorer/block/'+message.block_hash"
        >
          <h6
            v-bind:class="[message.premium ? ['text-dark'] : ['text-light'] ]"
            class="text-center font-weight-light break-word tk-ibm-plex-mono text-small"
          >{{message.content}}</h6>
        </a>
      </div>
      <div class="row align-items-center d-flex justify-content-center px-3 px-md-4">
        <a
          target="_blank"
          class="py-2"
          v-bind:class="[message.premium ? ['underline-dark'] : ['underline-light'] ]"
          :href="'https://creeper.banano.cc/explorer/account/'+message.address"
        >
          <h6
            v-bind:class="[message.premium ? ['text-dark'] : ['text-light'] ]"
            class="text-center font-weight-light break-word tk-ibm-plex-mono text-small"
          >{{message.address}}</h6>
        </a>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from "vue";
import Stenography from "../util/stenography.ts";
import Util from "../util/util.ts";

export default Vue.extend({
  name: "ChatListItemAdvanced",
  props: {
    message: Object
  },
  methods: {
    decodeMessage(content) {
      let decodedMessage = Stenography.decodeMessage(content);
      let emojiMap = this.$store.state.emojiMap;
      // Process emojis/images
      decodedMessage = Util.escapeHtml(decodedMessage);
      Object.keys(emojiMap).forEach(function(key) {
        decodedMessage = decodedMessage.replace(
          new RegExp(key, "g"),
          `<img src=${emojiMap[key]} class="emoji" />`
        );
      });
      return decodedMessage;
    },
    formatDate(dateStr) {
      return Util.formatDateStr(dateStr);
    }
  }
});
</script>
<style scoped>
.underline-dark:hover {
  text-decoration: underline !important;
  text-decoration-color: #2a1052 !important;
}

.underline-light:hover {
  text-decoration: underline !important;
  text-decoration-color: #d1a4ff !important;
}
.inline-text {
  display: inline !important;
}
</style>
