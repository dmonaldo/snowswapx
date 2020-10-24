<template>
  <div id="select-pool">
    <div class="tint"></div>

    <div id="pools-list" v-if="pools && pools.length">
      <div class="pools-list__content">
        <h2>Earn SNOW</h2>

        <p class="subheading">
          by supplying your LP tokens and SNOW to Uniswap and staking UNI V2 LP
          tokens
        </p>

        <ul class="pools">
          <li v-for="pool in pools.filter((p) => p.isSnowLp)" :key="pool.name">
            <pool-card :pool="pool" :on-select-pool="onSelectPool" />
          </li>
        </ul>
      </div>

      <div class="promo">
        <img :src="publicPath + 'jack-frost-champion.svg'" />

        <div>
          <h4>Boost Your Rewards</h4>

          <p>
            Enjoy up to 3x boost in rewards for staking your tokens for longer
          </p>
        </div>
      </div>

      <div class="pools-list__content">
        <h2>Earn even more SNOW</h2>

        <p class="subheading">
          by providing liquidity to SNOW-ETH Uniswap pool
        </p>

        <ul class="pools">
          <li v-for="pool in pools.filter((p) => !p.isSnowLp)" :key="pool.name">
            <pool-card
              :pool="pool"
              :on-select-pool="() => onSelectPool(pool)"
            />
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import PoolCard from "@/components/stake/PoolCard";

export default {
  components: {
    PoolCard,
  },
  props: ["pools", "onSelectPool"],
  computed: {
    publicPath() {
      return process.env.BASE_URL;
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
#select-pool {
  position: relative;
  padding-top: 50px;
}
.tint {
  position: absolute;
  top: -41px;
  left: 0;
  width: 100%;
  height: 35vw;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.8) 35.02%,
    #ffffff 59.26%,
    #ffffff 100%
  );
  z-index: 0;
}
#pools-list {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}
#pools-list > div {
  max-width: 100%;
}
#pools-list h2 {
  margin: 0 0 26px;
  font-style: normal;
  font-weight: bold;
  font-size: 36px;
  line-height: 32px;
  text-align: center;
  letter-spacing: -0.6px;
  color: #414a5b;
}
#pools-list .subheading {
  margin: 12px 0 36px 0;
  text-align: center;
  font-weight: 400;
  font-size: 24px;
  line-height: 24px;
  letter-spacing: -0.2px;
  color: #414a5b;
}
.pools-list__content {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.pools {
  display: grid;
  grid-auto-flow: column;
  grid-auto-columns: 1fr;
  grid-gap: 20px;
  justify-items: center;
}
.promo {
  display: flex;
  justify-content: center;
  align-items: center;
  align-self: stretch;
  margin: 56px 0;
  padding: 21px 13px;
  background: linear-gradient(91.69deg, #ecfcfb 19.53%, #e0f4ff 75.26%);
}
.promo > * {
  margin: 0 13px;
}
.promo h4 {
  margin: 0;
  font-style: normal;
  font-weight: bold;
  font-size: 26px;
  line-height: 32px;
  letter-spacing: -0.6px;
  color: #00b9c2;
}
.promo p {
  max-width: 400px;
  margin: 12px 0 0 0;
  font-style: normal;
  font-weight: normal;
  font-size: 24px;
  line-height: 26px;
  letter-spacing: -0.2px;
  color: #414a5b;
}
</style>
