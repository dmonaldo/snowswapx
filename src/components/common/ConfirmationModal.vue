<template>
  <div class="tint">
    <div class="window white">
      <div class="content">
        <div class="inner-content">
          <p class="text">
            {{message}}
          </p>

          <button
            class="s-button s-button--primary s-button--submit"
            @click="confirmAction"
          >
            {{buttonText}}
          </button>
        </div>

        <button
          class="s-button s-button--additional s-button--close"
          @click="$emit('close')"
        >
          <img :src="publicPath + 'close.svg'" alt="Close" />
        </button>
      </div>
    </div>

  </div>
</template>

<script>

export default {
  name: "ConfirmationModal",
  props: ["message", "buttonText"],
  computed: {
    publicPath() {
      return process.env.BASE_URL;
    },
  },
  methods: {
    async confirmAction() {
      this.$emit("onConfirmAction");
    },
  },
};
</script>

<style scoped>
* {
  box-sizing: border-box;
}
.tint {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: rgba(0, 0, 0, 0.42);
}
.window.white {
  max-width: 490px;
  height: auto;
  max-height: 85%;
  padding: 0;
  box-shadow: 0 8px 40px rgba(0, 0, 0, 0.09);
}
.content {
  position: relative;
  height: calc(100% - 88px);
  overflow: hidden;
}
.inner-content {
  padding: 50px 26px 26px;
  text-align: center;
}
.text {
  margin: 20px 0 40px 0;
  font-family: BrandonGrotesque, sans-serif;
  font-style: normal;
  font-weight: 400;
  font-size: 16px;
  line-height: 21px;
  letter-spacing: -0.2px;
  color: #414a5b;
}
.s-button.s-button--close {
  position: absolute;
  top: 16px;
  right: 16px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 34px;
  height: 34px;
  padding: 0;
  border-radius: 50%;
  z-index: 1;
}
.s-button.s-button--submit {
  margin: 0 auto;
}
</style>
