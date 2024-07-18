<script>
import axios from "axios";
import ChatBalloon from "./ChatBalloon.vue";
import { getChatArray, disclaimerText } from "../assets/chat.js";
import { ConstantineInfo } from "../assets/chainInfo";
import { SigningCosmWasmClient } from "@cosmjs/cosmwasm-stargate";
import { GasPrice } from "@cosmjs/stargate";

export default {
  name: "ChatBox",
  data() {
    return {
      chatArray: getChatArray(),
      inputText: "",
      typing: false,
      keplrAddress: null, // Store Keplr wallet address
      keplrClient: null, // Store Keplr client
    };
  },
  mounted() {
    this.loadOnMounted();
  },
  components: {
    ChatBalloon: ChatBalloon,
  },
  methods: {
    async connectKeplr() {
      if (!window.keplr) {
        throw new Error("Please install keplr extension");
      }
      try {
        await window.keplr.experimentalSuggestChain(ConstantineInfo);
      } catch (error) {
        alert("Failed to suggest the chain");
      }

      await window.keplr.enable(ConstantineInfo.chainId);
      const offlineSigner = await window.getOfflineSigner(
        ConstantineInfo.chainId
      );
      const accounts = await offlineSigner.getAccounts();
      const gasPrice = GasPrice.fromString(
        "0" + ConstantineInfo.currencies[0].coinMinimalDenom
      );
      const client = await SigningCosmWasmClient.connectWithSigner(
        ConstantineInfo.rpc,
        offlineSigner,
        {
          gasPrice,
        }
      );

      this.keplrAddress = accounts[0].address;
      this.keplrClient = client;
    },
    formatAddress(address) {
      return `${address.slice(0, 11)}...${address.slice(-4)}`;
    },

    async loadOnMounted() {
      this.scrollToBottom(true);
    },
    async send(msg) {
      if (msg === "") return;
      this.inputText = "";
      await this.pushMsg(msg);
      this.scrollToBottom(true);
      await this.typingMsg();
      this.scrollToBottom(true);
      await this.postMsg(msg);
      this.scrollToBottom(true);
    },
    async pushMsg(msg) {
      this.chatArray.push({
        type: "human",
        data: {
          content: msg,
          additional_kwargs: {},
          example: false,
        },
      });
    },
    async typingMsg() {
      this.typing = true;
    },
    async postMsg(msg) {
      let words = `${msg} MEOW.`;
      this.typing = false;
      const index = this.chatArray.push({
        type: "ai",
        data: {
          content: words,
          additional_kwargs: {},
          example: false,
        },
      });
      await this.typeWriter(index, words);
    },
    async typeWriter(index, words) {
      this.chatArray[index - 1].data.content = "";
      for (let i = 0; i < words.length; i++) {
        await new Promise((resolve) => {
          setTimeout(() => {
            this.chatArray[index - 1].data.content += words[i];
            resolve();
          }, 50);
        });
        this.scrollToBottom(false);
      }
    },
    scrollToBottom(smooth) {
      var el = this.$el.querySelector("#chatballoons-container");
      if (el && el.lastElementChild) {
        el.style.scrollBehavior = smooth ? "smooth" : "";
        el.scrollTop = el.lastElementChild.offsetTop;
      }
    },
    async disclaimer() {
      const index = this.chatArray.push({
        type: "ai",
        data: {
          content: disclaimerText,
          additional_kwargs: {},
          example: false,
        },
      });
      await this.typeWriter(index, disclaimerText);
      this.scrollToBottom(true);
    },
  },
};
</script>
<template>
  <div>
    <div id="navbar">
      <span class="text">CW7007 CAT</span>
      <button class="connect-button" @click="connectKeplr">
        {{ keplrAddress ? formatAddress(keplrAddress) : "Connect Wallet" }}
      </button>
    </div>
    <div id="chatballoons-container">
      <ChatBalloon
        v-for="chat in chatArray"
        :key="chat"
        :type="chat.type"
        :msg="chat.data.content"
      />
      <ChatBalloon v-if="typing === true" type="bubble" msg="" />
    </div>
    <div>
      <div id="input-container">
        <div class="padded-input">
          <input
            v-model="inputText"
            type="text"
            placeholder="Input your message to mint CW7007 ..."
            aria-label="Input"
            @keyup.enter="send(inputText)"
          />
        </div>
        <img class="send-icon" src="/send-icon.svg" @click="send(inputText)" />
      </div>
    </div>
    <div id="copyright-container">
      <span
        >created by
        <a
          href="https://sharp-saw-d58.notion.site/D3LAB-10c829858e4c42eda1ce140f3e7e77bf"
          target="_blank"
          >@D3LAB</a
        >
        | <a @click="disclaimer()">Disclaimer</a></span
      >
    </div>
  </div>
</template>

<style scoped>
#navbar {
  height: 50px;
  width: 100%;
  border-bottom: 1px solid #ffffff99;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  position: relative;
}
.text {
  color: #ffffff99;
  letter-spacing: 2px;
  font-weight: 400;
  margin-right: auto;
  margin-left: 30px;
}
.wastebin-icon {
  position: absolute;
  width: 15px;
  right: 20px;
  top: 17px;
  filter: invert(1);
  opacity: 0.7;
}
.wastebin-icon:hover {
  cursor: pointer;
  opacity: 0.9;
}
#chatballoons-container {
  overflow: scroll;
  height: calc(100% - 140px);

  /* Firefox */
  scrollbar-width: none;

  /* IE 10+ */
  -ms-overflow-style: none;
}
#chatballoons-container::-webkit-scrollbar {
  /* Chrome */
  display: none;
}

#input-container {
  height: 45px;
  position: relative;
  padding: 20px 30px 0px 30px;
}
.padded-input {
  padding: 0px 20px 0px 20px;
  border: 1px solid #ffffff99;
  border-radius: 15px;
  color: #ffffff99;
}
.send-icon {
  position: absolute;
  width: 18px;
  right: 45px;
  top: 33px;
  filter: brightness(0) invert(1);
  opacity: 0.7;
}
.send-icon:hover {
  cursor: pointer;
  opacity: 0.9;
}

input {
  width: calc(100% - 25px);
  height: 40px;
  background-color: transparent;
  border: none;
  caret-color: #ffffff99;
  color: #ffffff99;
}

input::placeholder {
  color: #ffffff99;
}

textarea:focus,
input:focus {
  outline: none;
}

#copyright-container {
  height: 25px;
  text-align: center;
  color: #ffffff99;
  font-size: 0.5rem;
}
#copyright-container a:link {
  color: #ffffff99;
  text-decoration: none;
}
#copyright-container a:visited {
  color: #ffffff99;
  text-decoration: none;
}
#copyright-container a:hover {
  color: #ffffff99;
  text-decoration: underline;
  cursor: pointer;
}

button.connect-button {
  border: 1px solid #ffffff99;
  color: white; /* White text */
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 12px;
  cursor: pointer;
  border-radius: 15px; /* Rounded corners */
  background-color: #00000000;
  transition: transform 0.2s; /* Animation */
  position: absolute;
  right: 30px; /* Position the button to the right */
}
button.connect-button:hover {
  transform: scale(1.05); /* Slightly larger on hover */
  cursor: pointer; /* Ensures pointer cursor on hover */
}
.connect-button {
}
</style>
