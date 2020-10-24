<template>
  <div class="tint">
    <div class="window white">
      <div class="content">
        <div class="illustration-wrapper">
          <img
            class="illustration"
            :src="publicPath + statuses[currentStatus].illustration"
            :alt="currentStatus"
          />
        </div>

        <div class="inner-content">
          <p class="text">
            The Snowman has made some SNOW for LPs and SnowSwap users. Enter
            your wallet address to check if you qualify.
          </p>

          <div class="top-row">
            <label class="label" for="input">Recipient</label>
            <a
              class="etherscanLink"
              :href="etherscanUrl()"
              target="_blank"
              v-show="currentStatus !== 'default'"
            >
              View on Etherscan
            </a>
          </div>

          <div class="input-wrapper">
            <input
              :class="{
                input: true,
                'input--success': currentStatus === 'success',
                'input--fail': currentStatus === 'fail',
              }"
              id="input"
              type="text"
              v-model="inputValue"
              @focus="inputFocused = true"
              @blur="inputFocused = false"
            />

            <span class="placeholder" v-show="!inputValue && !inputFocused">
              Wallet Address or ENS name
            </span>
          </div>

          <div class="message-wrapper">
            <p
              class="s-message s-message--error"
              v-if="currentStatus === 'fail'"
            >
              Address has no available claim
            </p>

            <p
              class="s-message s-message--success"
              v-if="currentStatus === 'success'"
            >
              Congratulations! Snow is already on your account
            </p>
          </div>

          <button
            class="s-button s-button--primary s-button--submit"
            @click="letItSnow"
            :disabled="!inputValueIsValid"
          >
            Let it snow
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

    <div class="snow-wrapper" v-show="currentStatus === 'success'">
      <div class="snow layer1 a"></div>
      <div class="snow layer1"></div>
      <div class="snow layer2 a"></div>
      <div class="snow layer2"></div>
      <div class="snow layer3 a"></div>
      <div class="snow layer3"></div>
    </div>
  </div>
</template>

