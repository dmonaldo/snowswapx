<template>
  <div id="stake-options">
    <div class="window white header">
      <img :src="publicPath + pool.img" class="header__img" />

      <div class="header__content">
        <h3 class="header__title">{{ pool.name }}</h3>
        <span class="header__subtitle">
          <a
            :href="link"
            rel="noopener noreferrer"
            target="_blank"
            class="clickable"
            :id="clickableHover ? 'clickableHover' : null"
          >
            Deposit
            <span v-if="pool.pair">
              <span class="header__subtitle--accent">{{ pool.pair }}</span>
              UNI-V2 LP
            </span>
          </a>
          and earn SNOW
        </span>
      </div>
    </div>

    <div class="window white">
      <div class="showings">
        <div class="showings__block">
          <span class="label">APY:</span>
          <span class="value">
            {{ pool.apy ? pool.apy.apy.toFixed(1) + " %" : "🧐" }}
          </span>

          <img :src="publicPath + 'info-icon.svg'" class="info-icon" alt="" />

          <div class="s-tooltip">
            <span v-if="pool.apy" class="s-tooltip__text">
              APY is estimated.
              <br />
              • There are
              <span id="grayText">
                {{ pool.apy.data.totalStaked.dividedBy(1e18).toFixed(1) }}
              </span>
              staked UNI-V2 token with price
              <span id="grayText">
                {{ pool.apy.data.UniPriceInETH.toFixed(3) }}
              </span>
              Eth
              <br />
              • There are currently
              <span id="grayText">5600</span>
              locked snow token with price
              <span id="grayText">
                {{ pool.apy.data.snowPriceETH.toFixed(3) }}
              </span>
              Eth, for
              <span id="grayText">1 week</span>
              unstaking schedule.
              <br />
              • There is also a bonus which can go up to
              <span id="grayText">3x in 60 days</span>
            </span>
            <span v-else class="s-tooltip__text">Not calculated</span>
          </div>
        </div>

        <div class="showings__block">
          <span class="label">Reward Multiplier:</span>
          <span class="value">{{ rewardMultiplier }}x</span>

          <img :src="publicPath + 'info-icon.svg'" class="info-icon" alt="" />

          <div class="s-tooltip">
            <span class="s-tooltip__text">
              Deposit liquidity tokens for 60 days to achieve a 3x reward
              multiplier.
            </span>
          </div>
        </div>

        <div class="showings__block">
          <span class="label">Accrued Rewards:</span>
          <span class="value">{{ accruedRewards.toFixed(6) }} ❄️</span>

          <img :src="publicPath + 'info-icon.svg'" class="info-icon" alt="" />

          <div class="s-tooltip">
            <span class="s-tooltip__text">
              Total rewards you have accrued till date. Larger your deposit and
              for longer, higher the accrual rate. This thing is something we're
              working on, we're not sure it's working as expected.
            </span>
          </div>
        </div>
      </div>

      <div
        :class="{ toggle: true, 'toggle--withdraw': !isDeposit }"
        @click="toggleMode"
      >
        <div class="toggle__label-wrapper">
          <span class="toggle__label">Deposit</span>
        </div>

        <div class="toggle__label-wrapper">
          <span class="toggle__label">Withdraw</span>
        </div>

        <div class="toggle__switch">
          <span class="toggle__label toggle__label--active" v-show="isDeposit">
            Deposit
          </span>
          <span class="toggle__label toggle__label--active" v-show="!isDeposit">
            Withdraw
          </span>
        </div>
      </div>

      <div class="input-block">
        <div class="input-wrapper">
          <input
            type="text"
            placeholder="Amount"
            v-model="displayAmount"
            v-bind:id="validId"
          />
          <span class="s-max-btn" @click="onMax">Max</span>
          <div>
            <label>
              max amount to {{ isDeposit ? "deposit" : "withdraw" }} :
              {{ maxAmount.dividedBy(1e18) }}
            </label>
          </div>
        </div>
        <div v-show="isDeposit" class="input-block__showings">
          <span class="label">Your estimated rewards</span>
          <span class="value">
            {{ estRewards }}
            <span>❄️/ Month</span>
          </span>
        </div>

        <div v-show="!isDeposit" class="input-block__showings">
          <!-- <div>
            <span class="label">Amount to Withdraw:</span>
            &nbsp;
            <span class="value">{{ amountToWithdraw }}$</span>
          </div> -->

          <div>
            <span class="label">Rewards Claimed:</span>
            &nbsp;
            <span class="value">{{ rewardsClaimed.toFixed(6) }}❄️</span>
          </div>
        </div>
      </div>
      <input
        id="inf-approval"
        type="checkbox"
        name="inf-approval"
        checked
        v-model="inf_approval"
      />
      <label for="inf-approval">
        Infinite approval - trust this contract forever
      </label>
      <button class="s-button s-button--primary submit-btn" @click="onSubmit">
        <span v-show="isDeposit">Deposit</span>
        <span v-show="!isDeposit">Withdraw</span>
      </button>
    </div>
  </div>
