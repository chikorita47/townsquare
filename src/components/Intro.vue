<template>
  <div class="intro">
    <img src="static/demon-head2.png" alt="" class="logo" />
    <div>
      Welcome to the (unofficial)
      <b>Virtual Town Square and Grimoire</b> for Blood on the Clocktower!
      Please add more players through the
      <span class="button" @click="toggleMenu">
        <font-awesome-icon icon="cog" /> Menu
      </span>
      on the top right or by pressing <b>[A]</b>. You can also join a game
      session by pressing <b>[J]</b>.<br />
      <div class="footer">
        This project is free and open source and can be found on
        <a href="https://github.com/bra1n/townsquare" target="_blank">GitHub</a
        >. It is not affiliated with The Pandemonium Institute. "Blood on the
        Clocktower" is a trademark of Steven Medway and The Pandemonium
        Institute.
      </div>
    </div>
    <a
      class="redirect"
      v-if="language === 'zh-CN'"
      href="https://clocktower.gstonegames.com"
    >
      <img src="../assets/gstone.png" class="gstone" alt="" />
      你想使用中文版魔典吗？
    </a>
  </div>
</template>

<script>
import { mapMutations } from "vuex";

export default {
  data() {
    return {
      language: window.navigator.userLanguage || window.navigator.language,
    };
  },
  methods: mapMutations(["toggleMenu"]),
};
</script>

<style scoped lang="scss">
@import "../vars.scss";
// Intro
.intro {
  text-align: center;
  width: 50%;
  font-size: 120%;
  position: absolute;
  padding: 10px;
  background: rgba(96, 96, 96, 0.5);
  border-radius: 10px;
  z-index: 3;
  display: flex;
  justify-content: center;
  a {
    color: white;
  }
  a.redirect {
    display: block;
    text-decoration: none;
    position: absolute;
    top: 100%;
    margin-top: 2vh;
    padding: 10px;
    background: rgba(96, 96, 96, 0.5);
    border-radius: 10px;

    &:hover {
      color: $hover-red;
    }
    img {
      width: 120px;
      display: block;
      margin: auto;
      margin-bottom: 1vh;
    }
  }

  img.logo {
    position: absolute;
    bottom: 100%;
    width: 25vh;
    margin-bottom: 2vh;
    max-width: 192px;
  }
  .footer {
    font-size: 60%;
    opacity: 0.75;
  }
}
</style>
