<template>
  <span class="bad-word" :data-word="key" :style="{ top: top + 'px', left: left + 'px' }">{{spacer}}</span>
</template>

<script>
export default {
  name: "BadWord",
  props: ['key', 'top', 'left'],
  computed: {
    spacer() {
      return this.key.split('').map(m => ' ').join('');
    }
  }
};
</script>

<style>
.bad-word {
  border-bottom: 2px solid blue;
  position: absolute;
  pointer-events: none;
}
</style>