</template>

<script>
import * as common from "../../utils/common.js";
import { contract as contractstate } from "../../contract";
import allabis, {
  ycrvsnow_snow_unilp,
  ysnow_snow_unilp,
  snow_address,
  ERC20_abi,
  snow_eth_unilp,
} from "../../allabis";
import * as errorStore from "../../components/common/errorStore";
import { notify, notifyHandler } from "../../init";
import BN from "bignumber.js";
BN.config({ ROUNDING_MODE: BN.ROUND_DOWN });
BN.config({ EXPONENTIAL_AT: 100 });
export default {
  props: ["pool"],
  data: () => ({
    displayAmount: 0,
    BNamount: BN(0),
    isDeposit: true,
    rewardMultiplier: 3,
    accruedRewards: 0,
    estRewards: 0,
    amountToWithdraw: 0,
    rewardsClaimed: 0,
    maxAmount: BN(0),
    link: null,
    inf_approval: false,
    clickableHover: false,
    validId: "redBG",
  }),
  computed: {
    publicPath() {
      return process.env.BASE_URL;
    },
  },
  created: async function () {
    console.log(this.pool);
    //maxAmount
    if (this.pool.pair === "ySNOW-SNOW") {
      this.link =
        "https://info.uniswap.org/pair/0x4a80683e4dad35581f3c528851b647c77191bf78";
    }
    if (this.pool.pair === "ycrvSNOW-SNOW") {
      this.link =
        "https://info.uniswap.org/pair/0xb43a0beca632c286cd1d5748a749561c49b02b08";
    }
    if (this.pool.pair === "SNOW - ETH") {
      this.link =
        "https://info.uniswap.org/pair/0xE4f8F3CB9b33247789e4984A457bb69A1a621Df3";
    }
    const delay = (ms) => new Promise((res) => setTimeout(res, ms));

    let balance = await this.getMax(true);
    console.log(balance);
    this.maxAmount = BN(balance);
    let clc = this.clickableHover;
    //blinking text
    for (let x = 0; x < 5; x++) {
      this.clickableHover = !clc;
      await delay(500);
      this.clickableHover = clc;
      await delay(500);
    }
    this.accruedRewards = await this.getRewards(0, true);
    this.rewardsClaimed = this.accruedRewards;

    //accruedRewards:
  },
  methods: {
    amountCheck() {
      if (this.displayAmount[this.displayAmount.length - 1] === ".") {
        this.validId = "redBG";
        return;
      }
      let id =
        this.displayAmount <= 0 || this.maxAmount.lt(this.BNamount)
          ? "redBG"
          : "normalBG";
      this.validId = id;
    },
    async getRewards(balance_, isAccrual) {
      let tokenContract = await this.getGeyserTokenContract();
      let balance =
        balance_ > 0
          ? balance_
          : await tokenContract.methods
              .totalStakedFor(contractstate.default_account)
              .call();
      let totalBalance = await tokenContract.methods.totalStaked().call();
      let totalLockedShares = await tokenContract.methods
        .totalLockedShares()
        .call();
      let totalStakingShares = await tokenContract.methods
        .totalStakingShares()
        .call();
      let totalUnlocked = await tokenContract.methods.totalUnlocked().call();
      let totalLocked = await tokenContract.methods.totalLocked().call();

      // Figure out how much of the unlock schedule has been completed
      let unlockSchedules = await tokenContract.methods
        .unlockSchedules(0)
        .call();
      const jsTimestampToEthTimestamp = (jsTime, ethTime) => {
        let jsTimeStr = jsTime + "",
          ethTimeStr = ethTime + "",
          extraZeros = jsTimeStr.length - ethTimeStr.length;
        return Math.round(jsTime / 10 ** extraZeros);
      };
      let endAtSecUnixTimestamp = parseInt(unlockSchedules[3]);
      let durationSec = parseInt(unlockSchedules[4]);
      let startAtSec = endAtSecUnixTimestamp - durationSec;
      let curTime = jsTimestampToEthTimestamp(
        Date.now(),
        endAtSecUnixTimestamp
      );
      let timeElapsedSec = curTime - startAtSec;
      let pctElapsed = timeElapsedSec / durationSec;

      let shareConst = 1e18;
      let weekToMonthlyRewardsMultiplier = 2.5;
      let rewardProposed =
        (totalLocked * weekToMonthlyRewardsMultiplier) / shareConst;
      if (isAccrual) rewardProposed = (totalUnlocked * pctElapsed) / shareConst;
      let result = Math.min(
        (balance / totalBalance) * rewardProposed,
        rewardProposed
      );

      // console.log(balance, result, totalBalance, totalStakingShares, totalLockedShares, rewardProposed, totalLocked)
      return result;
    },
    async getGeyserTokenContract() {
      let tokenContract = null;
      if (this.pool.pair === "ySNOW-SNOW") {
        tokenContract = contractstate.ySnowGeyser;
      }
      if (this.pool.pair === "ycrvSNOW-SNOW") {
        tokenContract = contractstate.ycrvSnowGeyser;
      }
      if (this.pool.pair === "SNOW - ETH") {
        tokenContract = contractstate.snowGeyser;
      }
      return tokenContract;
    },
    async getMax(isDeposit) {
      let token_address = "0x";
      if (this.isDeposit) {
        if (this.pool.pair === "ySNOW-SNOW") {
          token_address = ysnow_snow_unilp;
        }
        if (this.pool.pair === "ycrvSNOW-SNOW") {
          token_address = ycrvsnow_snow_unilp;
        }
        if (this.pool.pair === "SNOW - ETH") {
          token_address = snow_eth_unilp;
        }

        const tokenContract = new contractstate.web3.eth.Contract(
          ERC20_abi,
          token_address
        );
        let balance = await tokenContract.methods
          .balanceOf(contractstate.default_account)
          .call();
        let rewards = await this.getRewards(balance);
        this.estRewards = rewards < 0.01 ? "< 0.01" : rewards.toFixed(6);
        return balance;
      } else {
        let tokenContract = await this.getGeyserTokenContract();
        let balance = await tokenContract.methods
          .totalStakedFor(contractstate.default_account)
          .call();
        return balance;
      }
    },
    toggleMode() {
      this.isDeposit = !this.isDeposit;
    },
    async onSubmit() {
      try {
        // console.log(this.BNamount.toString(), this.maxAmount.toString());
        if (
          this.BNamount.lte(0) ||
          this.BNamount.gt(this.maxAmount) ||
          this.displayAmount[this.displayAmount.length - 1] === "."
        )
          return;
        // let amounts = BN(Math.floor(this.amount * 1e12) * 1e6);
        // let amounts = BN(this.BNamount);
        //get the allowances
        let tokenContract = "0x0000000000000000000000000000000000000000";
        let geyser = "0x0000000000000000000000000000000000000000";
        if (this.pool.pair === "ySNOW-SNOW") {
          tokenContract = new contractstate.web3.eth.Contract(
            ERC20_abi,
            ysnow_snow_unilp
          );
          geyser = contractstate.ySnowGeyser;
        }
        if (this.pool.pair === "ycrvSNOW-SNOW") {
          tokenContract = new contractstate.web3.eth.Contract(
            ERC20_abi,
            ycrvsnow_snow_unilp
          );
          geyser = contractstate.ycrvSnowGeyser;
        }
        if (this.pool.pair === "SNOW - ETH") {
          tokenContract = new contractstate.web3.eth.Contract(
            ERC20_abi,
            snow_eth_unilp
          ); // What coin are we spending?
          geyser = contractstate.snowGeyser;
        }
        // console.log(geyser, tokenContract);
        //might be same
        let default_account = contractstate.default_account;
        let coin = contractstate.votingToken; //doesn't matter I guess

        let tokenAllowence = await tokenContract.methods
          .allowance(default_account, geyser._address)
          .call();
        let newTokenAllowence = BN(tokenAllowence);
        if (newTokenAllowence.lt(this.BNamount) && this.isDeposit) {
          if (!this.inf_approval) {
            let myAmounts = this.BNamount.toString();
            console.log("1", myAmounts);
            await common.approve(
              tokenContract,
              myAmounts,
              contractstate.web3.utils.toChecksumAddress(default_account),
              geyser._address
            );
          } else {
            await common.approve(
              tokenContract,
              contractstate.max_allowance,
              contractstate.web3.utils.toChecksumAddress(default_account),
              geyser._address
            );
          }
          return;
        }
        // console.log(geyser, common);
        if (this.isDeposit) {
          //deposit
          let zero = contractstate.web3.utils.fromAscii(0);
          let myAmounts = this.BNamount.toString();
          console.log("2", myAmounts);
          geyser.methods
            .stake(myAmounts, zero)
            .send({
              from: contractstate.default_account,
            })
            .once("transactionHash", function (hash) {
              notifyHandler(hash);
              resolve(true);
            })
            .on("error", (err) => {
              errorStore.handleError(err);
              reject(err);
            })
            .catch((err) => {
              errorStore.handleError(err);
              reject(err);
            });
        } else {
          //withdraw
          let zero = contractstate.web3.utils.fromAscii(0);
          let myAmounts = this.BNamount.toString();
          console.log(3, myAmounts);
          geyser.methods
            .unstake(myAmounts, zero)
            .send({
              from: contractstate.default_account,
            })
            .once("transactionHash", function (hash) {
              notifyHandler(hash);
              resolve(true);
            })
            .on("error", (err) => {
              errorStore.handleError(err);
              reject(err);
            })
            .catch((err) => {
              errorStore.handleError(err);
              reject(err);
            });
        }
      } catch (error) {
        console.log(error);
      }
    },
    onMax() {
      // logic to maximize this.amount
      this.BNamount = BN(this.maxAmount);
      this.displayAmount = this.maxAmount.dividedBy(1e18).toString();
    },
  },
  watch: {
    displayAmount: function (newValue) {
      if (
        newValue[newValue.length - 1] === "." &&
        newValue[newValue.length - 2] !== "."
      ) {
        this.displayAmount = newValue;
        this.BNamount = BN(0);
        this.amountCheck();
        return;
      }
      if (isNaN(newValue)) {
        this.displayAmount = this.BNamount.dividedBy(1e18).toString();
        return;
      }
      if (!newValue) {
        this.displayAmount = 0;
      } else {
        this.displayAmount = newValue;
      }
      this.BNamount = BN(this.displayAmount).multipliedBy(1e18);
      this.displayAmount = this.BNamount.dividedBy(1e18).toString();
      this.amountCheck();
    },
    isDeposit: async function (newValue) {
      this.displayAmount = 0;
      this.BNamount = BN(0);
      let maxAmount = await this.getMax(newValue);
      this.maxAmount = BN(maxAmount);
    },
    maxAmount: function (value) {
      console.log(value);
    },
  },
};
</script>

