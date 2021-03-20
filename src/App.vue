<template>
  <div id="app">
    <div class="top">
      <div class="container-left">
          <img id="logo-neec-full" src="../public/Logo_NEEC_White.png" alt="Logo NEEC Completo">
      </div>

      <div class="container-center">
          <h1>Speed Quiz</h1>
      </div>

      <div class="container-right">
      </div>
    </div>
    <Quiz />
    <footer>
      <p id="createdBy">
        created by <a href="https://github.com/NEECIST">NEECIST</a>
      </p>
    </footer>
  </div>
</template>

<script>
import Quiz from "@/components/Quiz.vue";

export default {
  name: "App",
  components: {
    Quiz,
  },
};
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
}

/* DUARTE CSS - Top bar */

@font-face {
    font-family: Arcade;
    src: url(../public/PressStart2P-Regular.ttf);
}

.top {
    position: relative;
    top: 0;
    width: 100%;
    background-color: #009de0;
    font-family: Arcade;
    font-size: 14px;
    display: flex;
    justify-content: space-between;
    align-items: center
}

.container-left {
    display: flex;
    height: auto;
    width: 15vw;
    text-align: left;
}

.container-right {
    display: flex;
    height: auto;
    width: 15vw;
    align-items: center;
}

.container-center {
    display: flex;
    height: auto;
    width: auto;
}

.top h1 {
    margin: 5px;
    color: white;
    font-size: 2.5em;
    text-align: center;
}

.top img {
    width: 10vw;
    margin-right: auto;
    margin-left: 0.5vw;
    margin-top: auto;
    margin-bottom: auto;
}

.top a {
    margin-left: auto;
    margin-right: 0.5vw;
    margin-top: auto;
    margin-bottom: auto;
    text-decoration: none;
    text-align: end;
    color: white;
}

/* DUARTE CSS END */

#app {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica,
    Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

footer {
  position: fixed;
  bottom: 0;
  padding: 0.5rem 1rem;
  width: 100%;
  font-size: 0.7rem;
  background-color: #009de0;
}

#createdBy {
  float: right;
  color: white;
  font-weight: bold;
}

#createdBy a {
  color: white;
  font-weight: bold;
}
</style>
