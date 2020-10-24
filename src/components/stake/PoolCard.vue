<template>
  <div class="pool-card">
    <div class="pool-card__content">
      <div class="pool-card__img-wrapper">
        <img :src="publicPath + pool.img" />
      </div>

      <h3 class="pool-card__name">{{ pool.name }}</h3>

      <span v-if="pool.pair" class="pool-card__pair">
        <span class="pool-card__pair--accent">{{ pool.pair }}</span>
        UNI-V2 LP
      </span>

      <div class="pool-card__apy">
        <span>
          APY:
          <span class="pool-card__apy--accent">
            <div v-if="pool.apy == 'loading'"><loading /></div>
            <div v-else-if="pool.apy">{{ pool.apy.apy.toFixed(1) }}</div>
            <div v-else>🧐</div>
          </span>
        </span>
      </div>

      <button class="pool-card__select-btn" @click="onSelectPool(pool)">
        Select
      </button>
    </div>
  </div>
</template>

<script>
import Loading from "../common/loading";
export default {
  components: {
    Loading,
  },
  props: ["pool", "onSelectPool"],
  computed: {
    publicPath() {
      return process.env.BASE_URL;
    },
  },
};
</script>

<style scoped>
.pool-card {
  padding: 2px;
  border: 1px solid rgba(217, 223, 234, 0.5);
  border-radius: 16px;
  box-shadow: 0 8px 30px rgba(225, 230, 236, 0.6);
  background-color: #fff;
}
.pool-card:hover {
  padding: 3px;
  box-shadow: 0 5px 40px rgba(24, 219, 194, 0.15);
  border: none;
  background: linear-gradient(147.57deg, #1fdacf 2.48%, #00dd92 94.42%);
}
.pool-card__content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 23px;
  border-radius: 13px;
  background-color: #fff;
}
.pool-card__img-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 96px;
  height: 96px;
  border-radius: 50%;
  box-shadow: inset 0 2px 10px #e1e6ec;
}
.pool-card__name {
  margin: 15px 0 0 0;
  font-style: normal;
  font-weight: bold;
  font-size: 26px;
  line-height: 18px;
  text-align: center;
  letter-spacing: -0.4px;
  color: #414a5b;
}
.pool-card__stake {
  margin-top: 12px;
  font-style: normal;
  font-weight: 500;
  font-size: 18px;
  line-height: 22px;
  text-align: center;
  color: #838b9d;
}
.pool-card__stake--accent {
  font-weight: 700;
  color: #00b9c2;
}
.pool-card__apy {
  text-align: center;
  min-width: 148px;
  margin-top: 26px;
  padding: 8px 14px;
  border: 1px solid rgba(217, 223, 234, 0.5);
  border-radius: 16px;
  font-weight: 500;
  font-size: 18px;
  line-height: 22px;
  color: #838b9d;
  display: flex;
  flex-direction: column;
}
.pool-card__apy--accent {
  font-weight: 700;
  color: #414a5b;
}
.pool-card__pair {
  margin-top: 12px;
  font-weight: 500;
  font-size: 18px;
  line-height: 22px;
  text-align: center;
  color: #838b9d;
}
.pool-card__pair--accent {
  font-weight: 700;
  color: #00b9c2;
}
.pool-card__select-btn {
  min-width: 240px;
  height: 51px;
  margin-top: 28px;
  border: 1px solid #00b9c2;
  border-radius: 16px;
  text-transform: uppercase;
  font-weight: bold;
  font-size: 15px;
  line-height: 21px;
  color: #00b9c2;
  background-color: #fff;
  outline: none;
}
.pool-card__select-btn:hover {
  border: none;
  color: #fff;
  background: linear-gradient(88.31deg, #1fdacf 3.6%, #00dd92 99.16%);
}
</style>
