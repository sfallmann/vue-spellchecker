<template>
  <div
    class="box"
    id="box"
    @mouseover="overBox"
    @mouseleave="resetBox"
    v-show="showBox"
    :style="{ top: `${boxY}px`, left: `${boxX}px` }"
  >
    <p><strike>{{ boxText }}</strike> <span class="arrow">&#x2192;</span></p>

    <div :key="`div_${suggestion}`" v-for="suggestion in suggestions">
      <label>{{suggestion}}:</label> 
      <button  @click="fixWord(suggestion)" class="spellcheck-modal-btn"> Replace </button>
      <button @click="fixAll(suggestion)" class="spellcheck-modal-btn"> Replace All </button>
    </div>
    <br />
    <div class="spellcheck-modal-btn ignore" @click="ignoreWord(boxText)">Ignore Word</div>
    <br />
    <input type="text" @keyup.enter="handleReplace" v-model="wordInput" class="spellcheck-modal-btn" placeholder="Enter replacement"/>
  </div>
</template>
<script>
export default {
  props: ['showBox', 'boxY', 'boxX', 'boxText',  'suggestions'],
  data() {
    return {
      wordInput: '',
    }
  },
  methods: {
    handleReplace() {
      this.$emit('replace', this.wordInput);
    },
    fixWord(suggestion) {
      console.log(suggestion)
      this.$emit('fixword', suggestion)
    },
    ignoreWord(suggestion) {
      this.$emit('ignore', suggestion);
    },
    fixAll(suggestion) {
      console.log(suggestion)
      this.$emit('fixall', suggestion)
    },
    overBox() {
      this.$emit('overbox', true);
    },
    resetBox() {
      this.$emit('resetbox');
    }
  },
}
</script>
<style>
.box {
  text-align: justify;
  z-index: 100;
  padding: 10px 5px 5px 5px;
  max-width: 300px;
  height: 225px;
  position: absolute;
  background: white;
  border-radius: 5px;
  box-shadow: 0px 2px 5px 3px rgba(179, 170, 170, 0.64);
}
.arrow {
  font-size: 24px;
  font-weight: bold;
  margin: 0 3px 0 6px;
}

.spellcheck-modal-btn {
  cursor: pointer;
}
.ignore {
  font-weight: bold;
  color: green;
}
</style>