<style scoped>
*,
*::before,
*::after {
  font-family: BrandonGrotesque, sans-serif;
  box-sizing: border-box;
}
.window {
  overflow: visible;
}
.header {
  display: flex;
  align-items: center;
}
.header__img {
  width: 110px;
  height: 110px;
}
.header__content {
  margin-left: 26px;
}
.header__title {
  margin: 0;
  font-weight: 700;
  font-size: 26px;
  line-height: 26px;
  letter-spacing: -0.4px;
  color: #414a5b;
}
.header__subtitle {
  display: block;
  margin-top: 20px;
  font-weight: 400;
  font-size: 18px;
  line-height: 18px;
  letter-spacing: -0.2px;
  color: #414a5b;
}
.header__subtitle--accent {
  font-weight: 500;
  color: #00b9c2;
}
.showings {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-gap: 12px;
}
.showings__block {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  padding: 16px;
  border: 1px solid rgba(217, 223, 234, 0.5);
  border-radius: 16px;
}
.info-icon {
  position: absolute;
  top: 12px;
  right: 12px;
  cursor: pointer;
}
.info-icon:hover + .s-tooltip {
  opacity: 1;
}
.s-tooltip {
  position: absolute;
  bottom: 100%;
  right: 0;
  width: 260px;
  padding: 16px;
  background: #00b9c2;
  box-shadow: 0 8px 30px rgba(0, 185, 194, 0.3);
  border-radius: 10px;
  z-index: 1;
  transform: translateX(112px);
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.1s;
}
.s-tooltip__text {
  position: relative;
  font-weight: 500;
  font-size: 16px;
  line-height: 22px;
  color: #ffffff;
  z-index: 1;
}
.s-tooltip::before {
  content: "";
  display: block;
  position: absolute;
  bottom: -5px;
  left: calc(50% - 15px);
  width: 30px;
  height: 30px;
  border-radius: 7px;
  background: #00b9c2;
  box-shadow: 0 8px 30px rgba(0, 185, 194, 0.3);
  transform: rotateZ(45deg);
  z-index: 0;
}
.label {
  font-weight: 500;
  font-size: 18px;
  line-height: 18px;
  letter-spacing: -0.2px;
  color: #838b9d;
}
.value {
  font-family: RobotoMono, monospace;
  font-weight: 500;
  font-size: 16px;
  line-height: 26px;
  color: #414a5b;
}
.toggle {
  position: relative;
  display: flex;
  align-items: stretch;
  justify-content: space-between;
  width: 100%;
  height: 68px;
  margin-top: 12px;
  padding: 8px;
  background: #f5f8fa;
  border-radius: 14px;
  cursor: pointer;
}
.toggle--withdraw .toggle__switch {
  left: 50%;
}
.toggle__label-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: calc(50% - 8px);
}
.toggle__label {
  font-weight: 700;
  font-size: 16px;
  line-height: 12px;
  text-align: center;
  letter-spacing: 0.2px;
  text-transform: uppercase;
  color: #414a5b;
}
.toggle__label--active {
  color: #00b9c2;
}
.toggle__switch {
  position: absolute;
  left: 8px;
  top: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  width: calc(50% - 8px);
  height: calc(100% - 16px);
  box-shadow: 0 8px 20px rgba(225, 230, 236, 0.8);
  border-radius: 16px;
  background: #ffffff;
  transition: left, 0.3s;
}
.input-block {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 16px;
  margin-top: 26px;
  padding: 16px;
  border: 1px solid rgba(217, 223, 234, 0.5);
  border-radius: 16px;
}
.input-wrapper {
  position: relative;
  flex-grow: 1;
}
.input-block__showings {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  text-align: center;
  justify-content: center;
}
.input-block__showings .value {
  line-height: 16px;
  padding: 7px;
}
.submit-btn {
  width: 100%;
  max-width: 280px;
  margin: 26px auto 0 auto;
}
#redBG {
  background-color: rgba(228, 92, 137, 0.3);
  transition: 0.3s ease-in-out;
}
#normalBG {
  transition: 0.3s ease-in-out;
}
.clickable {
  font-weight: 900;
  border: 1px solid;
  border-color: white;
  text-decoration: none;
  transition: 0.3s ease-in-out;
}
.clickable:hover {
  border-color: #00b9c2;
  text-decoration: none;
  transition: 0.3s ease-in-out;
}
#clickableHover {
  transform: scale(0.9);
  border-color: #00b9c2;
  transition: 0.3s ease-in-out;
}
a {
  color: #414a5b;
  text-decoration: none;
}
#grayText {
  color: rgb(94, 94, 94);
}
</style>
