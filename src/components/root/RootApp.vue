<template>
  <div id="app" :class="{ root: true, [$route.name]: true }">
    <div class="screencontainer">
      <div class="top-menu-bar">
        <div class="top-menu-bar__content">
          <router-link :to="`/`">
            <img :src="logoSrc" />
          </router-link>

          <div class="top-menu-bar__menu">
            <label for="hamburger" class="border-menu"></label>
            <input type="checkbox" id="hamburger" />

            <div class="poolsdropdown">
              <button class="simplebutton">☰</button>
              <div class="dropdown">
                <!--  <a :href="'//compound.localhost:8080'+$route.path" :class="{selected: currentPool == 'compound'}" @click="changePools('compound')">Compound</a>
            <a :href="'//usdt.localhost:8080'+$route.path" :class="{selected: currentPool == 'usdt'}" @click="changePools('usdt')">USDT</a>
            <a :href="'//y.localhost:8080'+$route.path" :class="{selected: currentPool == 'iearn'}" @click="changePools('iearn')">Y</a>
            <a :href="'//busd.localhost:8080'+$route.path" :class="{selected: currentPool == 'busd'}" @click="changePools('busd')">bUSD</a> -->

                <!-- <router-link :to="'/compound/' + ($route.path.split('/')[2] || '')  ">Compound</router-link> -->
                <!-- <router-link :to="'/usdt/' + ($route.path.split('/')[2] || '')  ">USDT</router-link> -->
                <!-- <router-link :to="'/pax/' + ($route.path.split('/')[2] || '')">PAX</router-link> -->
                <router-link
                  :to="'/iearn/' + ($route.path.split('/')[2] || '')"
                >
                  yVault USD
                </router-link>
                <router-link :to="'/busd/' + ($route.path.split('/')[2] || '')">
                  yVault Curve
                </router-link>
                <!--                <router-link-->
                <!--                  :to="'/susdv2/' + ($route.path.split('/')[2] || '')"-->
                <!--                >-->
                <!--                  sUSD-->
                <!--                </router-link>-->
                <!-- <router-link :to="'/ren/' + ($route.path.split('/')[2] || '')">renBTC</router-link> -->
                <!-- <router-link :to="'/sbtc/' + ($route.path.split('/')[2] || '')">sBTC</router-link> -->
                <!-- <router-link :to="'/hbtc/' + ($route.path.split('/')[2] || '')">hBTC</router-link> -->
                <!--             <a href="https://iearn.finance/pool">sUSD</a> -->
                <!--            <p>____________</p>-->
                <button
                  id="changeAccounts"
                  class="s-button s-button--secondary s-button--small"
                  @click="changeAccounts"
                >
                  Change accounts
                </button>
                <button
                  class="s-button s-button--primary s-button--small"
                  @click="changeWallets"
                >
                  Change wallet
                </button>
              </div>
            </div>
            <router-link to="/">Home</router-link>
            <a href="https://dao.snowswap.org">DAO</a>
          </div>

          <div class="top-menu-bar__socials">
            <a
              class="s-btn-social"
              target="_blank"
              rel="noopener noreferrer"
              href="https://twitter.com/snow_swap"
            >
              <img :src="publicPath + 'twitter.svg'" alt="Twitter" />
            </a>

            <a
              class="s-btn-social"
              target="_blank"
              rel="noopener noreferrer"
              href="https://medium.com/snowswap"
            >
              <img :src="publicPath + 'medium.svg'" alt="Medium" />
            </a>

            <a
              class="s-btn-social"
              target="_blank"
              rel="noopener noreferrer"
              href="https://discord.gg/CSSJTs4"
            >
              <img :src="publicPath + 'discord.svg'" alt="Discord" />
            </a>

            <a
              class="s-btn-social"
              target="_blank"
              rel="noopener noreferrer"
              href="https://t.me/Snow_Swap"
            >
              <img :src="publicPath + 'telegram.svg'" alt="Telegram" />
            </a>
          </div>

          <!--      <router-link to='/earlyCRV'>Community CRV pre-allocation</router-link>-->
          <!--      <router-link to="/trade">Trade</router-link>-->
          <!--      <div class='poolsdropdown'>-->
          <!--        <router-link to="/combinedstats">Stats</router-link>-->
          <!--        <div class='dropdown'>-->
          <!--          <router-link to="/combinedstats">Stats</router-link>-->
          <!--          <router-link to="/dailystats">Daily stats</router-link>-->
          <!--          <router-link to="/volumepercoin">Coin volumes</router-link>-->
          <!--          <router-link to="/volumeperpair">Pair volumes</router-link>-->
          <!--          <router-link to="/totaldeposits">Total deposits</router-link>-->
          <!--        </div>-->
          <!--      </div>-->
          <!--      <router-link to="/curvepay">Pay</router-link>-->
          <!--      <router-link to="/risks">Risks</router-link>-->
          <!--      <div class='poolsdropdown right'>-->
          <!--        <span>?</span>-->
          <!--        <div class='dropdown'>-->
          <!--          <router-link to="/audits">Audits</router-link>-->
          <!--          <router-link to="/events">Events</router-link>-->
          <!--          <router-link to="/contracts">Contracts</router-link>-->
          <!--          <router-link to="/bugbounty">Bug Bounty</router-link>-->
          <!--          <router-link to="/rootfaq">FAQ</router-link>-->
          <!--          <router-link to="/integrations">Integrations</router-link>-->
          <!--          <router-link to="/donate">Donate</router-link>-->
          <!--          <router-link to="/devdocs">Developer Docs</router-link>-->
          <!--          <a href='https://guides.snowswap.org' rel='noopener noreferrer'>Guides</a>-->
          <!--          <a href='https://resources.snowswap.org' rel='noopener noreferrer'>Resources</a>-->
          <!--          <p>____________</p>-->
          <!--          <a href="https://twitter.com/CurveFinance" rel='noopener noreferrer'>#Twitter</a>-->
          <!--          <a href="https://t.me/curvefi" rel='noopener noreferrer'>@Telegram</a>-->
          <!--          <a href="https://t.me/curveficn" rel='noopener noreferrer'>@Telegram CN</a>-->
          <!--          <a href="https://discord.gg/CSSJTs4" rel='noopener noreferrer'>@Discord</a>-->
          <!--          <a href="https://explore.duneanalytics.com/public/dashboards/RTH47mNjQcoLv5oG0HMDdI0iDq7BHxk1PzCRdwQB" rel='noopener noreferrer'>Dune Analytics</a>-->
          <!--          <p>____________</p>-->
          <!--          <a href="https://github.com/curvefi/curve-contract">git@</a>-->
          <!--          <a href="https://github.com/curvefi/curve-vue">git@UI</a>-->
          <!--        </div>-->
          <!--      </div>-->
          <!--                <a href="https://dao.snowswap.org" class='showmobile'>DAO</a>-->
          <!--      <router-link to="/combinedstats" class='showmobile'>Stats</router-link>-->
          <!--      <router-link to="/dailystats" class='showmobile'>Daily stats</router-link>-->
          <!--      <router-link to="/volumepercoin" class='showmobile'>Coin volumes</router-link>-->
          <!--      <router-link to="/volumeperpair" class='showmobile'>Pair volumes</router-link>-->
          <!--      <router-link to="/totaldeposits" class='showmobile'>Total deposits</router-link>-->
          <!--      <router-link to="/audits" class='showmobile'>Audits</router-link>-->
          <!--      <router-link to="/events" class='showmobile'>Events</router-link>-->
          <!--      <router-link to="/bugbounty" class='showmobile'>Bug Bounty</router-link>-->
          <!--      <router-link to="/rootfaq" class='showmobile'>FAQ</router-link>-->
          <!--      <router-link to="/integrations" class='showmobile'>Integrations</router-link>-->
          <!--      <router-link to="/donate" class='showmobile'>Donate</router-link>-->
          <!--      <router-link to="/devdocs" class='showmobile'>Developer Docs</router-link>-->
          <!--      <a href='https://guides.snowswap.org' rel='noopener noreferrer' class='showmobile'>Guides</a>-->
          <!--      <a href='https://resources.snowswap.org' rel='noopener noreferrer' class='showmobile'>Resources</a>-->
          <!--      <a href="https://twitter.com/CurveFinance" class='showmobile' rel='noopener noreferrer'>#Twitter</a>-->
          <!--      <a href="https://t.me/curvefi" class='showmobile' rel='noopener noreferrer'>@Telegram</a>-->
          <!--      <a href="https://discord.gg/CSSJTs4" class='showmobile' rel='noopener noreferrer'>@Discord</a>-->
          <!--      <a href="https://explore.duneanalytics.com/public/dashboards/RTH47mNjQcoLv5oG0HMDdI0iDq7BHxk1PzCRdwQB" class='showmobile' rel='noopener noreferrer'>Dune Analytics</a>-->
          <!--      <a href="https://github.com/curvefi/curve-contract" class='showmobile'>git@</a>-->
          <!--      <a href="https://github.com/curvefi/curve-vue" class='showmobile'>git@UI</a>-->
          <button class="simplebutton showmobile" @click="changeWallets">
            Change wallet
          </button>
          <button
            id="changeAccounts"
            class="simplebutton showmobile"
            v-show="['ledger', 'trezor'].includes(walletName)"
            @click="changeAccounts"
          >
            Change accounts
          </button>
        </div>
      </div>
      <div id="screen">
        <!--        <div :class="'blue window ' + $route.name">-->
        <!--            <h1><img :src="logoSrc" alt="🌀 Curve"></h1>-->
        <!--        </div>-->

        <img class="logo" :src="logoSrc" />
        <div class="window white connected-wallet" v-if="hasConnectedWallet">
          <h3 class="window__heading">Connected wallet</h3>
          <div class="connected-wallet__content">
            <img :src="publicPath + 'wallets.png'" />
            <span class="connected-wallet__text">
              You haven't connected a wallet.
            </span>
            <button @click="changeWallets" class="s-button s-button--primary">
              Connect wallet
            </button>
          </div>
        </div>
        <div class="simple-error window" v-show="plsReturn">
          Your recent withdrawal from Curve resulted in getting 1000 more USDT
          because of another user mistakenly transferring funds to the contract.
          If you wish to return them - please contact us on
          <a href="https://twitter.com/CurveFinance">Twitter</a>
          /
          <a href="https://t.me/curvefi">Telegram</a>
          /
          <a href="https://discord.gg/CSSJTs4" rel="noopener noreferrer">
            @Discord
          </a>
          . Thank you!
        </div>
        <router-view />
      </div>
    </div>
    <footer class="footer">
      <!--      <a href="https://twitter.com/CurveFinance" rel="noopener noreferrer">-->
      <!--        #Twitter-->
      <!--      </a>-->
      <!--      <a href="https://t.me/curvefi" rel="noopener noreferrer">@Telegram</a>-->
      <!--      <a href="https://t.me/curveficn" rel="noopener noreferrer">-->
      <!--        @Telegram CN-->
      <!--      </a>-->
      <!--      <a href="https://discord.gg/CSSJTs4" rel="noopener noreferrer">-->
      <!--        @Discord-->
      <!--      </a>-->
      <!--      <a href="https://guides.snowswap.org" rel="noopener noreferrer">Guides</a>-->
      <!--      <a href="https://resources.snowswap.org" rel="noopener noreferrer">-->
      <!--        Resources-->
      <!--      </a>-->
      <!--      <a-->
      <!--        href="https://explore.duneanalytics.com/public/dashboards/RTH47mNjQcoLv5oG0HMDdI0iDq7BHxk1PzCRdwQB"-->
      <!--        rel="noopener noreferrer"-->
      <!--      >-->
      <!--        Dune Analytics-->
      <!--      </a>-->
      <!--      <a href="https://github.com/curvefi/curve-contract">git@</a>-->
      <!--      <a href="https://github.com/curvefi/curve-vue">git@UI</a>-->

      <div class="footer__center">
        <router-link :to="`/`">
          <img :src="publicPath + 'logo_snowball_white.svg'" />
        </router-link>

        <div class="footer__content">
          <h4>Beta Tester User Agreement</h4>

          <p>
            Please note that this is a Beta version of the SnowSwap DEX. The
            platform, its software and all content found on it are provided on
            an “as is” and “as available” basis.
            <br />
            <br />
            While we have made reasonable efforts to ensure the security and
            functionality of the SnowSwap platform including forking much of the
            codebase from existing well-audited projects, nothing approaching
            the rigor of a formal audit has been conducted at this time and the
            front-end is actively being tested.
            <br />
            <br />
            We strongly advise caution to anyone who chooses to use the early
            experimental version.
            <br />
            <br />
            PLEASE DO NOT RISK ANY FUNDS YOU CANNOT AFFORD TO LOSE.
          </p>

          <!--          <a class="footer__show-all-url">Show all</a>-->
        </div>
      </div>
      <!--      <router-link to="/devdocs">Developer Docs</router-link>-->
    </footer>
  </div>