<script>
import { distributor_abi, distributor_address } from "../../allabis";
import { contract as contractstate } from "../../contract";
import { airdrop } from "../../jsons/airdrop2.js";
export default {
  name: "ClaimSnow",
  data: () => {
    return {
      statuses: {
        default: {
          illustration: "claim-claim.svg",
        },
        fail: {
          illustration: "claim-fail.svg",
        },
        success: {
          illustration: "claim-success.svg",
        },
      },
      currentStatus: "default",
      inputValue: "",
      ethAddr: "",
      inputValueIsValid: false,
      inputFocused: false,
    };
  },
  computed: {
    publicPath() {
      return process.env.BASE_URL;
    },
  },
  methods: {
    async letItSnow() {
      // request logic here
      const contract = new contractstate.web3.eth.Contract(
        distributor_abi,
        distributor_address
      );
      console.log("let is snow");
      // console.log(airdrop);
      console.log(this.ethAddr);
      let tempStatus = false;

      const response = await fetch(
        `https://distributor-dot-snowball-infrastructure.appspot.com/get-payload?w=${contractstate.web3.utils.toChecksumAddress(
          this.ethAddr
        )}`
      );
      if (response.status !== 200) {
        // this.currentStatus=  "fail";
      } else {
        //logic for the first distribution
        const firstContract = new contractstate.web3.eth.Contract(
          distributor_abi,
          "0x5a2c04a3976a2a85efd5769b043e838c2448cc8a"
        );
        const claim = await response.json();
        console.log(`claim1:`, claim);
        const isClaimed = await firstContract.methods
          .isClaimed(globalThis.web3.utils.numberToHex(claim.index))
          .call();
        if (!isClaimed) {
          const called = await firstContract.methods
            .claim(
              globalThis.web3.utils.numberToHex(claim.index),
              this.ethAddr,
              claim.amount,
              claim.proof
            )
            .send({ from: contractstate.default_account });
          tempStatus = tempStatus || true;
          // this.currentStatus = "success";
        } else {
          tempStatus = tempStatus || false;
          console.log(`claim1:claimed`);
          // this.currentStatus = "fail";
        }
      }
      const claims = airdrop.claims;
      let claim2 = null;
      for (let key in claims) {
        if (key == this.ethAddr) {
          claim2 = claims[key];
        }
      }
      if (claim2) {
        // const claim = await response.json();
        const isClaimed = await contract.methods
          .isClaimed(globalThis.web3.utils.numberToHex(claim2.index))
          .call();
        if (!isClaimed) {
          console.log(`claim2:`, claim2);
          const called = await contract.methods
            .claim(
              globalThis.web3.utils.numberToHex(claim2.index),
              this.ethAddr,
              claim2.amount,
              claim2.proof
            )
            .send({ from: contractstate.default_account });
          tempStatus = tempStatus || true;
          // this.currentStatus = "success";
        } else {
          tempStatus = tempStatus || false;
          console.log(`claim2:claimed`);
          // this.currentStatus = "fail";
        }
      } else {
        console.log(`claim2:claimed`);
        tempStatus = tempStatus || false;
      }
      this.currentStatus = tempStatus ? "success" : "fail";
    },
    etherscanUrl() {
      if (this.ethAddr && this.ethAddr.length === 42) {
        return `https://etherscan.io/address/${this.ethAddr}`;
      } else {
        return `https://etherscan.io/enslookup-search?search=${this.ethAddr}`;
      }
    },
  },
  watch: {
    inputValue: function (newValue) {
      // input validation logic here
      if (window.web3.utils.isAddress(newValue)) {
        newValue = newValue.slice(0, 42);
        this.inputValueIsValid = true;
        this.ethAddr = newValue;
      } else {
        const getENS = async (v, _this) => {
          const ens = await web3.eth.ens.getResolver(newValue);

          if (ens._address === "0x0000000000000000000000000000000000000000") {
            _this.inputValueIsValid = false;
            _this.ethAddr = ens.address;
          } else {
            _this.inputValueIsValid = true;
            _this.ethAddr = ens._address;
          }
        };
        getENS(newValue, this);
        this.inputValueIsValid = false;
        this.ethAddr = "";
      }

      this.inputValue = newValue;
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
.illustration-wrapper {
  position: relative;
  width: 100%;
  padding-top: 51%;
}
.illustration {
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
}
.illustration {
  width: 100%;
}
.inner-content {
  padding: 32px 26px 26px;
}
.text {
  margin: 0;
  font-family: BrandonGrotesque, sans-serif;
  font-style: normal;
  font-weight: 400;
  font-size: 16px;
  line-height: 21px;
  letter-spacing: -0.2px;
  color: #414a5b;
}
.top-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 26px;
}
.label {
  font-family: BrandonGrotesque, sans-serif;
  font-style: normal;
  font-weight: bold;
  font-size: 16px;
  line-height: 18px;
  color: #838b9d;
}
.etherscanLink {
  font-family: BrandonGrotesque, sans-serif;
  font-weight: bold;
  font-size: 16px;
  line-height: 18px;
  text-transform: uppercase;
  color: #00b9c2;
}
.input-wrapper {
  position: relative;
  margin-top: 12px;
}
.input {
  height: 50px;
  text-align: center;
  font-family: RobotoMono, monospace;
  font-weight: 500;
  font-size: 16px;
  color: #414a5b;
}
.input.input--fail {
  background-color: #feedf0;
}
.input.input--success {
  background-color: #d9f9f8;
}
.placeholder {
  position: absolute;
  top: 16px;
  left: 16px;
  font-family: BrandonGrotesque, sans-serif;
  font-style: normal;
  font-weight: 500;
  font-size: 16px;
  line-height: 18px;
  color: rgba(65, 74, 91, 0.5);
  pointer-events: none;
}
.message-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 48px;
  padding: 4px 0;
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

/* SNOW ANIMATION CONVERTED FROM SCSS*/
.snow-wrapper {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
}
.snow {
  border-radius: 50%;
  opacity: 0.8;
  position: absolute;
  top: -100vh;
  animation-name: fall;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
}
.layer1 {
  width: 1.5rem;
  height: 1.5rem;
  filter: blur(1.5px);
  box-shadow: 10.3vw 28.6vh 0 -0.21rem#fff, 53.1vw 37.9vh 0 -0.5rem#fff,
    40.4vw 7.5vh 0 -0.47rem#fff, 92.7vw 20.3vh 0 -0.48rem#fff,
    82.7vw 62.9vh 0 -0.2rem#fff, 6.7vw 88vh 0 -0.13rem#fff,
    86.6vw 75.2vh 0 -0.15rem#fff, 74.6vw 65.9vh 0 -0.5rem#fff,
    69.5vw 23.6vh 0 -0.03rem#fff, 41.9vw 41.8vh 0 -0.47rem#fff,
    75.7vw 90.2vh 0 -0.24rem#fff, 32.9vw 72.3vh 0 -0.5rem#fff,
    65.2vw 77vh 0 -0.09rem#fff, 72.6vw 9.3vh 0 -0.41rem#fff,
    8.4vw 90.2vh 0 -0.06rem#fff, 97.8vw 56.4vh 0 -0.3rem#fff,
    97.7vw 24.8vh 0 -0.39rem#fff, 69.5vw 9.5vh 0 -0.14rem#fff,
    41.6vw 78.5vh 0 -0.3rem#fff, 94.4vw 33.3vh 0 -0.27rem#fff,
    13.2vw 55.8vh 0 -0.25rem#fff, 74vw 12.7vh 0 -0.42rem#fff,
    64.5vw 90.4vh 0 -0.06rem#fff, 46.9vw 95.6vh 0 -0.44rem#fff,
    72.7vw 8vh 0 -0.29rem#fff, 11.6vw 67.8vh 0 -0.44rem#fff,
    22.5vw 23.6vh 0 -0.05rem#fff, 26.2vw 13.3vh 0 -0.48rem#fff,
    75.2vw 16.3vh 0 -0.5rem#fff, 18vw 95.7vh 0 -0.47rem#fff,
    87.4vw 63.9vh 0 -0.14rem#fff, 48.4vw 83.6vh 0 -0.24rem#fff,
    56vw 46.8vh 0 -0.41rem#fff, 41vw 15.3vh 0 -0.23rem#fff,
    37.7vw 21.4vh 0 -0.41rem#fff, 38.2vw 88.1vh 0 -0.3rem#fff,
    82.6vw 44.4vh 0 -0.42rem#fff, 48.3vw 2.3vh 0 -0.49rem#fff,
    74vw 9vh 0 -0.41rem#fff, 16.2vw 94.3vh 0 -0.34rem#fff,
    10.7vw 59.2vh 0 -0.32rem#fff, 76.2vw 21.9vh 0 -0.12rem#fff,
    40.5vw 68.3vh 0 -0.48rem#fff, 67.6vw 95.7vh 0 -0.34rem#fff,
    65.5vw 13.1vh 0 -0.43rem#fff, 79.4vw 60.8vh 0 -0.42rem#fff,
    39.5vw 42.8vh 0 -0.23rem#fff, 61vw 29.3vh 0 -0.49rem#fff,
    96.4vw 88.8vh 0 -0.31rem#fff, 23vw 62.3vh 0 -0.14rem#fff,
    26.1vw 92.7vh 0 -0.5rem#fff, 83.3vw 61vh 0 -0.35rem#fff,
    29.4vw 62.7vh 0 -0.06rem#fff, 70.2vw 92.4vh 0 -0.06rem#fff,
    55.4vw 76.9vh 0 -0.42rem#fff, 14.9vw 50.1vh 0 -0.33rem#fff,
    30.5vw 44vh 0 -0.16rem#fff, 11.7vw 74.1vh 0 -0.48rem#fff,
    4.6vw 48.9vh 0 -0.36rem#fff, 53.9vw 82.6vh 0 -0.12rem#fff,
    37.4vw 44.7vh 0 -0.31rem#fff, 81.6vw 74.9vh 0 -0.46rem#fff,
    86.3vw 68.3vh 0 -0.37rem#fff, 89vw 89.1vh 0 -0.04rem#fff,
    33.6vw 76.3vh 0 -0.21rem#fff, 2.5vw 10.3vh 0 -0.04rem#fff,
    12.1vw 49vh 0 -0.27rem#fff, 24.2vw 76.9vh 0 -0.47rem#fff,
    88.1vw 78.9vh 0 -0.3rem#fff, 96.6vw 13.9vh 0 -0.18rem#fff,
    51.6vw 64.6vh 0 -0.21rem#fff, 52.3vw 63.1vh 0 -0.04rem#fff,
    70.7vw 1.4vh 0 -0.14rem#fff, 2.8vw 26.6vh 0 -0.47rem#fff,
    75.4vw 24.2vh 0 -0.47rem#fff, 66.7vw 17.6vh 0 -0.28rem#fff,
    89.4vw 40.1vh 0 -0.07rem#fff, 53.2vw 68.7vh 0 -0.06rem#fff,
    29.9vw 41.7vh 0 -0.4rem#fff, 69vw 20.4vh 0 -0.44rem#fff,
    81.9vw 29.4vh 0 -0.16rem#fff, 89.5vw 79.8vh 0 -0.32rem#fff,
    51.3vw 54.2vh 0 -0.17rem#fff, 29.6vw 2.8vh 0 -0.02rem#fff,
    7.1vw 60vh 0 -0.34rem#fff, 48vw 9.9vh 0 -0.3rem#fff,
    53.2vw 13vh 0 -0.16rem#fff, 50.7vw 61.6vh 0 -0.18rem#fff,
    31vw 2vh 0 -0.17rem#fff, 12.3vw 66.2vh 0 -0.25rem#fff,
    25.2vw 92.8vh 0 -0.26rem#fff, 38.3vw 20.4vh 0 -0.04rem#fff,
    0.8vw 82.8vh 0 -0.1rem#fff, 66.9vw 10.5vh 0 -0.34rem#fff,
    45.7vw 83.1vh 0 -0.42rem#fff, 22.1vw 89.2vh 0 -0.12rem#fff,
    25.6vw 94.4vh 0 -0.35rem#fff, 24.7vw 31.5vh 0 -0.14rem#fff,
    70.7vw 40.2vh 0 -0.07rem#fff, 34.7vw 4.4vh 0 -0.5rem#fff,
    18.8vw 35.8vh 0 -0.38rem#fff, 30.2vw 64.8vh 0 -0.3rem#fff,
    41.4vw 44.8vh 0 -0.31rem#fff, 96.8vw 0.8vh 0 -0.47rem#fff,
    42.3vw 36.8vh 0 -0.44rem#fff, 82.7vw 71.8vh 0 -0.22rem#fff,
    81vw 82.6vh 0 -0.31rem#fff, 54.1vw 62.7vh 0 -0.44rem#fff,
    86.1vw 25.8vh 0 -0.48rem#fff, 96.6vw 88.6vh 0 -0.43rem#fff,
    89.1vw 7.6vh 0 -0.28rem#fff, 86.9vw 29.1vh 0 -0.29rem#fff,
    15vw 91.2vh 0 -0.22rem#fff, 24.8vw 57.9vh 0 -0.39rem#fff,
    47.7vw 8.6vh 0 -0.33rem#fff, 22.9vw 97.8vh 0 -0.3rem#fff,
    83.3vw 25.7vh 0 -0.46rem#fff, 76.6vw 86.5vh 0 -0.39rem#fff,
    31.8vw 84.1vh 0 -0.24rem#fff, 80.6vw 58.6vh 0 -0.47rem#fff,
    16.8vw 75.2vh 0 -0.34rem#fff, 63.7vw 66.8vh 0 -0.2rem#fff,
    71vw 24.9vh 0 -0.08rem#fff, 38.2vw 42.3vh 0 -0.11rem#fff,
    20.6vw 36.1vh 0 -0.12rem#fff, 68.1vw 18.7vh 0 -0.11rem#fff,
    72.4vw 34.3vh 0 -0.19rem#fff, 87.7vw 35.2vh 0 -0.08rem#fff,
    98.3vw 31.5vh 0 -0.28rem#fff, 45.7vw 37.4vh 0 -0.05rem#fff,
    69.5vw 75vh 0 -0.4rem#fff, 18.5vw 37.2vh 0 -0.23rem#fff,
    66vw 82.4vh 0 -0.19rem#fff, 15.3vw 60.5vh 0 -0.09rem#fff,
    60.9vw 39.9vh 0 -0.33rem#fff, 81.3vw 87.8vh 0 -0.37rem#fff,
    31.4vw 18.1vh 0 -0.33rem#fff, 55vw 73.6vh 0 -0.2rem#fff,
    61.1vw 98.2vh 0 -0.18rem#fff, 28.5vw 31.3vh 0 -0.16rem#fff,
    70.9vw 85.7vh 0 -0.5rem#fff, 68.4vw 38.4vh 0 -0.04rem#fff,
    74.5vw 42.8vh 0 -0.41rem#fff, 85.4vw 62.3vh 0 -0.14rem#fff,
    81.3vw 34.4vh 0 -0.03rem#fff, 1.6vw 92.6vh 0 -0.3rem#fff,
    16vw 74.2vh 0 -0.44rem#fff, 58.8vw 27.8vh 0 -0.02rem#fff,
    54.4vw 32.6vh 0 -0.03rem#fff, 62.5vw 26.6vh 0 -0.05rem#fff,
    73.4vw 1.3vh 0 -0.19rem#fff, 68.4vw 42.8vh 0 -0.17rem#fff,
    52.8vw 15.3vh 0 -0.08rem#fff, 60.8vw 8.3vh 0 -0.07rem#fff,
    43.9vw 6.9vh 0 -0.49rem#fff, 34.7vw 19.7vh 0 -0.17rem#fff,
    55vw 23vh 0 -0.07rem#fff, 52vw 95.2vh 0 -0.02rem#fff,
    32.3vw 7.5vh 0 -0.41rem#fff, 9.8vw 95vh 0 -0.4rem#fff,
    62.6vw 39.6vh 0 -0.19rem#fff, 57.8vw 8.9vh 0 -0.24rem#fff,
    2.9vw 63.1vh 0 -0.37rem#fff, 48.8vw 98.8vh 0 -0.37rem#fff,
    60.6vw 55.7vh 0 -0.36rem#fff, 25vw 44vh 0 -0.31rem#fff,
    71vw 76.4vh 0 -0.46rem#fff, 18.8vw 30.1vh 0 -0.25rem#fff,
    7.7vw 1.4vh 0 -0.18rem#fff, 89.5vw 86.3vh 0 -0.41rem#fff,
    16.4vw 91.3vh 0 -0.1rem#fff, 30.5vw 73.9vh 0 -0.03rem#fff,
    22.9vw 58vh 0 -0.03rem#fff, 41vw 93.5vh 0 -0.25rem#fff,
    2.4vw 11.4vh 0 -0.33rem#fff, 4.7vw 62.9vh 0 -0.19rem#fff,
    30.1vw 76.9vh 0 -0.22rem#fff, 59.2vw 28.8vh 0 -0.27rem#fff,
    86.9vw 16.2vh 0 -0.04rem#fff, 36.4vw 94.8vh 0 -0.19rem#fff,
    91.2vw 40.1vh 0 -0.46rem#fff, 52.6vw 71.3vh 0 -0.1rem#fff,
    94.4vw 96.7vh 0 -0.18rem#fff, 65.8vw 3.7vh 0 -0.25rem#fff,
    48.3vw 68.7vh 0 -0.4rem#fff, 26.1vw 79.6vh 0 -0.25rem#fff,
    72.9vw 33.5vh 0 -0.04rem#fff, 85.5vw 40.9vh 0 -0.24rem#fff,
    79vw 2.5vh 0 -0.43rem#fff, 62.9vw 80.4vh 0 -0.11rem#fff,
    77.6vw 76.2vh 0 -0.3rem#fff, 28.1vw 31.2vh 0 -0.25rem#fff,
    13.5vw 13.4vh 0 -0.5rem#fff, 86.9vw 95vh 0 -0.09rem#fff,
    99.6vw 46.5vh 0 -0.16rem#fff, 53vw 70.5vh 0 -0.35rem#fff,
    47.1vw 27.7vh 0 -0.37rem#fff, 71.2vw 56.3vh 0 -0.35rem#fff,
    25.8vw 25.4vh 0 -0.04rem#fff, 69.9vw 65.7vh 0 -0.1rem#fff,
    96.4vw 92.6vh 0 -0.24rem#fff, 15.3vw 61.4vh 0 -0.19rem#fff,
    29.2vw 10vh 0 -0.44rem#fff, 67.9vw 71.8vh 0 -0.09rem#fff,
    92.1vw 67.3vh 0 -0.38rem#fff, 79.2vw 23.7vh 0 -0.18rem#fff,
    95.8vw 51.5vh 0 -0.45rem#fff, 58.5vw 52vh 0 -0.41rem#fff,
    24.1vw 8.7vh 0 -0.16rem#fff, 58.9vw 48.5vh 0 -0.03rem#fff,
    10.9vw 65.9vh 0 -0.33rem#fff, 62vw 20.7vh 0 -0.01rem#fff,
    68.5vw 53.6vh 0 -0.12rem#fff, 24.7vw 21.3vh 0 -0.17rem#fff,
    54vw 15.2vh 0 -0.46rem#fff, 56.9vw 73.4vh 0 -0.15rem#fff,
    11.2vw 41.9vh 0 -0.12rem#fff, 51.3vw 35.2vh 0 -0.48rem#fff,
    33.4vw 64.3vh 0 -0.06rem#fff, 76.6vw 52.1vh 0 -0.36rem#fff,
    6.3vw 71vh 0 -0.47rem#fff, 19.8vw 54.5vh 0 -0.01rem#fff,
    83.8vw 60.2vh 0 -0.43rem#fff, 50.8vw 76vh 0 -0.21rem#fff,
    11.5vw 24.1vh 0 -0.27rem#fff, 92.4vw 2.9vh 0 -0.07rem#fff,
    37.1vw 13.2vh 0 -0.2rem#fff, 29.8vw 77.3vh 0 -0.03rem#fff,
    96.4vw 71.2vh 0 -0.1rem#fff, 13.7vw 48.9vh 0 -0.33rem#fff,
    77vw 10vh 0 -0.28rem#fff, 55.7vw 11.5vh 0 -0.21rem#fff,
    58vw 18.8vh 0 -0.16rem#fff, 11.4vw 77vh 0 -0.38rem#fff,
    54.5vw 54.3vh 0 -0.49rem#fff, 71.7vw 65.6vh 0 -0.23rem#fff,
    69.7vw 72.4vh 0 -0.28rem#fff, 20vw 43.4vh 0 -0.28rem#fff,
    51.7vw 87.7vh 0 -0.19rem#fff, 50.3vw 66.4vh 0 -0.18rem#fff,
    21.6vw 12.3vh 0 -0.48rem#fff, 42.4vw 73.6vh 0 -0.2rem#fff,
    13.7vw 35vh 0 -0.46rem#fff, 88.9vw 95.1vh 0 -0.41rem#fff,
    98.8vw 46.7vh 0 -0.27rem#fff, 72.7vw 18.5vh 0 -0.47rem#fff,
    24.9vw 73.6vh 0 -0.01rem#fff, 87.2vw 63.4vh 0 -0.1rem#fff,
    43vw 72.2vh 0 -0.28rem#fff, 9.7vw 31.5vh 0 -0.1rem#fff,
    98.6vw 91.4vh 0 -0.03rem#fff, 32.5vw 66.7vh 0 -0.38rem#fff,
    57.7vw 70.9vh 0 -0.44rem#fff, 93.6vw 85.5vh 0 -0.2rem#fff,
    30.5vw 45.2vh 0 -0.03rem#fff, 24.6vw 88.4vh 0 -0.1rem#fff,
    79.6vw 5.4vh 0 -0.27rem#fff, 88.4vw 53.9vh 0 -0.15rem#fff,
    75vw 64.3vh 0 -0.34rem#fff, 65.2vw 15.8vh 0 -0.19rem#fff,
    65.8vw 7.3vh 0 -0.49rem#fff, 22.8vw 75.9vh 0 -0.3rem#fff,
    56.5vw 93vh 0 -0.03rem#fff, 38.1vw 25.1vh 0 -0.24rem#fff,
    20.4vw 76.9vh 0 -0.27rem#fff, 87.7vw 41.5vh 0 -0.02rem#fff,
    59.6vw 22vh 0 -0.08rem#fff, 19.2vw 51vh 0 -0.32rem#fff,
    8.3vw 99.8vh 0 -0.37rem#fff, 20.8vw 71.3vh 0 -0.05rem#fff,
    35vw 69.6vh 0 -0.48rem#fff, 23.3vw 71.9vh 0 -0.02rem#fff,
    50.5vw 1.6vh 0 -0.05rem#fff, 99.5vw 34.1vh 0 -0.23rem#fff,
    53.7vw 80.7vh 0 -0.32rem#fff, 46.2vw 76.8vh 0 -0.48rem#fff,
    47.5vw 59.7vh 0 -0.19rem#fff, 18vw 21.2vh 0 -0.38rem#fff,
    44.6vw 69.2vh 0 -0.2rem#fff, 59.6vw 38.3vh 0 -0.16rem#fff,
    83.5vw 62.6vh 0 -0.41rem#fff, 38.1vw 49.4vh 0 -0.32rem#fff,
    35.9vw 17.8vh 0 -0.01rem#fff, 74.4vw 47vh 0 -0.1rem#fff,
    67.3vw 31.5vh 0 -0.1rem#fff, 39.3vw 60vh 0 -0.06rem#fff,
    81.2vw 62.4vh 0 -0.08rem#fff, 53.8vw 71.6vh 0 -0.32rem#fff,
    85.9vw 49.2vh 0 -0.44rem#fff, 66.9vw 26.9vh 0 -0.16rem#fff,
    37.9vw 34.7vh 0 -0.26rem#fff, 28.6vw 70.4vh 0 -0.28rem#fff,
    18.2vw 42.9vh 0 -0.19rem#fff, 80.7vw 49.3vh 0 -0.45rem#fff,
    58.9vw 36.1vh 0 -0.25rem#fff, 7vw 10.3vh 0 -0.03rem#fff,
    89.5vw 42vh 0 -0.35rem#fff, 44.4vw 29.1vh 0 -0.16rem#fff,
    38vw 5.9vh 0 -0.48rem#fff, 43.5vw 93.3vh 0 -0.32rem#fff,
    98.4vw 28.6vh 0 -0.08rem#fff, 64.9vw 57.3vh 0 -0.07rem#fff,
    34.5vw 47.8vh 0 -0.49rem#fff, 76.5vw 6.9vh 0 -0.09rem#fff,
    25.5vw 96.1vh 0 -0.06rem#fff, 75.6vw 36.8vh 0 -0.07rem#fff,
    45.1vw 23.6vh 0 -0.03rem#fff, 2.6vw 4.1vh 0 -0.25rem#fff,
    40.4vw 23.8vh 0 -0.4rem#fff, 33.9vw 31.6vh 0 -0.03rem#fff,
    22.7vw 73.9vh 0 -0.19rem#fff, 40.1vw 8vh 0 -0.37rem#fff,
    88.5vw 97.5vh 0 -0.39rem#fff, 48.3vw 37.9vh 0 -0.04rem#fff,
    50.2vw 2.2vh 0 -0.36rem#fff, 41vw 45.5vh 0 -0.42rem#fff,
    34.1vw 12.9vh 0 -0.08rem#fff, 67.2vw 55.9vh 0 -0.12rem#fff,
    12.5vw 88.6vh 0 -0.01rem#fff, 95vw 68.4vh 0 -0.12rem#fff,
    74.4vw 52.2vh 0 -0.47rem#fff, 35.4vw 21.2vh 0 -0.37rem#fff,
    35.4vw 18.6vh 0 -0.03rem#fff, 99.9vw 17.2vh 0 -0.12rem#fff,
    19.4vw 31.4vh 0 -0.3rem#fff, 85.6vw 42.9vh 0 -0.3rem#fff,
    87vw 87.5vh 0 -0.01rem#fff, 84.4vw 24.5vh 0 -0.28rem#fff,
    91.3vw 51vh 0 -0.45rem#fff, 77.5vw 28.6vh 0 -0.32rem#fff,
    5.1vw 35.4vh 0 -0.12rem#fff, 18.3vw 5.9vh 0 -0.44rem#fff,
    6.1vw 77.6vh 0 -0.29rem#fff, 51.5vw 38.3vh 0 -0.35rem#fff,
    97.8vw 60.9vh 0 -0.13rem#fff, 2.6vw 44.4vh 0 -0.06rem#fff,
    84.6vw 9.3vh 0 -0.16rem#fff, 70.3vw 72.7vh 0 -0.33rem#fff,
    30.6vw 47.9vh 0 -0.36rem#fff, 83.4vw 44.7vh 0 -0.03rem#fff,
    74vw 52.4vh 0 -0.14rem#fff, 1.6vw 88.7vh 0 -0.21rem#fff,
    21vw 49.5vh 0 -0.14rem#fff, 7vw 0.3vh 0 -0.09rem#fff,
    47.3vw 99.8vh 0 -0.29rem#fff, 61.6vw 69.8vh 0 -0.11rem#fff,
    47.2vw 24.7vh 0 -0.4rem#fff, 23.7vw 38.1vh 0 -0.09rem#fff,
    56.6vw 18.6vh 0 -0.42rem#fff, 10.2vw 40.7vh 0 -0.43rem#fff,
    95.8vw 44.9vh 0 -0.39rem#fff, 11.1vw 39.2vh 0 -0.21rem#fff,
    38.8vw 59.8vh 0 -0.32rem#fff, 8.5vw 82vh 0 -0.47rem#fff,
    75.3vw 62.6vh 0 -0.29rem#fff, 73.3vw 58vh 0 -0.47rem#fff,
    42.5vw 22.2vh 0 -0.35rem#fff, 59.1vw 97.1vh 0 -0.15rem#fff,
    43.5vw 46.8vh 0 -0.25rem#fff, 20.3vw 79.2vh 0 -0.04rem#fff,
    37.2vw 54.9vh 0 -0.07rem#fff, 65.7vw 82.5vh 0 -0.44rem#fff,
    76.1vw 79.4vh 0 -0.48rem#fff, 23.5vw 72.4vh 0 -0.4rem#fff,
    72.1vw 83.5vh 0 -0.2rem#fff, 90vw 52.4vh 0 -0.32rem#fff,
    36.3vw 11.5vh 0 -0.41rem#fff, 44.2vw 92vh 0 -0.35rem#fff,
    99.4vw 53.6vh 0 -0.08rem#fff, 55.9vw 55.8vh 0 -0.35rem#fff,
    33.3vw 8vh 0 -0.18rem#fff, 84.6vw 32vh 0 -0.01rem#fff,
    84.5vw 52.5vh 0 -0.35rem#fff, 37.3vw 41.7vh 0 -0.02rem#fff,
    96.6vw 15.6vh 0 -0.42rem#fff, 88.5vw 60.5vh 0 -0.11rem#fff,
    67vw 81.9vh 0 -0.48rem#fff, 39vw 42.2vh 0 -0.31rem#fff,
    4.4vw 81.2vh 0 -0.42rem#fff, 98.1vw 9.8vh 0 -0.37rem#fff,
    18vw 43.7vh 0 -0.32rem#fff, 33.6vw 36.7vh 0 -0.02rem#fff,
    67.9vw 61vh 0 -0.11rem#fff, 17vw 18.3vh 0 -0.35rem#fff,
    82.2vw 18.1vh 0 -0.19rem#fff, 10.2vw 25.2vh 0 -0.34rem#fff,
    51.9vw 24.3vh 0 -0.07rem#fff, 92.8vw 37.8vh 0 -0.33rem#fff,
    57.6vw 8.8vh 0 -0.22rem#fff, 91.4vw 33.4vh 0 -0.11rem#fff,
    64.9vw 4.3vh 0 -0.4rem#fff, 19.3vw 0.6vh 0 -0.03rem#fff,
    53.1vw 27.3vh 0 -0.18rem#fff, 18.4vw 96.1vh 0 -0.07rem#fff,
    96.4vw 26.9vh 0 -0.12rem#fff, 40.1vw 68.8vh 0 -0.12rem#fff,
    44.1vw 97.7vh 0 -0.26rem#fff, 64.9vw 6.5vh 0 -0.2rem#fff,
    69vw 46.5vh 0 -0.32rem#fff, 17.7vw 5.3vh 0 -0.3rem#fff;
  animation-duration: 6s;
}
.layer1.a {
  animation-delay: -3s;
}
.layer2 {
  width: 1.2rem;
  height: 1.2rem;
  filter: blur(3px);
  box-shadow: 41.4vw 82.5vh 0 -0.38rem#fff, 70.6vw 15.8vh 0 -0.06rem#fff,
    42.6vw 73.5vh 0 -0.48rem#fff, 1.8vw 63.8vh 0 -0.49rem#fff,
    98.3vw 0.7vh 0 -0.29rem#fff, 92.6vw 0.8vh 0 -0.43rem#fff,
    80.6vw 31.6vh 0 -0.36rem#fff, 61.4vw 51.8vh 0 -0.12rem#fff,
    94.9vw 99.6vh 0 -0.17rem#fff, 8.3vw 17.7vh 0 -0.18rem#fff,
    76.9vw 23.3vh 0 -0.13rem#fff, 13.3vw 35.9vh 0 -0.28rem#fff,
    95.7vw 92.9vh 0 -0.12rem#fff, 24.4vw 99vh 0 -0.17rem#fff,
    38.8vw 44.8vh 0 -0.5rem#fff, 50.6vw 45vh 0 -0.02rem#fff,
    29.5vw 99.5vh 0 -0.17rem#fff, 52.2vw 81.9vh 0 -0.46rem#fff,
    2.1vw 74.1vh 0 -0.06rem#fff, 79.3vw 35.3vh 0 -0.15rem#fff,
    29.6vw 82.5vh 0 -0.3rem#fff, 93.7vw 14.4vh 0 -0.49rem#fff,
    36.9vw 91vh 0 -0.23rem#fff, 80.4vw 11.9vh 0 -0.05rem#fff,
    47.8vw 37.1vh 0 -0.32rem#fff, 38.3vw 19.4vh 0 -0.06rem#fff,
    76.6vw 31.8vh 0 -0.04rem#fff, 16.9vw 58.6vh 0 -0.49rem#fff,
    91.2vw 10.4vh 0 -0.18rem#fff, 40.7vw 38.2vh 0 -0.42rem#fff,
    18.2vw 98.9vh 0 -0.43rem#fff, 4.7vw 52.4vh 0 -0.48rem#fff,
    17.6vw 19.2vh 0 -0.2rem#fff, 80.8vw 60.4vh 0 -0.36rem#fff,
    95.5vw 1.8vh 0 -0.46rem#fff, 34.5vw 48.5vh 0 -0.19rem#fff,
    45.6vw 24.4vh 0 -0.21rem#fff, 83.2vw 6.7vh 0 -0.37rem#fff,
    80.2vw 70.4vh 0 -0.3rem#fff, 99.9vw 62.8vh 0 -0.17rem#fff,
    65.7vw 7.8vh 0 -0.13rem#fff, 91.2vw 95.5vh 0 -0.01rem#fff,
    63.8vw 8.5vh 0 -0.22rem#fff, 75.9vw 43.8vh 0 -0.31rem#fff,
    35.9vw 54.3vh 0 -0.32rem#fff, 96.6vw 26.5vh 0 -0.18rem#fff,
    33.8vw 39.3vh 0 -0.12rem#fff, 97.6vw 43.1vh 0 -0.48rem#fff,
    68.8vw 36.7vh 0 -0.07rem#fff, 52.5vw 7.2vh 0 -0.28rem#fff,
    98vw 79.6vh 0 -0.26rem#fff, 35.2vw 19.5vh 0 -0.41rem#fff,
    49.8vw 55.6vh 0 -0.36rem#fff, 16.1vw 55.3vh 0 -0.12rem#fff,
    19.7vw 13.1vh 0 -0.04rem#fff, 24.7vw 32vh 0 -0.17rem#fff,
    13.2vw 81.6vh 0 -0.13rem#fff, 42.8vw 96.9vh 0 -0.2rem#fff,
    98vw 31.3vh 0 -0.4rem#fff, 93.5vw 34.2vh 0 -0.35rem#fff,
    55.3vw 74.7vh 0 -0.31rem#fff, 75.4vw 40vh 0 -0.22rem#fff,
    87.3vw 39.1vh 0 -0.2rem#fff, 46.2vw 78.7vh 0 -0.35rem#fff,
    91.9vw 20.4vh 0 -0.16rem#fff, 98.3vw 5.4vh 0 -0.11rem#fff,
    93.5vw 34.5vh 0 -0.41rem#fff, 52.5vw 69.8vh 0 -0.15rem#fff,
    42.1vw 3.5vh 0 -0.29rem#fff, 90.6vw 6.5vh 0 -0.37rem#fff,
    13.3vw 10.1vh 0 -0.34rem#fff, 8.4vw 44.8vh 0 -0.02rem#fff,
    45.5vw 93.4vh 0 -0.36rem#fff, 55.4vw 24.5vh 0 -0.27rem#fff,
    75vw 59.7vh 0 -0.03rem#fff, 31.7vw 27.9vh 0 -0.2rem#fff,
    67.2vw 18.1vh 0 -0.46rem#fff, 7.5vw 52.8vh 0 -0.43rem#fff,
    81.3vw 30.6vh 0 -0.38rem#fff, 5.4vw 79.7vh 0 -0.02rem#fff,
    64.7vw 53.3vh 0 -0.34rem#fff, 52vw 93.2vh 0 -0.44rem#fff,
    64vw 98.1vh 0 -0.01rem#fff, 23.4vw 5.8vh 0 -0.33rem#fff,
    40.9vw 57.3vh 0 -0.15rem#fff, 23.7vw 64.9vh 0 -0.17rem#fff,
    54.1vw 15.8vh 0 -0.06rem#fff, 60.6vw 45.3vh 0 -0.3rem#fff,
    13.9vw 69.1vh 0 -0.37rem#fff, 61.9vw 69.6vh 0 -0.35rem#fff,
    6.6vw 56.6vh 0 -0.15rem#fff, 47.7vw 58.1vh 0 -0.37rem#fff,
    14.9vw 61.9vh 0 -0.44rem#fff, 34.5vw 10.2vh 0 -0.1rem#fff,
    45.3vw 6.6vh 0 -0.22rem#fff, 75.9vw 67.7vh 0 -0.04rem#fff,
    74.4vw 58.8vh 0 -0.02rem#fff, 68.4vw 3.2vh 0 -0.5rem#fff,
    33.7vw 25.6vh 0 -0.46rem#fff, 9.7vw 6.7vh 0 -0.16rem#fff,
    51.2vw 92.3vh 0 -0.21rem#fff, 21.3vw 69.3vh 0 -0.15rem#fff,
    63.4vw 17.6vh 0 -0.16rem#fff, 77.3vw 27.8vh 0 -0.49rem#fff,
    55.6vw 29.1vh 0 -0.42rem#fff, 16.3vw 92.2vh 0 -0.12rem#fff,
    19.9vw 85.5vh 0 -0.09rem#fff, 73.6vw 19vh 0 -0.41rem#fff,
    35.4vw 67.9vh 0 -0.3rem#fff, 58.3vw 26vh 0 -0.07rem#fff,
    81.3vw 2vh 0 -0.49rem#fff, 46.6vw 84.5vh 0 -0.28rem#fff,
    51.5vw 33.2vh 0 -0.04rem#fff, 98.6vw 59vh 0 -0.21rem#fff,
    20.8vw 7vh 0 -0.38rem#fff, 91.1vw 99.8vh 0 -0.07rem#fff,
    82.9vw 55.7vh 0 -0.38rem#fff, 36.1vw 8.9vh 0 -0.4rem#fff,
    17.7vw 22.4vh 0 -0.07rem#fff, 9.8vw 97vh 0 -0.32rem#fff,
    42.7vw 41.2vh 0 -0.31rem#fff, 2vw 78.1vh 0 -0.47rem#fff,
    16.2vw 41.3vh 0 -0.29rem#fff, 17.3vw 58.5vh 0 -0.21rem#fff,
    98.8vw 23.2vh 0 -0.48rem#fff, 62.9vw 56.4vh 0 -0.21rem#fff,
    18.9vw 65.9vh 0 -0.39rem#fff, 83vw 6vh 0 -0.31rem#fff,
    71.6vw 40.3vh 0 -0.45rem#fff, 35.9vw 53.6vh 0 -0.05rem#fff,
    85.2vw 13.3vh 0 -0.46rem#fff, 81.2vw 87.8vh 0 -0.36rem#fff,
    22.2vw 11.9vh 0 -0.28rem#fff, 89.5vw 76.8vh 0 -0.12rem#fff,
    96.7vw 24.2vh 0 -0.17rem#fff, 48.7vw 80.6vh 0 -0.27rem#fff,
    58.6vw 26.4vh 0 -0.23rem#fff, 12.5vw 34.7vh 0 -0.4rem#fff,
    96.8vw 45.4vh 0 -0.13rem#fff, 74.3vw 50.1vh 0 -0.09rem#fff,
    68.8vw 21vh 0 -0.19rem#fff, 96.1vw 36.3vh 0 -0.31rem#fff,
    98.8vw 61.8vh 0 -0.44rem#fff, 36.9vw 78.9vh 0 -0.47rem#fff,
    87.7vw 75.4vh 0 -0.1rem#fff, 35.3vw 74.4vh 0 -0.27rem#fff,
    44.4vw 12.2vh 0 -0.15rem#fff, 90.3vw 90.8vh 0 -0.47rem#fff,
    62.9vw 5.6vh 0 -0.49rem#fff, 78.5vw 97.4vh 0 -0.27rem#fff,
    33.1vw 86.6vh 0 -0.39rem#fff, 28vw 22.9vh 0 -0.07rem#fff,
    53.5vw 61vh 0 -0.37rem#fff, 7.1vw 3.2vh 0 -0.05rem#fff,
    76.1vw 98.7vh 0 -0.36rem#fff, 92vw 73.5vh 0 -0.07rem#fff,
    58.7vw 49vh 0 -0.01rem#fff, 98vw 88.3vh 0 -0.14rem#fff,
    63.3vw 78.9vh 0 -0.44rem#fff, 67.6vw 78.3vh 0 -0.48rem#fff,
    62.3vw 19.4vh 0 -0.43rem#fff, 73.7vw 45vh 0 -0.36rem#fff,
    86.3vw 72.4vh 0 -0.01rem#fff, 94.7vw 67.9vh 0 -0.28rem#fff,
    46.5vw 94.9vh 0 -0.35rem#fff, 13.3vw 41.8vh 0 -0.3rem#fff,
    73.2vw 79.2vh 0 -0.01rem#fff, 3.6vw 74.9vh 0 -0.13rem#fff,
    96.7vw 46.6vh 0 -0.32rem#fff, 61.9vw 33.1vh 0 -0.24rem#fff,
    24.3vw 86.2vh 0 -0.25rem#fff, 9.3vw 61.2vh 0 -0.01rem#fff,
    9vw 85.6vh 0 -0.13rem#fff, 24.7vw 25.7vh 0 -0.18rem#fff,
    64.6vw 73.8vh 0 -0.48rem#fff, 56.8vw 96.9vh 0 -0.45rem#fff,
    89.1vw 20.2vh 0 -0.34rem#fff, 3.4vw 4.6vh 0 -0.35rem#fff,
    11.6vw 4.7vh 0 -0.22rem#fff, 2.5vw 82.7vh 0 -0.28rem#fff,
    44.5vw 93.7vh 0 -0.42rem#fff, 61.6vw 13.8vh 0 -0.1rem#fff,
    34.5vw 52.9vh 0 -0.49rem#fff, 39.6vw 15.5vh 0 -0.23rem#fff,
    16.7vw 88.1vh 0 -0.3rem#fff, 44.8vw 75.1vh 0 -0.19rem#fff,
    95.9vw 61.8vh 0 -0.11rem#fff, 2.5vw 12.7vh 0 -0.06rem#fff,
    31vw 1.3vh 0 -0.07rem#fff, 62.3vw 2.6vh 0 -0.09rem#fff,
    33.5vw 15.9vh 0 -0.12rem#fff, 55.4vw 75.2vh 0 -0.09rem#fff,
    10.9vw 29.3vh 0 -0.25rem#fff, 3.3vw 60vh 0 -0.41rem#fff,
    15.7vw 77.5vh 0 -0.04rem#fff, 51.8vw 10.9vh 0 -0.45rem#fff,
    78vw 91.2vh 0 -0.24rem#fff, 59.9vw 14vh 0 -0.2rem#fff,
    73.1vw 90.2vh 0 -0.13rem#fff, 1.4vw 4.4vh 0 -0.34rem#fff,
    6vw 99.2vh 0 -0.15rem#fff, 55.6vw 89.4vh 0 -0.19rem#fff,
    95.2vw 12.4vh 0 -0.06rem#fff, 31.9vw 95.5vh 0 -0.1rem#fff,
    17vw 76.3vh 0 -0.05rem#fff, 48.8vw 47.3vh 0 -0.17rem#fff,
    8.9vw 68.8vh 0 -0.03rem#fff, 54.2vw 65.5vh 0 -0.41rem#fff,
    22.1vw 11.1vh 0 -0.44rem#fff, 59vw 41vh 0 -0.48rem#fff,
    36.5vw 26.2vh 0 -0.11rem#fff, 35.8vw 71.2vh 0 -0.38rem#fff,
    7.5vw 26.5vh 0 -0.49rem#fff, 14.4vw 27.4vh 0 -0.33rem#fff,
    95vw 2vh 0 -0.36rem#fff, 84.8vw 96.4vh 0 -0.29rem#fff,
    38.9vw 43.3vh 0 -0.43rem#fff, 75.2vw 76.2vh 0 -0.31rem#fff,
    20.9vw 10.8vh 0 -0.05rem#fff, 22.7vw 42.2vh 0 -0.15rem#fff,
    25.9vw 22.2vh 0 -0.11rem#fff, 61.6vw 94.3vh 0 -0.3rem#fff,
    97vw 95.5vh 0 -0.45rem#fff, 80.4vw 46.5vh 0 -0.44rem#fff,
    17.9vw 12.1vh 0 -0.14rem#fff, 9.8vw 46.7vh 0 -0.3rem#fff,
    36.5vw 12.5vh 0 -0.47rem#fff, 23.2vw 7.1vh 0 -0.08rem#fff,
    72.7vw 69vh 0 -0.49rem#fff, 37.5vw 11.8vh 0 -0.41rem#fff,
    58.4vw 46.9vh 0 -0.37rem#fff, 61.2vw 10.8vh 0 -0.22rem#fff,
    85vw 6.2vh 0 -0.01rem#fff, 55.1vw 24.5vh 0 -0.23rem#fff,
    74.6vw 62vh 0 -0.45rem#fff, 33.2vw 30.7vh 0 -0.16rem#fff,
    72.4vw 73.2vh 0 -0.33rem#fff, 32.2vw 30.5vh 0 -0.08rem#fff,
    9.8vw 4.3vh 0 -0.47rem#fff, 49.1vw 81.7vh 0 -0.36rem#fff,
    50.4vw 83.2vh 0 -0.05rem#fff, 98.1vw 54.2vh 0 -0.23rem#fff,
    89.9vw 5.1vh 0 -0.26rem#fff, 88.6vw 54.1vh 0 -0.5rem#fff,
    51.3vw 68.3vh 0 -0.27rem#fff, 94.9vw 62.4vh 0 -0.38rem#fff,
    70.8vw 33.3vh 0 -0.16rem#fff, 15.8vw 66.9vh 0 -0.36rem#fff,
    20.9vw 14.7vh 0 -0.19rem#fff, 29.8vw 48.6vh 0 -0.08rem#fff,
    0.7vw 60.2vh 0 -0.47rem#fff, 81vw 68.3vh 0 -0.22rem#fff,
    61.2vw 41.8vh 0 -0.5rem#fff, 69.2vw 49.2vh 0 -0.17rem#fff,
    10vw 15.6vh 0 -0.24rem#fff, 11.6vw 96.4vh 0 -0.16rem#fff,
    27.6vw 24.1vh 0 -0.24rem#fff, 21.4vw 78.8vh 0 -0.28rem#fff,
    10.9vw 28.8vh 0 -0.39rem#fff, 23.5vw 78.7vh 0 -0.15rem#fff,
    48.5vw 39.5vh 0 -0.35rem#fff, 4.8vw 83.9vh 0 -0.11rem#fff,
    26.2vw 62.7vh 0 -0.18rem#fff, 66.8vw 85vh 0 -0.01rem#fff,
    58.1vw 77.2vh 0 -0.08rem#fff, 6.8vw 13.6vh 0 -0.02rem#fff,
    89.5vw 92.3vh 0 -0.38rem#fff, 67.1vw 11.4vh 0 -0.19rem#fff,
    7.1vw 70.9vh 0 -0.09rem#fff, 62.4vw 79.4vh 0 -0.34rem#fff,
    2.9vw 47vh 0 -0.32rem#fff, 83.3vw 48.2vh 0 -0.34rem#fff,
    27.1vw 96.9vh 0 -0.3rem#fff, 98.1vw 42.1vh 0 -0.49rem#fff,
    88.3vw 40.6vh 0 -0.3rem#fff, 82.3vw 38.8vh 0 -0.13rem#fff,
    67.5vw 81.6vh 0 -0.09rem#fff, 61.1vw 30.6vh 0 -0.38rem#fff,
    82.6vw 14.8vh 0 -0.48rem#fff, 99.2vw 60.1vh 0 -0.29rem#fff,
    35.4vw 45.6vh 0 -0.06rem#fff, 98.5vw 44.2vh 0 -0.45rem#fff,
    3.3vw 48.5vh 0 -0.12rem#fff, 80.1vw 76.4vh 0 -0.29rem#fff,
    98.2vw 5.7vh 0 -0.27rem#fff, 63.6vw 3.6vh 0 -0.43rem#fff,
    42.4vw 55.9vh 0 -0.02rem#fff, 34.3vw 36vh 0 -0.11rem#fff,
    80vw 87.2vh 0 -0.05rem#fff, 57.8vw 70.7vh 0 -0.32rem#fff,
    23.8vw 54.8vh 0 -0.24rem#fff, 27.2vw 42.5vh 0 -0.1rem#fff,
    34.9vw 69.3vh 0 -0.09rem#fff, 56.9vw 84.9vh 0 -0.07rem#fff,
    61.9vw 93.8vh 0 -0.2rem#fff, 52.6vw 62vh 0 -0.35rem#fff,
    75.2vw 63.3vh 0 -0.27rem#fff, 35.6vw 1.4vh 0 -0.32rem#fff,
    79vw 2.7vh 0 -0.21rem#fff, 19.3vw 66.5vh 0 -0.5rem#fff,
    24.4vw 55.5vh 0 -0.49rem#fff, 3.7vw 95vh 0 -0.07rem#fff,
    52.7vw 4.6vh 0 -0.16rem#fff, 71.6vw 88vh 0 -0.29rem#fff,
    50.6vw 11.7vh 0 -0.01rem#fff, 64.1vw 70.9vh 0 -0.24rem#fff,
    8.8vw 31.1vh 0 -0.43rem#fff, 28.8vw 6.1vh 0 -0.29rem#fff,
    63.4vw 81.8vh 0 -0.15rem#fff, 40.6vw 43.2vh 0 -0.15rem#fff,
    78.8vw 98.5vh 0 -0.46rem#fff, 30.3vw 53.6vh 0 -0.35rem#fff,
    48.5vw 43.6vh 0 -0.19rem#fff, 76.8vw 93.5vh 0 -0.42rem#fff,
    91.2vw 90.7vh 0 -0.02rem#fff, 53.2vw 68.9vh 0 -0.34rem#fff,
    34.9vw 45.5vh 0 -0.14rem#fff, 9vw 5.4vh 0 -0.24rem#fff,
    2.8vw 53.2vh 0 -0.39rem#fff, 8.3vw 95.7vh 0 -0.29rem#fff,
    24.1vw 62.6vh 0 -0.01rem#fff, 85.1vw 43.6vh 0 -0.01rem#fff,
    1.1vw 60.1vh 0 -0.43rem#fff, 75.9vw 53.2vh 0 -0.34rem#fff,
    67.8vw 35vh 0 -0.24rem#fff, 84.7vw 67.2vh 0 -0.5rem#fff,
    27.9vw 2.1vh 0 -0.48rem#fff, 48.6vw 25.8vh 0 -0.5rem#fff,
    65.6vw 34.3vh 0 -0.43rem#fff, 75.1vw 74.2vh 0 -0.19rem#fff,
    33.2vw 4.3vh 0 -0.5rem#fff, 58.8vw 17vh 0 -0.42rem#fff,
    56.2vw 24.7vh 0 -0.24rem#fff, 85.2vw 79.3vh 0 -0.22rem#fff,
    29.1vw 65.5vh 0 -0.3rem#fff, 4vw 76.5vh 0 -0.16rem#fff,
    84.5vw 75.8vh 0 -0.01rem#fff, 88.8vw 71.5vh 0 -0.26rem#fff,
    10.6vw 45vh 0 -0.03rem#fff, 53.8vw 20.8vh 0 -0.14rem#fff,
    31.7vw 50.7vh 0 -0.42rem#fff, 27.9vw 18.6vh 0 -0.02rem#fff,
    51.3vw 72.6vh 0 -0.26rem#fff, 53.8vw 17vh 0 -0.29rem#fff,
    10.3vw 97.6vh 0 -0.08rem#fff, 42.7vw 81.9vh 0 -0.34rem#fff,
    79.9vw 16.3vh 0 -0.19rem#fff, 75.6vw 57.5vh 0 -0.49rem#fff,
    99.4vw 55.6vh 0 -0.11rem#fff, 42.4vw 51.6vh 0 -0.27rem#fff,
    80.9vw 51.4vh 0 -0.04rem#fff, 23vw 42.5vh 0 -0.33rem#fff,
    3vw 20.4vh 0 -0.12rem#fff, 3.5vw 83.2vh 0 -0.29rem#fff,
    24.2vw 51.8vh 0 -0.28rem#fff, 51.7vw 79.2vh 0 -0.28rem#fff,
    30.4vw 59.8vh 0 -0.4rem#fff, 6.2vw 22.2vh 0 -0.31rem#fff,
    77.9vw 98.2vh 0 -0.48rem#fff, 16.3vw 63vh 0 -0.25rem#fff,
    90.8vw 85.2vh 0 -0.19rem#fff, 58.9vw 38.3vh 0 -0.21rem#fff,
    23.9vw 27.4vh 0 -0.1rem#fff, 93.6vw 52.4vh 0 -0.49rem#fff,
    4.6vw 77vh 0 -0.18rem#fff, 66vw 2.3vh 0 -0.06rem#fff,
    39.5vw 40.2vh 0 -0.17rem#fff, 13.5vw 57.4vh 0 -0.09rem#fff,
    27vw 71.1vh 0 -0.44rem#fff, 75.1vw 95.1vh 0 -0.33rem#fff,
    18.7vw 83vh 0 -0.41rem#fff, 96.4vw 79.4vh 0 -0.02rem#fff,
    38.7vw 79.7vh 0 -0.28rem#fff, 19.8vw 20.4vh 0 -0.17rem#fff,
    49.1vw 76.3vh 0 -0.15rem#fff, 98vw 98vh 0 -0.48rem#fff,
    4.3vw 34.9vh 0 -0.27rem#fff, 17.2vw 33.5vh 0 -0.43rem#fff,
    67.1vw 3.2vh 0 -0.33rem#fff, 65.8vw 55.3vh 0 -0.17rem#fff,
    17vw 18.7vh 0 -0.34rem#fff, 15vw 52.4vh 0 -0.2rem#fff,
    74.4vw 97.2vh 0 -0.43rem#fff, 60.6vw 53.6vh 0 -0.13rem#fff,
    69.7vw 47.8vh 0 -0.38rem#fff, 25.5vw 53.1vh 0 -0.37rem#fff,
    35.3vw 36.6vh 0 -0.2rem#fff, 91.4vw 61vh 0 -0.5rem#fff,
    41.4vw 47.2vh 0 -0.21rem#fff, 56.5vw 28.9vh 0 -0.31rem#fff,
    21.5vw 6.1vh 0 -0.37rem#fff, 36.1vw 93.2vh 0 -0.02rem#fff,
    68vw 77.9vh 0 -0.11rem#fff, 37.1vw 47.4vh 0 -0.21rem#fff,
    35.8vw 10.7vh 0 -0.03rem#fff, 12.8vw 29.5vh 0 -0.43rem#fff,
    59.2vw 75.9vh 0 -0.48rem#fff, 43.3vw 36.4vh 0 -0.29rem#fff,
    58.3vw 55.4vh 0 -0.27rem#fff, 15.6vw 8vh 0 -0.27rem#fff;
  animation-duration: 8s;
}
.layer2.a {
  animation-delay: -4s;
}
.layer3 {
  width: 0.8rem;
  height: 0.8rem;
  filter: blur(6px);
  box-shadow: 56vw 84vh 0 -0.2rem#fff, 40.8vw 87.5vh 0 -0.5rem#fff,
    83.4vw 0.6vh 0 -0.1rem#fff, 54.2vw 10.7vh 0 -0.47rem#fff,
    94.3vw 41.3vh 0 -0.27rem#fff, 51.2vw 82.7vh 0 -0.37rem#fff,
    68.7vw 74.6vh 0 -0.11rem#fff, 93.8vw 1.3vh 0 -0.14rem#fff,
    16.1vw 33.3vh 0 -0.3rem#fff, 43.3vw 51vh 0 -0.33rem#fff,
    61.8vw 98.8vh 0 -0.45rem#fff, 85.2vw 14.5vh 0 -0.14rem#fff,
    74.9vw 15vh 0 -0.14rem#fff, 13vw 99.2vh 0 -0.27rem#fff,
    15.6vw 4.3vh 0 -0.24rem#fff, 5.7vw 11.6vh 0 -0.43rem#fff,
    33.2vw 47.8vh 0 -0.09rem#fff, 10.8vw 11.6vh 0 -0.37rem#fff,
    88vw 78.9vh 0 -0.45rem#fff, 13.2vw 24vh 0 -0.13rem#fff,
    35.5vw 20.9vh 0 -0.38rem#fff, 45.9vw 89.8vh 0 -0.23rem#fff,
    69vw 43.4vh 0 -0.1rem#fff, 12.4vw 37.4vh 0 -0.29rem#fff,
    1.6vw 54.4vh 0 -0.36rem#fff, 38.9vw 89.9vh 0 -0.18rem#fff,
    47.8vw 10.7vh 0 -0.21rem#fff, 91vw 78vh 0 -0.29rem#fff,
    41.7vw 59.4vh 0 -0.24rem#fff, 74.7vw 79.9vh 0 -0.33rem#fff,
    69.5vw 51vh 0 -0.13rem#fff, 65.8vw 90.6vh 0 -0.37rem#fff,
    88vw 65.4vh 0 -0.34rem#fff, 83.8vw 16.2vh 0 -0.37rem#fff,
    16.5vw 57.1vh 0 -0.27rem#fff, 92vw 12.3vh 0 -0.27rem#fff,
    19.1vw 33.6vh 0 -0.31rem#fff, 6.5vw 92.1vh 0 -0.11rem#fff,
    90.3vw 22.8vh 0 -0.25rem#fff, 84.9vw 27.7vh 0 -0.26rem#fff,
    9.6vw 73.5vh 0 -0.36rem#fff, 98.1vw 42.2vh 0 -0.37rem#fff,
    24vw 64vh 0 -0.31rem#fff, 81.3vw 51vh 0 -0.47rem#fff,
    31.5vw 14vh 0 -0.19rem#fff, 48.5vw 36.3vh 0 -0.22rem#fff,
    74.8vw 12.3vh 0 -0.05rem#fff, 27.9vw 84.4vh 0 -0.27rem#fff,
    62.5vw 81.4vh 0 -0.24rem#fff, 50.5vw 55.2vh 0 -0.46rem#fff,
    66.4vw 63vh 0 -0.06rem#fff, 33.2vw 18.5vh 0 -0.13rem#fff,
    4.4vw 47.2vh 0 -0.02rem#fff, 69.1vw 68.9vh 0 -0.45rem#fff,
    96.5vw 32.8vh 0 -0.15rem#fff, 44.7vw 86.7vh 0 -0.41rem#fff,
    24.8vw 27.9vh 0 -0.25rem#fff, 25.9vw 85.7vh 0 -0.46rem#fff,
    75.4vw 1.4vh 0 -0.03rem#fff, 71.4vw 39.1vh 0 -0.13rem#fff,
    97.6vw 41vh 0 -0.39rem#fff, 46.2vw 76.3vh 0 -0.04rem#fff,
    28.3vw 96vh 0 -0.37rem#fff, 94.6vw 43vh 0 -0.28rem#fff,
    56vw 60.8vh 0 -0.21rem#fff, 72.7vw 25.7vh 0 -0.37rem#fff,
    89.9vw 48.4vh 0 -0.29rem#fff, 21.6vw 12vh 0 -0.11rem#fff,
    94.4vw 59.7vh 0 -0.38rem#fff, 7.2vw 77.5vh 0 -0.01rem#fff,
    97.8vw 100vh 0 -0.47rem#fff, 48.1vw 85.4vh 0 -0.22rem#fff,
    68.8vw 91.1vh 0 -0.27rem#fff, 93.3vw 32vh 0 -0.03rem#fff,
    40.1vw 12.9vh 0 -0.16rem#fff, 54.6vw 30.8vh 0 -0.4rem#fff,
    60.7vw 32.6vh 0 -0.35rem#fff, 83.9vw 30.9vh 0 -0.26rem#fff,
    26.1vw 93.9vh 0 -0.18rem#fff, 75.5vw 90.8vh 0 -0.38rem#fff,
    95.5vw 12.7vh 0 -0.1rem#fff, 70.2vw 25.9vh 0 -0.38rem#fff,
    87.2vw 65.4vh 0 -0.31rem#fff, 99.1vw 14.5vh 0 -0.36rem#fff,
    74.9vw 32.4vh 0 -0.39rem#fff, 84.1vw 6vh 0 -0.05rem#fff,
    81.6vw 86.3vh 0 -0.06rem#fff, 43.6vw 24vh 0 -0.45rem#fff,
    14.9vw 46vh 0 -0.06rem#fff, 74.6vw 24.6vh 0 -0.41rem#fff,
    62.5vw 36.2vh 0 -0.02rem#fff, 33.6vw 38.5vh 0 -0.46rem#fff,
    35.1vw 66.6vh 0 -0.13rem#fff, 74.1vw 89.9vh 0 -0.39rem#fff,
    22vw 84.4vh 0 -0.15rem#fff, 85.6vw 68.4vh 0 -0.12rem#fff,
    27.4vw 50.8vh 0 -0.31rem#fff, 52.7vw 59.1vh 0 -0.39rem#fff,
    63.4vw 14.7vh 0 -0.5rem#fff, 95.7vw 15.7vh 0 -0.11rem#fff,
    68.2vw 63.8vh 0 -0.15rem#fff, 12.6vw 88.1vh 0 -0.29rem#fff,
    28.6vw 82vh 0 -0.44rem#fff, 74.5vw 87.8vh 0 -0.24rem#fff,
    37.1vw 87.5vh 0 -0.47rem#fff, 26.5vw 95.3vh 0 -0.09rem#fff,
    85.2vw 62vh 0 -0.32rem#fff, 31.5vw 60.8vh 0 -0.21rem#fff,
    79.4vw 88.7vh 0 -0.33rem#fff, 3vw 99.3vh 0 -0.43rem#fff,
    87.4vw 98.3vh 0 -0.34rem#fff, 42.1vw 83.7vh 0 -0.26rem#fff,
    82.5vw 94.2vh 0 -0.48rem#fff, 70.7vw 46.1vh 0 -0.08rem#fff,
    37.1vw 49.8vh 0 -0.23rem#fff, 40.8vw 53.4vh 0 -0.29rem#fff,
    0.3vw 27.1vh 0 -0.31rem#fff, 12vw 55.1vh 0 -0.49rem#fff,
    38vw 95.9vh 0 -0.39rem#fff, 59.7vw 57.8vh 0 -0.17rem#fff,
    40.9vw 68.3vh 0 -0.28rem#fff, 94.5vw 86.7vh 0 -0.21rem#fff,
    86vw 59vh 0 -0.14rem#fff, 66.8vw 42.9vh 0 -0.35rem#fff,
    7.5vw 24.3vh 0 -0.01rem#fff, 25.3vw 10.7vh 0 -0.42rem#fff,
    20.2vw 54.8vh 0 -0.02rem#fff, 41.7vw 63.2vh 0 -0.27rem#fff,
    6.6vw 69.7vh 0 -0.08rem#fff, 38.2vw 89.2vh 0 -0.36rem#fff,
    9.2vw 5.3vh 0 -0.08rem#fff, 9.7vw 62.1vh 0 -0.31rem#fff,
    6.7vw 24.7vh 0 -0.22rem#fff, 64.9vw 48.9vh 0 -0.12rem#fff,
    96.8vw 80.7vh 0 -0.15rem#fff, 42.1vw 5.8vh 0 -0.28rem#fff,
    57.8vw 32vh 0 -0.03rem#fff, 83.3vw 94.2vh 0 -0.2rem#fff,
    84.7vw 26.3vh 0 -0.31rem#fff, 18.7vw 55.6vh 0 -0.02rem#fff,
    3.6vw 4.8vh 0 -0.07rem#fff, 54.8vw 23.2vh 0 -0.13rem#fff,
    65.2vw 11.2vh 0 -0.37rem#fff, 52.8vw 3.1vh 0 -0.38rem#fff,
    81.1vw 5vh 0 -0.38rem#fff, 13.4vw 17.5vh 0 -0.24rem#fff,
    60.8vw 14vh 0 -0.12rem#fff, 71.7vw 95.2vh 0 -0.06rem#fff,
    20.5vw 1.4vh 0 -0.46rem#fff, 71.4vw 70vh 0 -0.14rem#fff,
    63.3vw 59.7vh 0 -0.2rem#fff, 19vw 45.7vh 0 -0.41rem#fff,
    93.5vw 11.6vh 0 -0.12rem#fff, 20.3vw 19.7vh 0 -0.18rem#fff,
    87vw 4.2vh 0 -0.49rem#fff, 23.4vw 56.1vh 0 -0.25rem#fff,
    2.6vw 59.9vh 0 -0.4rem#fff, 76.5vw 75.1vh 0 -0.36rem#fff,
    12vw 9.2vh 0 -0.4rem#fff, 95.6vw 17vh 0 -0.05rem#fff,
    81.9vw 44.2vh 0 -0.12rem#fff, 57vw 65.3vh 0 -0.44rem#fff,
    21.8vw 23.4vh 0 -0.05rem#fff, 6.6vw 69.5vh 0 -0.44rem#fff,
    63.2vw 54.7vh 0 -0.15rem#fff, 23.4vw 8.2vh 0 -0.26rem#fff,
    96.3vw 37.6vh 0 -0.5rem#fff, 49.3vw 87.6vh 0 -0.34rem#fff,
    11.9vw 70.5vh 0 -0.22rem#fff, 80.8vw 31.1vh 0 -0.35rem#fff,
    72.8vw 2.2vh 0 -0.31rem#fff, 90.7vw 5.4vh 0 -0.49rem#fff,
    53vw 67vh 0 -0.02rem#fff, 79.1vw 76.8vh 0 -0.21rem#fff,
    59.9vw 3.6vh 0 -0.49rem#fff, 91.2vw 79vh 0 -0.29rem#fff,
    17.6vw 9.7vh 0 -0.17rem#fff, 39.4vw 14.8vh 0 -0.45rem#fff,
    38.8vw 74.8vh 0 -0.3rem#fff, 3.2vw 58.8vh 0 -0.14rem#fff,
    28.6vw 65.9vh 0 -0.25rem#fff, 55.7vw 11.2vh 0 -0.25rem#fff,
    99.8vw 24.2vh 0 -0.34rem#fff, 50.3vw 57.2vh 0 -0.09rem#fff,
    62.8vw 22.9vh 0 -0.29rem#fff, 62vw 57.3vh 0 -0.43rem#fff,
    16.9vw 25.5vh 0 -0.48rem#fff, 75.9vw 8vh 0 -0.24rem#fff,
    1.5vw 7.3vh 0 -0.39rem#fff, 61.9vw 14.2vh 0 -0.26rem#fff,
    12.1vw 23.7vh 0 -0.26rem#fff, 75.9vw 2.7vh 0 -0.03rem#fff,
    85.9vw 36.6vh 0 -0.08rem#fff, 61.8vw 46.9vh 0 -0.4rem#fff,
    81.2vw 66.8vh 0 -0.14rem#fff, 16vw 22.1vh 0 -0.36rem#fff,
    96.1vw 20vh 0 -0.46rem#fff, 55.2vw 72.8vh 0 -0.37rem#fff,
    86.8vw 70.5vh 0 -0.42rem#fff, 20.8vw 3.8vh 0 -0.16rem#fff,
    70.9vw 93.8vh 0 -0.37rem#fff, 97.1vw 64.5vh 0 -0.06rem#fff,
    34.8vw 69.1vh 0 -0.48rem#fff, 49.8vw 1.5vh 0 -0.42rem#fff,
    71.7vw 34.4vh 0 -0.42rem#fff, 94.8vw 78.4vh 0 -0.02rem#fff,
    71.3vw 86vh 0 -0.32rem#fff, 17vw 87.7vh 0 -0.21rem#fff,
    99.8vw 33.1vh 0 -0.02rem#fff, 27.2vw 7.3vh 0 -0.37rem#fff,
    69.1vw 52.7vh 0 -0.23rem#fff, 82.7vw 21.1vh 0 -0.34rem#fff,
    63.8vw 82.6vh 0 -0.05rem#fff, 73.5vw 23.9vh 0 -0.48rem#fff,
    40.5vw 76.5vh 0 -0.49rem#fff, 30vw 15.8vh 0 -0.46rem#fff,
    59.4vw 51.3vh 0 -0.24rem#fff, 49.6vw 43.4vh 0 -0.12rem#fff,
    38.1vw 89.3vh 0 -0.3rem#fff, 96.6vw 26.7vh 0 -0.42rem#fff,
    98.3vw 22.3vh 0 -0.37rem#fff, 19.1vw 5.3vh 0 -0.49rem#fff,
    51.9vw 83.9vh 0 -0.12rem#fff, 91.1vw 60.1vh 0 -0.48rem#fff,
    83.2vw 54.7vh 0 -0.31rem#fff, 53vw 17vh 0 -0.25rem#fff,
    11.5vw 48.5vh 0 -0.26rem#fff, 34.9vw 53.1vh 0 -0.4rem#fff,
    61.8vw 2.4vh 0 -0.43rem#fff, 64.2vw 75.7vh 0 -0.26rem#fff,
    57.4vw 43.8vh 0 -0.1rem#fff, 78.9vw 38.9vh 0 -0.23rem#fff,
    36vw 84.8vh 0 -0.15rem#fff, 81.6vw 36.6vh 0 -0.11rem#fff,
    92.7vw 72.4vh 0 -0.41rem#fff, 30.8vw 26.5vh 0 -0.09rem#fff,
    42.8vw 57vh 0 -0.49rem#fff, 77.1vw 60.1vh 0 -0.06rem#fff,
    17.8vw 24.2vh 0 -0.4rem#fff, 21.7vw 71.1vh 0 -0.49rem#fff,
    38.4vw 54.5vh 0 -0.3rem#fff, 21.5vw 61.9vh 0 -0.49rem#fff,
    38vw 79.8vh 0 -0.41rem#fff, 45vw 92.8vh 0 -0.02rem#fff,
    44.6vw 76.3vh 0 -0.44rem#fff, 7.9vw 3.3vh 0 -0.22rem#fff,
    76vw 42.8vh 0 -0.44rem#fff, 20.6vw 26.9vh 0 -0.4rem#fff,
    63.9vw 82.1vh 0 -0.02rem#fff, 96.7vw 78.7vh 0 -0.5rem#fff,
    5.9vw 42.3vh 0 -0.28rem#fff, 46.1vw 23vh 0 -0.4rem#fff,
    25vw 65.1vh 0 -0.13rem#fff, 96.5vw 12.1vh 0 -0.36rem#fff,
    71.3vw 86.8vh 0 -0.25rem#fff, 79.5vw 12.6vh 0 -0.34rem#fff,
    26.4vw 25.6vh 0 -0.17rem#fff, 92.2vw 77.8vh 0 -0.29rem#fff,
    34vw 63.1vh 0 -0.12rem#fff, 15.5vw 69.9vh 0 -0.4rem#fff,
    80.9vw 78vh 0 -0.2rem#fff, 81.2vw 66.3vh 0 -0.08rem#fff,
    70.2vw 4.9vh 0 -0.1rem#fff, 80.4vw 41.4vh 0 -0.23rem#fff,
    27.6vw 14.2vh 0 -0.11rem#fff, 10.8vw 44.1vh 0 -0.06rem#fff,
    76.8vw 20.9vh 0 -0.27rem#fff, 43.3vw 74.5vh 0 -0.43rem#fff,
    19.3vw 63.7vh 0 -0.4rem#fff, 3.9vw 40vh 0 -0.47rem#fff,
    3.6vw 63.8vh 0 -0.24rem#fff, 44.8vw 73.2vh 0 -0.38rem#fff,
    95.4vw 76.5vh 0 -0.27rem#fff, 3.7vw 74.3vh 0 -0.06rem#fff,
    77vw 2.7vh 0 -0.29rem#fff, 50.3vw 100vh 0 -0.05rem#fff,
    33vw 67.4vh 0 -0.16rem#fff, 48.7vw 26.3vh 0 -0.31rem#fff,
    89.3vw 4vh 0 -0.34rem#fff, 92.8vw 52.8vh 0 -0.02rem#fff,
    60.6vw 32.4vh 0 -0.27rem#fff, 59.1vw 27.1vh 0 -0.13rem#fff,
    47.7vw 8.3vh 0 -0.37rem#fff, 1.2vw 74.1vh 0 -0.29rem#fff,
    35.6vw 10vh 0 -0.21rem#fff, 23vw 59.4vh 0 -0.25rem#fff,
    56.8vw 39.1vh 0 -0.1rem#fff, 27.4vw 13.8vh 0 -0.06rem#fff,
    98vw 83.9vh 0 -0.13rem#fff, 49.3vw 34.4vh 0 -0.46rem#fff,
    11.2vw 53.3vh 0 -0.25rem#fff, 51.3vw 56.3vh 0 -0.48rem#fff,
    20vw 5.1vh 0 -0.41rem#fff, 88.8vw 88.1vh 0 -0.3rem#fff,
    82.4vw 6.3vh 0 -0.29rem#fff, 83.2vw 19.3vh 0 -0.31rem#fff,
    4.1vw 97.2vh 0 -0.49rem#fff, 46.4vw 92.7vh 0 -0.28rem#fff,
    42.2vw 52.9vh 0 -0.05rem#fff, 23.8vw 41vh 0 -0.28rem#fff,
    70.6vw 45.4vh 0 -0.48rem#fff, 81.2vw 18.9vh 0 -0.18rem#fff,
    59.4vw 92.1vh 0 -0.38rem#fff, 3.6vw 43.6vh 0 -0.07rem#fff,
    15.6vw 16.2vh 0 -0.35rem#fff, 25.1vw 29.1vh 0 -0.36rem#fff,
    19.9vw 66.3vh 0 -0.22rem#fff, 66vw 42vh 0 -0.35rem#fff,
    33.7vw 25.7vh 0 -0.16rem#fff, 65.8vw 56.6vh 0 -0.09rem#fff,
    28.9vw 65.6vh 0 -0.26rem#fff, 26.5vw 12.3vh 0 -0.33rem#fff,
    69.1vw 44.7vh 0 -0.31rem#fff, 78.9vw 29.3vh 0 -0.33rem#fff,
    71.4vw 78.7vh 0 -0.5rem#fff, 36.5vw 11vh 0 -0.5rem#fff,
    96.7vw 34vh 0 -0.32rem#fff, 78.6vw 67.4vh 0 -0.28rem#fff,
    29.1vw 2.2vh 0 -0.3rem#fff, 23vw 81.2vh 0 -0.42rem#fff,
    18.8vw 25.4vh 0 -0.47rem#fff, 41.3vw 9.7vh 0 -0.23rem#fff,
    27.2vw 23vh 0 -0.29rem#fff, 6vw 26.4vh 0 -0.46rem#fff,
    27.4vw 12.2vh 0 -0.31rem#fff, 10.4vw 90.6vh 0 -0.44rem#fff,
    10vw 71.8vh 0 -0.42rem#fff, 17vw 48.9vh 0 -0.1rem#fff,
    48.6vw 75.5vh 0 -0.44rem#fff, 2.6vw 14vh 0 -0.34rem#fff,
    41.5vw 39.6vh 0 -0.28rem#fff, 50.3vw 72.3vh 0 -0.11rem#fff,
    49.5vw 64.4vh 0 -0.24rem#fff, 7.1vw 13.6vh 0 -0.36rem#fff,
    96.8vw 5.1vh 0 -0.26rem#fff, 79.5vw 13.8vh 0 -0.26rem#fff,
    88.2vw 33.2vh 0 -0.21rem#fff, 59.6vw 85.3vh 0 -0.03rem#fff,
    27.9vw 6.7vh 0 -0.17rem#fff, 92.5vw 39.5vh 0 -0.01rem#fff,
    0.9vw 17.9vh 0 -0.24rem#fff, 51.9vw 12.4vh 0 -0.33rem#fff,
    80.2vw 18vh 0 -0.47rem#fff, 80.8vw 58.9vh 0 -0.12rem#fff,
    0.9vw 14.4vh 0 -0.28rem#fff, 54.6vw 61vh 0 -0.49rem#fff,
    85.9vw 79.8vh 0 -0.12rem#fff, 66vw 3.8vh 0 -0.25rem#fff,
    61vw 15.8vh 0 -0.46rem#fff, 78.7vw 72.3vh 0 -0.02rem#fff,
    24.2vw 20.9vh 0 -0.43rem#fff, 52.6vw 64.4vh 0 -0.21rem#fff,
    34.7vw 47.7vh 0 -0.16rem#fff, 97vw 62vh 0 -0.07rem#fff,
    57.2vw 63.4vh 0 -0.41rem#fff, 33.1vw 6.3vh 0 -0.04rem#fff,
    81.6vw 16.8vh 0 -0.14rem#fff, 75vw 83.4vh 0 -0.12rem#fff,
    85.1vw 22.2vh 0 -0.16rem#fff, 78.7vw 53.6vh 0 -0.15rem#fff,
    92.3vw 39.1vh 0 -0.5rem#fff, 53.7vw 83.8vh 0 -0.02rem#fff,
    88.5vw 42.5vh 0 -0.42rem#fff, 93.6vw 20.1vh 0 -0.25rem#fff,
    1.2vw 68.9vh 0 -0.46rem#fff, 27.8vw 53.8vh 0 -0.24rem#fff,
    76.6vw 36.9vh 0 -0.5rem#fff, 41.6vw 23.6vh 0 -0.01rem#fff,
    53.4vw 92.8vh 0 -0.12rem#fff, 19.1vw 21.9vh 0 -0.1rem#fff,
    27.9vw 32.5vh 0 -0.25rem#fff, 82.1vw 94.1vh 0 -0.35rem#fff,
    98.4vw 92.6vh 0 -0.03rem#fff, 24.6vw 58.4vh 0 -0.38rem#fff,
    35.4vw 13.6vh 0 -0.12rem#fff, 72.8vw 87.3vh 0 -0.09rem#fff,
    44.2vw 13.4vh 0 -0.28rem#fff, 70.7vw 36.4vh 0 -0.31rem#fff,
    11.8vw 28.1vh 0 -0.26rem#fff, 50vw 73.9vh 0 -0.07rem#fff,
    99.7vw 11.7vh 0 -0.23rem#fff, 50.9vw 0.7vh 0 -0.06rem#fff,
    39.4vw 29.7vh 0 -0.07rem#fff, 13.7vw 3.7vh 0 -0.16rem#fff,
    69.8vw 41.2vh 0 -0.43rem#fff, 63.9vw 23.2vh 0 -0.07rem#fff,
    1.3vw 71.7vh 0 -0.35rem#fff, 35vw 55.6vh 0 -0.39rem#fff,
    90.1vw 40.4vh 0 -0.13rem#fff, 94vw 3.7vh 0 -0.37rem#fff,
    71.5vw 2.2vh 0 -0.08rem#fff, 61.3vw 92.5vh 0 -0.23rem#fff,
    9.5vw 51.7vh 0 -0.23rem#fff, 16.6vw 72.2vh 0 -0.01rem#fff,
    9.6vw 67.8vh 0 -0.2rem#fff, 18.6vw 93vh 0 -0.49rem#fff,
    61.1vw 19.8vh 0 -0.49rem#fff, 27.3vw 62vh 0 -0.38rem#fff,
    19.4vw 50.9vh 0 -0.09rem#fff, 51.7vw 35.3vh 0 -0.23rem#fff;
  animation-duration: 10s;
}
.layer3.a {
  animation-delay: -5s;
}
@keyframes fall {
  100% {
    transform: translateY(200vh);
  }
}
</style>
