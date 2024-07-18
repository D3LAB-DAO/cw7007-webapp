<script>
import ChatBox from "./components/ChatBox.vue";

export default {
  data() {
    return {};
  },
  components: {
    ChatBox: ChatBox,
  },
  methods: {},
};
</script>

<template>
  <div class="gradient-bg">
    <svg xmlns="http://www.w3.org/2000/svg">
      <defs>
        <filter id="goo">
          <feGaussianBlur in="SourceGraphic" stdDeviation="10" result="blur" />
          <feColorMatrix
            in="blur"
            mode="matrix"
            values="1 0 0 0 0  0 1 0 0 0  0 0 1 0 0  0 0 0 18 -8"
            result="goo"
          />
          <feBlend in="SourceGraphic" in2="goo" />
        </filter>
      </defs>
    </svg>
    <div class="gradients-container">
      <div class="g1"></div>
      <div class="g2"></div>
      <div class="g3"></div>
      <div class="g4"></div>
      <div class="g5"></div>
      <!-- <div class="interative"></div> -->
    </div>
  </div>
  <div class="container">
    <ChatBox id="chatbox" />
  </div>
</template>

<style scoped>
#chatbox {
  height: calc(100vh);
}
</style>
