<template>
  <div id="app">
    <Spellchecker :spellcheckText="spellcheckText" @donechecking="updateText"/>
  </div>
</template>

<script>
import Spellchecker from "./components/Spellchecker";

export default {
  name: "App",
  components: {
    Spellchecker
  },
  data() {
    return { spellcheckText: 'something to start wth' }
  },
  methods: {
    updateText(value) {
      this.spellcheckText = value;
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
