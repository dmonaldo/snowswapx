<template>
  <div id="app">
    <router-view></router-view>
    <disclosure v-if="showDisclosure" @close="hideDisclosure"></disclosure>
  </div>
</template>

<script>
import { getters, contract as currentContract } from "./contract";
import { capitalizeFirstLetter } from "./utils/helpers";
import Disclosure from "@/components/common/Disclosure";

const descriptions = {
  compound: "A snowswap.org portal for swapping cDAI/cUSDC",
  usdt: "A snowswap.org Tethered portal for swapping cDAI/cUSDC/USDT",
  iearn: "A snowswap.org yTokens portal for swapping DAI/USDC/USDT/TUSD",
  busd: "A snowswap.org portal for swapping BUSD and other stablecoins",
};

const titles = {
  compound: "Compounded",
  usdt: "Tethered",
  iearn: "Yield",
  busd: "bUSD",
  susd: "sUSD-yCurve old",
  susdv2: "sUSD",
  pax: "PAX",
  tbtc: "TBTC",
  ren: "renBTC",
  sbtc: "sBTC",
};

export default {
  components: { Disclosure },
  metaInfo() {
    return {
      title: "snowswap.org",
      meta: [
        { property: "og:title", content: "snowswap.org/" + this.currentPool },
        {
          property: "og:url",
          content: "https://snowswap.org/" + this.currentPool,
        },
        { property: "og:type", content: "website" },
        { property: "og:description", content: descriptions[this.currentPool] },
        {
          property: "og:image",
          content: "/curve" + this.currentPool + "_preview.png",
        },
        { name: "twitter:card", content: "summary_large_image" },
        { name: "twitter:title", content: "snowswap.org/" + this.currentPool },
        { name: "twitter:site", content: "@snow_swap" },
        { name: "twitter:creator", content: "@snow_swap" },
        {
          name: "twitter:description",
          content: descriptions[this.currentPool],
        },
        {
          name: "twitter:url",
          content: "https://snowswap.org/" + this.currentPool,
        },
        {
          name: "twitter:image",
          content: this.currentPool + "/c",
        },
      ],
    };
  },
  computed: {
    ...getters,
  },
  data: () => ({
    showDisclosure: false,
  }),
  mounted() {
    if (localStorage.getItem("wasVisited") === null) {
      this.showDisclosure = true;
      document.body.style.overflow = "hidden";
    }
  },
  watch: {
    showDisclosure: function () {
      if (!this.showDisclosure) {
        document.body.style.overflow = "";
      }
    },
  },
  methods: {
    hideDisclosure: function () {
      localStorage.setItem("wasVisited", "true");
      this.showDisclosure = false;
    },
  },
};
</script>
