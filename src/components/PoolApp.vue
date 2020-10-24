<template>
  <div id="app">
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
              <button class="simplebutton">{{ poolMenu[currentPool] }}</button>
              <div class="dropdown">
                <!--  <a :href="'//compound.localhost:8080'+$route.path" :class="{selected: currentPool == 'compound'}" @click="changePools('compound')">Compound</a>
            <a :href="'//usdt.localhost:8080'+$route.path" :class="{selected: currentPool == 'usdt'}" @click="changePools('usdt')">USDT</a>
            <a :href="'//y.localhost:8080'+$route.path" :class="{selected: currentPool == 'iearn'}" @click="changePools('iearn')">Y</a>
            <a :href="'//busd.localhost:8080'+$route.path" :class="{selected: currentPool == 'busd'}" @click="changePools('busd')">bUSD</a> -->

                <!-- <router-link :to="'/compound/' + ($route.path.split('/')[2] || '')  " :class="{selected: currentPool == 'compound'}">Compound</router-link>-->
                <!-- <router-link :to="'/usdt/' + ($route.path.split('/')[2] || '')  " :class="{selected: currentPool == 'usdt'}">USDT</router-link> -->
                <!-- <router-link :to="'/pax/' + ($route.path.split('/')[2] || '') " :class="{selected: currentPool == 'pax'}">PAX</router-link>-->
                <router-link
                  :to="'/iearn/' + ($route.path.split('/')[2] || '')"
                  :class="{ selected: currentPool == 'iearn' }"
                >
                  yVault USD
                </router-link>
                <router-link
                  :to="'/busd/' + ($route.path.split('/')[2] || '')"
                  :class="{ selected: currentPool == 'busd' }"
                >
                  yVault Curve
                </router-link>
                <!--                <router-link-->
                <!--                  :to="'/susdv2/' + ($route.path.split('/')[2] || '')"-->
                <!--                  :class="{ selected: currentPool == 'susdv2' }"-->
                <!--                >-->
                <!--                  sUSD-->
                <!--                </router-link>-->
                <!-- <router-link :to="'/ren/' + ($route.path.split('/')[2] || '')">renBTC</router-link>-->
                <!-- <router-link :to="'/sbtc/' + ($route.path.split('/')[2] || '')">sBTC</router-link>-->
                <!-- <router-link :to="'/hbtc/' + ($route.path.split('/')[2] || '')">HBTC</router-link> -->
                <!-- <a href="https://iearn.finance/pool">sUSD</a> -->
                <!--            <p>____________</p>-->
                <!--            <router-link to='/'>Home</router-link>-->
                <!--            <router-link to='/trade'>Trade</router-link>-->
                <!--            <router-link to='/combinedstats'>All stats</router-link>-->
                <!--            <router-link to='/dailystats'>Daily stats</router-link>-->
                <!--            <router-link to='/volumepercoin'>Coin volumes</router-link>-->
                <!--            <a href="https://twitter.com/CurveFinance">#Twitter</a>-->
                <!--            <a href="https://t.me/curvefi">@Telegram</a>-->
                <!--            <a href="https://t.me/curveficn">@Telegram CN</a>-->
                <!--            <a href="https://discord.gg/CSSJTs4" rel='noopener noreferrer'>@Discord</a>-->
                <!--            <a href="https://explore.duneanalytics.com/public/dashboards/RTH47mNjQcoLv5oG0HMDdI0iDq7BHxk1PzCRdwQB">Dune Analytics</a>-->
                <!--            <p>____________</p>-->
                <button
                  class="s-button s-button--primary s-button--small"
                  @click="changeWallets"
                >
                  Change wallet
                </button>
                <button
                  id="changeAccounts"
                  class="simplebutton"
                  v-show="['ledger', 'trezor'].includes(walletName)"
                  @click="changeAccounts"
                >
                  Change accounts
                </button>
              </div>
            </div>

            <!-- <router-link to="/">Root</router-link> -->
            <a href="https://dao.snowswap.org">DAO</a>
            <!--      <router-link to='/combinedstats' class='showmobile'>All stats</router-link>-->
            <router-link :to="'/' + currentPool" v-show="currentPool != 'susd'">
              Buy and sell
            </router-link>
            <router-link
              :to="'/' + currentPool + '/deposit'"
              v-show="currentPool != 'susd'"
            >
              Deposit
            </router-link>
            <router-link :to="'/' + currentPool + '/withdraw'">
              Withdraw
            </router-link>
            <router-link :to="'/stake'">
              Stake
            </router-link>
            <!--      <router-link :to="'/' + currentPool + '/withdraw_old'" v-show="currentPool == 'compound' && oldBalance > 0">Withdraw old</router-link>-->
            <!--      <router-link to="/susd/withdraw" v-show="currentPool == 'susdv2' && oldBalance > 0">Withdraw old</router-link>-->
            <!--      <router-link :to="'/' + currentPool + '/stats'" v-show="currentPool !='susd'">Stats</router-link>-->
            <!--      <router-link :to="'/' + currentPool + '/profit'" v-show="currentPool !='susd'">Profit</router-link>-->
            <!--      <router-link :to="'/curvepay/' + currentPool">Pay</router-link>-->
            <!--      <div class='poolsdropdown right'>-->
            <!--        <span>?</span>-->
            <!--        <div class='dropdown'>-->
            <!--          <a :href="'https://etherscan.io/address/' + this.poolAddress" rel='noopener noreferrer'>Pool contract</a>-->
            <!--          <a :href="'https://etherscan.io/address/' + this.tokenAddress" rel='noopener noreferrer'>Token contract</a>-->
            <!--          <p>____________</p>-->
            <!--          <router-link to="/audits">Audits</router-link>-->
            <!--          <router-link to="/events">Events</router-link>-->
            <!--          <router-link :to="'/' + currentPool + '/risks'">Risks</router-link>-->
            <!--          <router-link to="/bugbounty">Bug Bounty</router-link>-->
            <!--          <router-link :to="'/' + currentPool + '/faq'">FAQ</router-link>-->
            <!--          <router-link to="/integrations">Integrations</router-link>-->
            <!--          <router-link :to="'/' + currentPool + '/donate'">Donate</router-link>-->
            <!--          <router-link to="/devdocs">Developer Docs</router-link>-->
            <!--          <a href='https://guides.snowswap.org' rel='noopener noreferrer'>Guides</a>-->
            <!--          <a href='https://resources.snowswap.org' rel='noopener noreferrer'>Resources</a>-->
            <!--          <p>____________</p>-->
            <!--          <a :href="'https://github.com/curvefi/curve-contract/tree/pool_'+gitBranches[currentPool]" rel='noopener noreferrer'>git@</a>-->
            <!--          <a href="https://github.com/curvefi/curve-vue" rel='noopener noreferrer'>git@UI</a>-->
            <!--        </div>-->
            <!--      </div>-->
            <!--                  <a href="https://dao.snowswap.org" class='showmobile'>DAO</a>-->
            <!--      <router-link to="/audits" class='showmobile'>Audits</router-link>-->
            <!--      <router-link to="/events" class='showmobile'>Events</router-link>-->
            <!--      <router-link :to="'/' + currentPool + '/faq'" class='showmobile'>FAQ</router-link>-->
            <!--      <router-link to="/integrations" class='showmobile'>Integrations</router-link>-->
            <!--      <router-link to="/bugbounty" class='showmobile'>Bug Bounty</router-link>-->
            <!--      <router-link :to="'/' + currentPool + '/donate'" class='showmobile'>Donate</router-link>-->
            <!--      <a href='https://guides.snowswap.org' rel='noopener noreferrer' class='showmobile'>Guides</a>-->
            <!--      <a href='https://resources.snowswap.org' rel='noopener noreferrer' class='showmobile'>Resources</a>-->
            <!--      <a :href="'https://github.com/curvefi/curve-contract/tree/pool_'+gitBranches[currentPool]" class='showmobile' rel='noopener noreferrer'>git@</a>-->
            <!--      <a href="https://github.com/curvefi/curve-vue" class='showmobile' rel='noopener noreferrer'>git@UI</a>-->
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

          <div class="top-menu-bar__right">
            <div class="uniPools" @click="uniswapHovered = !uniswapHovered">
              <transition name="fade" mode="out-in">
                <div v-if="uniswapHovered" class="uniswap">
                  <!-- uniswap logo in svg -->
                  <svg
                    class="header__StyledUni-i6ilmw-7 crTzbB"
                    width="14"
                    height="15"
                    viewBox="0 0 14 15"
                    fill="none"
                  >
                    <g style="mix-blend-mode: darken">
                      <path
                        d="M4.15217 1.55141C3.96412 1.52242 3.95619 1.51902 4.04468 1.5055C4.21427 1.47958 4.61472 1.51491 4.89067 1.58012C5.53489 1.73232 6.12109 2.12221 6.74683 2.81466L6.91307 2.99862L7.15088 2.96062C8.15274 2.8006 9.17194 2.92778 10.0244 3.31918C10.2589 3.42686 10.6287 3.64121 10.6749 3.69629C10.6896 3.71384 10.7166 3.82684 10.7349 3.94742C10.7982 4.36458 10.7665 4.68434 10.6382 4.92317C10.5683 5.05313 10.5644 5.09432 10.6114 5.20554C10.6489 5.2943 10.7534 5.35999 10.8569 5.35985C11.0687 5.35956 11.2967 5.0192 11.4024 4.54561L11.4444 4.3575L11.5275 4.45109C11.9835 4.96459 12.3417 5.66488 12.4032 6.16335L12.4192 6.29332L12.3426 6.17517C12.2107 5.97186 12.0781 5.83346 11.9084 5.72183C11.6024 5.52062 11.2789 5.45215 10.4222 5.40727C9.64839 5.36675 9.21045 5.30106 8.77621 5.16032C8.03738 4.9209 7.66493 4.60204 6.78729 3.4576C6.39748 2.94928 6.15654 2.66804 5.91687 2.44155C5.37228 1.92691 4.83716 1.65701 4.15217 1.55141Z"
                        fill="black"
                      ></path>
                      <path
                        d="M10.8494 2.68637C10.8689 2.34575 10.9153 2.12108 11.0088 1.9159C11.0458 1.83469 11.0804 1.76822 11.0858 1.76822C11.0911 1.76822 11.075 1.82816 11.05 1.90142C10.9821 2.10054 10.9709 2.3729 11.0177 2.68978C11.0771 3.09184 11.1109 3.14985 11.5385 3.58416C11.739 3.78788 11.9723 4.0448 12.0568 4.15511L12.2106 4.35568L12.0568 4.21234C11.8688 4.03705 11.4364 3.6952 11.3409 3.64633C11.2768 3.61356 11.2673 3.61413 11.2278 3.65321C11.1914 3.68922 11.1837 3.74333 11.1787 3.99915C11.1708 4.39786 11.1161 4.65377 10.9842 4.90965C10.9128 5.04805 10.9015 5.01851 10.9661 4.8623C11.0143 4.74566 11.0192 4.69439 11.0189 4.30842C11.0181 3.53291 10.9255 3.34647 10.3823 3.02709C10.2447 2.94618 10.0179 2.8295 9.87839 2.76778C9.73887 2.70606 9.62805 2.6523 9.63208 2.64828C9.64746 2.63307 10.1772 2.78675 10.3905 2.86828C10.7077 2.98954 10.76 3.00526 10.7985 2.99063C10.8244 2.98082 10.8369 2.90608 10.8494 2.68637Z"
                        fill="black"
                      ></path>
                      <path
                        d="M4.51745 4.01304C4.13569 3.49066 3.89948 2.68973 3.95062 2.091L3.96643 1.90572L4.05333 1.92148C4.21652 1.95106 4.49789 2.05515 4.62964 2.13469C4.9912 2.35293 5.14773 2.64027 5.30697 3.37811C5.35362 3.59423 5.41482 3.8388 5.44298 3.9216C5.48831 4.05487 5.65962 4.36617 5.7989 4.56834C5.89922 4.71395 5.83258 4.78295 5.61082 4.76305C5.27215 4.73267 4.8134 4.41799 4.51745 4.01304Z"
                        fill="black"
                      ></path>
                      <path
                        d="M10.3863 7.90088C8.60224 7.18693 7.97389 6.56721 7.97389 5.52157C7.97389 5.36769 7.97922 5.24179 7.98571 5.24179C7.99221 5.24179 8.06124 5.29257 8.1391 5.35465C8.50088 5.64305 8.906 5.76623 10.0275 5.92885C10.6875 6.02455 11.0589 6.10185 11.4015 6.21477C12.4904 6.57371 13.1641 7.30212 13.3248 8.29426C13.3715 8.58255 13.3441 9.12317 13.2684 9.4081C13.2087 9.63315 13.0263 10.0388 12.9779 10.0544C12.9645 10.0587 12.9514 10.0076 12.9479 9.93809C12.9296 9.56554 12.7402 9.20285 12.4221 8.93116C12.0604 8.62227 11.5745 8.37633 10.3863 7.90088Z"
                        fill="black"
                      ></path>
                      <path
                        d="M9.13385 8.19748C9.11149 8.06527 9.07272 7.89643 9.04769 7.82228L9.00217 7.68748L9.08672 7.7818C9.20374 7.91234 9.2962 8.07937 9.37457 8.30185C9.43438 8.47165 9.44111 8.52215 9.44066 8.79807C9.4402 9.06896 9.43273 9.12575 9.3775 9.27859C9.29042 9.51959 9.18233 9.69048 9.00097 9.87391C8.67507 10.2036 8.25607 10.3861 7.65143 10.4618C7.54633 10.4749 7.24 10.4971 6.97069 10.511C6.292 10.5461 5.84531 10.6186 5.44393 10.7587C5.38623 10.7788 5.3347 10.7911 5.32947 10.7859C5.31323 10.7698 5.58651 10.6079 5.81223 10.4998C6.1305 10.3474 6.44733 10.2643 7.15719 10.1468C7.50785 10.0887 7.86998 10.0183 7.96194 9.99029C8.83033 9.72566 9.27671 9.04276 9.13385 8.19748Z"
                        fill="black"
                      ></path>
                      <path
                        d="M9.95169 9.64109C9.71465 9.13463 9.66022 8.64564 9.79008 8.18961C9.80399 8.14088 9.82632 8.101 9.83976 8.101C9.85319 8.101 9.90913 8.13105 9.96404 8.16777C10.0733 8.24086 10.2924 8.36395 10.876 8.68023C11.6043 9.0749 12.0196 9.3805 12.302 9.72965C12.5493 10.0354 12.7023 10.3837 12.776 10.8084C12.8177 11.0489 12.7932 11.6277 12.7311 11.8699C12.5353 12.6337 12.0802 13.2336 11.4311 13.5837C11.336 13.635 11.2506 13.6771 11.2414 13.6773C11.2321 13.6775 11.2668 13.5899 11.3184 13.4827C11.5367 13.029 11.5616 12.5877 11.3965 12.0965C11.2954 11.7957 11.0893 11.4287 10.6732 10.8084C10.1893 10.0873 10.0707 9.89539 9.95169 9.64109Z"
                        fill="black"
                      ></path>
                      <path
                        d="M3.25046 12.3737C3.91252 11.8181 4.73629 11.4234 5.48666 11.3022C5.81005 11.25 6.34877 11.2707 6.64823 11.3469C7.12824 11.469 7.55763 11.7425 7.78094 12.0683C7.99918 12.3867 8.09281 12.6642 8.19029 13.2816C8.22875 13.5252 8.27057 13.7697 8.28323 13.8251C8.35644 14.1451 8.4989 14.4008 8.67544 14.5293C8.95583 14.7333 9.43865 14.7459 9.91362 14.5618C9.99423 14.5305 10.0642 14.5089 10.0691 14.5138C10.0864 14.5308 9.84719 14.6899 9.67847 14.7737C9.45143 14.8864 9.2709 14.93 9.03102 14.93C8.59601 14.93 8.23486 14.7101 7.9335 14.2616C7.87419 14.1733 7.7409 13.909 7.63729 13.6741C7.3191 12.9528 7.16199 12.7331 6.79255 12.4926C6.47104 12.2834 6.05641 12.2459 5.74449 12.3979C5.33475 12.5976 5.22043 13.118 5.51389 13.4478C5.63053 13.5789 5.84803 13.6919 6.02588 13.7139C6.35861 13.7551 6.64455 13.5035 6.64455 13.1696C6.64455 12.9528 6.56071 12.8291 6.34966 12.7344C6.0614 12.6051 5.75156 12.7563 5.75304 13.0254C5.75368 13.1402 5.80396 13.2122 5.91971 13.2643C5.99397 13.2977 5.99569 13.3003 5.93514 13.2878C5.67066 13.2333 5.6087 12.9164 5.82135 12.706C6.07667 12.4535 6.60461 12.5649 6.78591 12.9097C6.86208 13.0545 6.87091 13.3429 6.80451 13.517C6.6559 13.9068 6.22256 14.1117 5.78297 14.0002C5.48368 13.9242 5.36181 13.842 5.00097 13.4726C4.37395 12.8306 4.13053 12.7062 3.22657 12.566L3.05335 12.5391L3.25046 12.3737Z"
                        fill="black"
                      ></path>
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M0.308383 0.883984C2.40235 3.40996 3.84457 4.45213 4.00484 4.67231C4.13717 4.85412 4.08737 5.01757 3.86067 5.14567C3.7346 5.21689 3.47541 5.28905 3.34564 5.28905C3.19887 5.28905 3.14847 5.23278 3.14847 5.23278C3.06337 5.15255 3.01544 5.16658 2.5784 4.39555C1.97166 3.45981 1.46389 2.68357 1.45004 2.67057C1.41801 2.64052 1.41856 2.64153 2.51654 4.59413C2.69394 5.0011 2.55182 5.15049 2.55182 5.20845C2.55182 5.32636 2.51946 5.38834 2.37311 5.55059C2.12914 5.8211 2.02008 6.12505 1.94135 6.7541C1.8531 7.45926 1.60492 7.95737 0.917156 8.80989C0.514562 9.30893 0.448686 9.4004 0.3471 9.60153C0.219144 9.85482 0.183961 9.99669 0.169701 10.3165C0.154629 10.6547 0.183983 10.8732 0.287934 11.1965C0.378939 11.4796 0.473932 11.6665 0.716778 12.0403C0.926352 12.3629 1.04702 12.6027 1.04702 12.6965C1.04702 12.7711 1.06136 12.7712 1.38611 12.6983C2.16328 12.5239 2.79434 12.2171 3.14925 11.8411C3.36891 11.6084 3.42048 11.4799 3.42215 11.1611C3.42325 10.9525 3.41587 10.9088 3.35914 10.7888C3.2668 10.5935 3.09869 10.4311 2.72817 10.1794C2.2427 9.84953 2.03534 9.58398 1.97807 9.21878C1.93108 8.91913 1.98559 8.70771 2.25416 8.14825C2.53214 7.56916 2.60103 7.32239 2.64763 6.73869C2.67773 6.36158 2.71941 6.21286 2.82842 6.09348C2.94212 5.969 3.04447 5.92684 3.32584 5.88863C3.78457 5.82635 4.07667 5.70839 4.31677 5.48849C4.52505 5.29772 4.61221 5.11391 4.62558 4.8372L4.63574 4.62747L4.51934 4.49259C4.09783 4.00411 0.0261003 0.5 0.000160437 0.5C-0.00538105 0.5 0.133325 0.672804 0.308383 0.883984ZM1.28364 10.6992C1.37894 10.5314 1.3283 10.3158 1.16889 10.2104C1.01827 10.1109 0.78428 10.1578 0.78428 10.2875C0.78428 10.3271 0.806303 10.3559 0.855937 10.3813C0.939514 10.424 0.945581 10.4721 0.879823 10.5703C0.81323 10.6698 0.818604 10.7573 0.894991 10.8167C1.0181 10.9125 1.19237 10.8598 1.28364 10.6992Z"
                        fill="black"
                      ></path>
                      <path
                        fill-rule="evenodd"
                        clip-rule="evenodd"
                        d="M4.92523 5.99865C4.70988 6.06439 4.50054 6.29124 4.43574 6.5291C4.39621 6.67421 4.41864 6.92875 4.47785 7.00736C4.57351 7.13433 4.66602 7.16778 4.91651 7.16603C5.40693 7.16263 5.83327 6.95358 5.88284 6.69224C5.92347 6.47801 5.73622 6.18112 5.4783 6.05078C5.34521 5.98355 5.06217 5.95688 4.92523 5.99865ZM5.49853 6.44422C5.57416 6.33741 5.54107 6.22198 5.41245 6.14391C5.1675 5.99525 4.79708 6.11827 4.79708 6.34826C4.79708 6.46274 4.99025 6.58765 5.16731 6.58765C5.28516 6.58765 5.44644 6.5178 5.49853 6.44422Z"
                        fill="black"
                      ></path>
                    </g>
                  </svg>
                  <!-- uniswap logo in svg -->
                  Uniswap Pools
                </div>
              </transition>
              <transition name="slowFade">
                <div v-if="!uniswapHovered" class="uniPool" id="first">
                  <a
                    class="uni-btn"
                    rel="noopener noreferrer"
                    href="https://uniswap.info/pair/0x9371801bb71ff2756c9ffac371b9c311f77bf986"
                    target="_blank"
                  >
                    1
                  </a>
                </div>
              </transition>
              <transition name="slowFade">
                <div v-if="!uniswapHovered" class="uniPool" id="second">
                  <a
                    class="uni-btn"
                    rel="noopener noreferrer"
                    href="https://uniswap.info/pair/0x45ea0521d9e428ab078af2586c3bee97a5e5156c"
                    target="_blank"
                  >
                    2
                  </a>
                </div>
              </transition>
            </div>
            <div>
              <a
                class="uniswap"
                id="token"
                rel="noopener noreferrer"
                href="https://info.uniswap.org/pair/0xE4f8F3CB9b33247789e4984A457bb69A1a621Df3"
                target="_blank"
              >
                <img src="../assets/logo.svg" alt="" id="logoInButton" />
                <div>Buy Snow!</div>
              </a>
            </div>
            <button
              v-show="snowAmount === 0"
              class="claim-snow-btn"
              type="button"
              @click="openClaimSnowPopup"
            >
              <div>
                <span>Claim snow</span>
              </div>
            </button>

            <button
              class="s-button s-button--additional snow-btn"
              type="button"
              v-show="snowAmount > 0"
            >
              <img :src="publicPath + 'snow.svg'" />
              <span>{{ snowAmount }} SNOW</span>
            </button>
          </div>
        </div>
      </div>
      <div id="screen" v-show="poolClosed">
        <!--        <div :class="{'blue window': true, [$route.name]: true}">-->
        <!--            <h1><img :src="logoSrc" alt="🌀 Curve"></h1>-->
        <!--        </div>-->
        <h1 class="main-header">{{ $route.name }}</h1>
        <div
          class="error window half-width info"
          id="error-window"
          v-show="error"
        >
          {{ error }}
        </div>
        <div class="window white connected-wallet" v-show="hasConnectedWallet">
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
        <!--        <div class="simple-error window" v-show="plsReturn">-->
        <!--          Your recent withdrawal from SnowSwap resulted in getting 1000 more USDT-->
        <!--          because of another user mistakenly transferring funds to the contract.-->
        <!--          If you wish to return them - please contact us on-->
        <!--          <a href="https://twitter.com/snow_swap">Twitter</a>-->
        <!--          /-->
        <!--          <a href="https://discord.gg/CSSJTs4" rel="noopener noreferrer">-->
        <!--            @Discord-->
        <!--          </a>-->
        <!--          . Thank you!-->
        <!--        </div>-->
        <router-view />

        <div class="socials-wrapper">
          <div class="socials">
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
        </div>

        <PopUp v-if="popUp" />
      </div>
      <balances-info
        v-show="poolClosed"
        :class="{ [$route.name]: true }"
        :bal_info="bal_info"
        :total="balTotal"
        :l_info="l_info"
        :totalShare="totalShare"
        :staked_info="staked_info"
        :totalStake="totalStake"
        :fee="fee"
        :admin_fee="admin_fee"
        :currencies="currencies"
        v-if="
          !['Stats', 'FAQ', 'Donate', 'Root', 'CombinedStats', 'Stake'].includes(
            $route.name
          )
        "
      />
      <div v-show="!poolClosed" class="coming-soon">
        <h1 class="main-header">Coming soon</h1>
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

    <ClaimSnow v-if="showClaimSnowPopup" @close="hideClaimSnowPopup" />
  </div>
