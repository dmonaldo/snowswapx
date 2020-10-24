<template>
  <div class="PopUp" v-if="!closed">
    <div class="PopUp-Container">
      <div class="s-btn-social" id="close" @click="closePopUp"></div>
      <a
        href="https://medium.com/snowswap/snow-whats-next-29639c181732"
        rel="noopener noreferrer"
        target="_blank"
      >
        <div id="text">
          Snow Airdrop on
          <br />
          Oct 30th!
          <br />
          <div id="image"></div>
          <span style="font-size: 18px">
            Deposit during the period of
            <br />
            Oct 15th-29th to qualify.
          </span>
        </div>
      </a>
    </div>
  </div>
</template>

<script>
export default {
  name: "PopUp",
  data: () => {
    return {
      closed: false,
    };
  },
  methods: {
    closePopUp() {
      this.closed = true;
    },
  },
};
</script>

<style scoped>
.PopUp {
  font-family: BrandonGrotesque, sans-serif;
  font-weight: 700;
  font-size: 22px;
  text-align: center;
  color: #1adbc5;
  margin: 0;
  display: flex;
  justify-content: end;
  align-items: flex-end;
  position: absolute;
  height: 100%;
  right: 2rem;
  bottom: 0;
  pointer-events: none;
}

.PopUp-Container {
  width: 271px;
  height: 350px;
  bottom: 3rem;
  position: sticky;
  position: -webkit-sticky;
  background-color: white;
  box-shadow: 0 8px 30px 0 #e1e6ec7f;
  border-radius: 16px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  transition: 0.3s ease-in-out;
  pointer-events: auto;
}
.PopUp-Container:hover {
  transform: scale(1.05);
  transition: 0.3s ease-in-out;
}
#close {
  background-image: url("../assets/X.png");
  background-color: transparent;
  background-position: center;
  position: absolute;
  right: 5px;
  top: 5px;
  transition: 0.15s ease-in-out;
}
#close:hover {
  transform: scale(0.9);
  transition: 0.15s ease-in-out;
  box-shadow: inset 0 0 10px 0 #838b9d74;
}
#image {
  width: 209px;
  height: 180px;
  background-image: url("../assets/Coming soon Placeholder 4 1.png");
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}
a {
  text-decoration: none;
  font-family: BrandonGrotesque, sans-serif;
  font-weight: 700;
  font-size: 23.5px;
  color: #1adbc5;
}
</style>
