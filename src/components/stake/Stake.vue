<template>
  <div id="stake">
    <select-pool
      v-if="!selectedPool"
      :pools="pools"
      :on-select-pool="onSelectPool"
    />
    <stake-options v-if="selectedPool" :pool="selectedPool" />
  </div>
</template>

<script>
import SelectPool from "@/components/stake/SelectPool";
import StakeOptions from "@/components/stake/StakeOptions";
import axios from "axios";
import { contract as contractstate } from "../../contract";
import allabis, {
  // ycrvsnow_snow_unilp,
  // ysnow_snow_unilp,
  snow_address,
  ERC20_abi_uni,
  snow_eth_unilp,
} from "../../allabis";
import BN from "bignumber.js";
BN.config({ ROUNDING_MODE: BN.ROUND_DOWN });
BN.config({ EXPONENTIAL_AT: 100 });
export default {
  components: {
    SelectPool,
    StakeOptions,
  },
  data: () => ({
    pools: [
      {
        name: "Jack Frost’s Pool",
        img: "pool-jack-frost.svg",
        pair: "ySNOW-SNOW",
        apy: null,
        isSnowLp: true,
      },
      {
        name: "Santa Claus’s Pool",
        pair: "ycrvSNOW-SNOW",
        img: "pool-santa-claus.svg",
        apy: null,
        isSnowLp: true,
      },
      {
        name: "Rudolph’s ETH Pool",
        pair: "SNOW - ETH",
        // stake: "Snow",
        img: "pool-rudolph.svg",
        apy: null,
      },
      // {
      //   name: "Dancer’s LINK Pool",
      //   pair: "SNOW - LINK",
      //   img: "pool-dancer.svg",
      //   apy: null,
      // },
    ],
    selectedPool: null,
  }),
  computed: {
    publicPath() {
      return process.env.BASE_URL;
    },
  },
  methods: {
    async getApySNOW(pool) {
      let snow = await axios.get(
        "https://api.coingecko.com/api/v3/coins/snowswap?localization=false&tickers=false&market_data=true&community_data=false&developer_data=false&sparkline=false"
      );
      let snowPriceETH = snow.data.market_data.current_price.eth;
      // get uni-v2 price in ETH
      const tokenContract = new contractstate.web3.eth.Contract(
        ERC20_abi_uni,
        snow_eth_unilp
      );
      let totalSupplyUni = BN(await tokenContract.methods.totalSupply().call());
      let reserves = await tokenContract.methods.getReserves().call();
      let cumulativeReservesInEth = BN(reserves["0"]).plus(
        BN(reserves["1"]).multipliedBy(snowPriceETH)
      );
      let UniPriceInETH = cumulativeReservesInEth.dividedBy(totalSupplyUni);
      // get total staked  snow-eth uni in address
      let totalStaked = await contractstate.snowGeyser.methods
        .totalStaked()
        .call();
      let totalStakedInETH = BN(BN(totalStaked).multipliedBy(UniPriceInETH));

      //get total locked amount in eth
      let TOTALLOCKEDFORWEEK = 5600 * 1e18;
      // let annualizedTOTALLOCKEDFORWEEK = 52 * TOTALLOCKEDFORWEEK;
      let totalLockedInETH = BN(TOTALLOCKEDFORWEEK).multipliedBy(snowPriceETH);

      // get total locked/staked to understand how much does a staker's apy in eth
      let apyWEEK = totalLockedInETH.dividedBy(totalStakedInETH);
      let apy = apyWEEK * (365 / 7);
      let bonusConstantFor60Days = (33 * 60 + (67 / 60) * ((60 * 61) / 2)) / 60; // 33 to 100(x3) in 60 day period bonus. then x3 contiuos. 33+34.1+...100+100+100...%
      // make it a percentage
      console.log(bonusConstantFor60Days);
      return {
        apy: ((apy * 100) / 33) * bonusConstantFor60Days,
        data: {
          totalSupplyUni,
          cumulativeReservesInEth,
          UniPriceInETH,
          totalStaked: BN(totalStaked),
          snowPriceETH,
          totalLockedInETH,
          bonusConstantFor60Days,
        },
      };
    },
    onSelectPool(pool) {
      this.selectedPool = pool;
    },
  },
  created: async function () {
    let snowETH = this.pools[2];
    snowETH.apy = "loading";
    this.pools[2] = snowETH;
    let apyData = await this.getApySNOW();
    snowETH.apy = apyData;
    this.pools[2] = snowETH;
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
#stake {
  position: relative;
  padding-top: 50px;
}
</style>