</template>

<script>
import {
  getters,
  contract as currentContract,
  changeContract,
  poolMenu,
} from "../../contract";
import init, { onboard, changeWallets } from "../../init";

export default {
  metaInfo: {
    title: "SnowSwap",
    meta: [
      { property: "og:title", content: "snowswap.org" },
      { property: "og:url", content: "https://snowswap.org" },
      { property: "og:type", content: "website" },
      {
        property: "og:description",
        content:
          "SnowSwap is an exchange liquidity pool on Ethereum designed for extremely efficient stablecoin trading",
      },
      { property: "og:image", content: "/curve.png" },
      { name: "twitter:card", content: "summary_large_image" },
      { name: "twitter:title", content: "snowswap.org" },
      { name: "twitter:site", content: "@snow_swap" },
      { name: "twitter:creator", content: "@snow_swap" },
      {
        name: "twitter:description",
        content:
          "Curve is an exchange liquidity pool on Ethereum designed for extremely efficient stablecoin trading",
      },
      { name: "twitter:url", content: "https://snowswap.org" },
      { name: "twitter:image", content: "/curve.png" },
    ],
  },
  components: {},
  computed: {
    ...getters,
    poolMenu() {
      return poolMenu;
    },
    publicPath() {
      return process.env.BASE_URL;
    },
    logoSrc() {
      return this.publicPath + "logo_snowball.svg";
    },
    hasConnectedWallet() {
      return (
        this.default_account == "0x0000000000000000000000000000000000000000" &&
        ![
          "Donate",
          "StatsDaily",
          "Audits",
          "Stats",
          "Contracts",
          "FAQ",
          "RootFAQ",
        ].includes(this.$route.name)
      );
    },
    plsReturn() {
      return (
        currentContract.currentContract.toLowerCase() ==
        "0x72c20f89008729c91b6bb85f3104fda942494cef".toLowerCase()
      );
    },
  },
  methods: {
    changePools(pool) {
      changeContract(pool);
    },
    async changeWallets() {
      changeWallets();
    },
    async changeAccounts() {
      return onboard.accountSelect();
    },
  },
};
</script>

<style scoped>
#changeAccounts {
  margin-top: 0.3em;
}
@media only screen and (min-device-width: 320px) and (max-device-width: 730px) {
  .top-menu-bar > .poolsdropdown {
    display: none;
  }
  .blue.window.half-width,
  .info-message.window.half-width {
    width: 90%;
  }
}
h1 > img {
  height: 52.125px;
}
.simple-error.window {
  box-shadow: none;
}
.blue.window.Events + .info-message.gentle-message {
  width: 730px;
  max-width: 730px;
}
.blue.window.Trade + .info-message.gentle-message {
  width: 1000px;
  max-width: 1000px;
}
.CRV a:hover,
.CRV a:visited {
  color: white;
}
.dropdown {
}
.dropdown > .s-button + .s-button {
  margin-top: 6px;
}
</style>
