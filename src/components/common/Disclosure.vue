<template>
  <div class="tint">
    <div class="window white">
      <div class="content">
        <button class="s-button s-button--additional" @click="$emit('close')">
          <img :src="publicPath + 'close.svg'" alt="Close" />
        </button>

        <div class="fade fade--top"></div>

        <div class="text">
          <h2>Beta Tester User Agreement</h2>

          <p>⚠️</p>

          <p>
            Please note that this is a Beta version of the SnowSwap DEX. The
            platform, its software and all content found on it are provided on
            an “as is” and “as available” basis.
          </p>

          <p>
            While we have made reasonable efforts to ensure the security and
            functionality of the SnowSwap platform including forking much of the
            codebase from existing well-audited projects, nothing approaching
            the rigor of a formal audit has been conducted at this time and the
            front-end is actively being tested.
          </p>

          <p>
            We strongly advise caution to anyone who chooses to use the early
            experimental version. PLEASE DO NOT RISK ANY FUNDS YOU CANNOT AFFORD
            TO LOSE.
          </p>

          <p>
            SnowSwap does not give any warranties, whether express or implied,
            as to the suitability or usability of the platform or any of its
            content. SnowSwap will not be liable for any loss, whether such loss
            is direct, indirect, special or consequential, suffered by any party
            as a result of their use of the SnowSwap.
          </p>

          <p>Thank you for being a Beta Tester.</p>

          <p>(Yes, we care ☃️)</p>
        </div>
        <div class="fade fade--bottom"></div>
      </div>

      <button class="s-button s-button--primary" @click="$emit('close')">
        I Understand
      </button>
    </div>
  </div>
</template>

<script>
export default {
  computed: {
    publicPath() {
      return process.env.BASE_URL;
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
  height: 725px;
  max-height: 85%;
  padding: 0 16px;
  box-shadow: 0 8px 40px rgba(0, 0, 0, 0.09);
}

.content {
  position: relative;
  height: calc(100% - 88px);
  overflow: hidden;
}

.fade {
  position: absolute;
  right: 0;
  left: 0;
  display: flex;
}

.fade.fade--top {
  height: 58px;
  top: 0;
  background: linear-gradient(
    180deg,
    #ffffff 41.24%,
    rgba(255, 255, 255, 0) 93.63%
  );
}

.fade.fade--bottom {
  bottom: 0;
  justify-content: center;
  align-items: flex-end;
  height: 58px;
  background: linear-gradient(
    0deg,
    #ffffff 41.24%,
    rgba(255, 255, 255, 0) 93.63%
  );
}

.text {
  overflow-y: scroll;
  height: 100%;
  padding: 58px 10px 58px;
  font-family: BrandonGrotesque, sans-serif;
  font-style: normal;
}

.text h2 {
  margin: 0 0 16px;
  text-align: center;
  font-style: normal;
  font-weight: bold;
  font-size: 26px;
  line-height: 18px;
  color: #414a5b;
}

.text p {
  font-weight: 400;
  font-size: 18px;
  line-height: 22px;
  color: #414a5b;
}
.text p:last-child {
  margin-bottom: 0;
}

.s-button.s-button--primary {
  margin: 0 auto;
}

.s-button.s-button--additional {
  position: absolute;
  top: 16px;
  right: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 34px;
  height: 34px;
  padding: 0;
  border-radius: 50%;
  z-index: 1;
}
</style>