</template>

<script>
import BalancesInfo from "../components/BalancesInfo.vue";
import {
  getters,
  contract as currentContract,
  changeContract,
  poolMenu,
} from "../contract";
import init, { onboard, changeWallets } from "../init";
import allabis from "../allabis";
import PopUp from "./PopUp";
import ClaimSnow from "@/components/common/ClaimSnow";

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
  hbtc: "HBTC",
};

export default {
  metaInfo() {
    return {
      title: "SnowSwap :: " + titles[this.currentPool],
    };
  },
  data: () => ({
    gitBranches: {
      compound: "compound",
      usdt: "usdt",
      iearn: "y",
      y: "y",
      busd: "busd",
      susd: "susd_vulnerable",
      susdv2: "susd_plain",
      pax: "pax",
      tbtc: "tbtc",
      ren: "ren",
      sbtc: "renbtc_sbtc",
      hbtc: "hbtc",
    },
    uniswapHovered: true,
    popUp: true,
    // snowAmount: null,
    snowAmount: 0, // for test purpose
    showClaimSnowPopup: false,
  }),
  components: {
    BalancesInfo,
    ClaimSnow,
    PopUp,
  },
  computed: {
    poolClosed() {
      // return this.currentPool === "iearn";
      return true;
    },
    allGetters() {
      return getters;
    },
    ...getters,
    poolMenu() {
      return poolMenu;
    },
    poolAddress() {
      return allabis[this.currentPool].swap_address;
    },
    tokenAddress() {
      return allabis[this.currentPool].token_address;
    },
    publicPath() {
      return process.env.BASE_URL;
    },
    logoSrc() {
      if (!currentContract.swapbtc)
        return this.publicPath + "logo_snowball.svg";
      else return this.publicPath + "logo_ren_beta_optimized.svg";
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
    openClaimSnowPopup: function () {
      this.showClaimSnowPopup = true;
      document.body.style.overflow = "hidden";
    },
    hideClaimSnowPopup: function () {
      this.showClaimSnowPopup = false;
      document.body.style.overflow = "";
    },
  },
};
</script>

<style>
#screen {
  position: relative;
}
#changeAccounts {
  margin-top: 0.3em;
}
a.showmobile {
  display: none;
}
@media only screen and (min-device-width: 320px) and (max-device-width: 730px) {
  #hamburger:checked ~ a.showmobile {
    display: block;
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
.CRV a:hover,
.CRV a:visited {
  color: white;
}
.coming-soon {
  display: flex;
  height: 100%;
  justify-content: center;
  align-items: center;
}
.uniPools {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  /* background-color: rgba(6, 215, 190, 0.1); */
  align-items: center;
  justify-content: space-between;
  width: 80px;
  height: 80%;
  max-height: 44px;
  font-family: Arial;
  cursor: pointer;
  text-align: center;
}
.uniPool {
  border: 1px solid #06d7be55;
  border-radius: 33%;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
  transition: 0.11s ease-in-out;
}
.uniPool#first {
  padding: 0 8px;
  border-right: 1px solid transparent;
  margin-right: -5px;
  margin-left: 5px;
}
.uniPool#second {
  padding: 0 8px;
  border-left: 1px solid transparent;
  margin-right: 6px;
  margin-left: -4px;
}
.uniPool:hover {
  transform: scale(0.9);
  transition: 0.11s ease-in-out;
  background-color: #06d7be11;
}
.uni-btn {
  display: block;
  padding: 30px 0 0 0;
  height: 100%;
  width: 100%;
  text-align: center;
  color: #414a5b;
  text-decoration: none;
  transition: 0.3s ease-in-out;
}
.uni-btn:hover,
.uni-btn:active {
  color: #00b9c2;
  transition: 0.3s ease-in-out;
}
.uniswap {
  display: inline-block;
  border: 1px solid #06d7be55;
  height: 100%;
  padding: 10px 8px 10px 8px;
  margin-left: 5px;
  color: #00b9c2;
  /* color: #414a5b; */
  font-family: Arial;
  text-align: center;
  border-radius: 17%;
  transition: 0.15s ease-in-out;
}
.uniswap:hover {
  transform: scale(0.9);
  box-shadow: 0 3px 15px #06d7be55;
  transition: 0.15s ease-in-out;
}
.fade-enter-active {
  animation: slowfade 0.5s;
}
.fade-leave-active {
  transition: opacity 0.2s;
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}

.slowFade-enter-active {
  animation: slowfade 0.5s;
}
.slowFade-leave-active {
  transition: opacity 0s;
}
.slowFade-enter,
.slowFade-leave-to {
  opacity: 0;
}
@keyframes slowfade {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
#token {
  color: #00b9c2;
  text-decoration: none;
  font-family: Arial;
  text-align: center;
  font-weight: 700;
  margin-left: 10px;
  max-width: 60px;
  padding: 5px;
  display: block;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
#token:active {
  transform: rotate(20deg) scale(0.9);
}
#logoInButton {
  width: 20px;
}
</style>
