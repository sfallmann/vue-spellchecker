<template>
  <div class="wrapper" @mouseover="handleMouseOver" @keyup.tab="handleTab">
    <SuggestionPopup 
      @overbox="overBox = true"
      @resetbox="resetBox"
      @fixword="fixWord"
      @fixall="fixAll"
      @replace="handleReplace"
      @ignore="ignoreWord"
      :showBox="showbox"
      :boxY="boxY"
      :boxX="boxX"
      :boxText="boxText"
      :suggestions="corrections && corrections[boxText]"
    />
    <div
      :tabindex="isChecking ? 4 : 0"
      id="editor"
      :key="editorKey"
      :class="{ noselect: isChecking }"
      :contenteditable="!isChecking"
      v-html="text"
      @focusout="onDivInput($event);"
      spellcheck="false"
    >
    </div>
    <br />
    <button tabindex="5" id="spellcheck" @click="spellcheck">{{ isChecking ? 'Done' : 'Spellcheck' }}</button>
  </div>
</template>

<script>
import axios from 'axios';
import { setTimeout } from 'timers';
import SuggestionPopup from './SuggestionPopup';
import json from '../../secrets.json';
export default {
  name: "Spellchecker",
  components: {
    SuggestionPopup,
  },
  props: ['spellcheckText'],
  created() {
    this.text = this.spellcheckText;
  },
  updated() {
    if (!this.isChecking && this.text !== this.spellcheckText) {
      console.log('done checking and text doesnt equal spellcheckText');
      this.text = this.spellcheckText;
    }
  },
  data() {
    return {
      text: "",
      showbox: false,
      boxY: 0,
      boxX: 0,
      boxText: "",
      selectedEl: null,
      editorKey: "editor_start",
      overBox: false,
      corrections: [],
      ignoreList: new Set(),
      isChecking: false,
      lastIndex: null,
      json,
    };
  },
  methods: {
    mounted() {
      document.body.addEventListener('keyup', this.handleTab);
    },
    created() {
      const reader = new FileReader();
      reader.readAsText()
    },
    destroyed() {
      document.body.removeEventListener('keyup', this.handleTab);
    },
    onDivInput(event) {
      if (this.isChecking) return;
      this.text = event.target.textContent;
      this.$emit('donechecking', this.text)
    },
    async spellcheck() {
      this.isChecking = !this.isChecking;
      let text = this.sanitize();
      
      if (!this.isChecking) {
        this.resetBox();
        this.text = text;
        this.corrections = {};
        this.$emit('donechecking', this.text);
        return;
      }
      
      const encodedText = encodeURIComponent(text);
      let response = await axios.get(`https://montanaflynn-spellcheck.p.rapidapi.com/check/?text=${encodedText}`, 
        { headers: { "X-RapidAPI-Key": json.API_KEY } }
      )

      const { corrections } = response.data;

      if (!corrections) return;
    
      let keys = Object.keys(corrections);
      this.corrections = corrections;
      keys = keys.concat([...this.ignoreList]);
      keys = keys.filter((k) => { 
        const keep = keys.indexOf(k) === keys.lastIndexOf(k);
        if (!keep) delete this.corrections[k];
        return keep
      });
      console.log(this.corrections)
      keys.forEach((key) => {

        if (this.corrections[key] && this.corrections[key].length > 3) {
          this.corrections[key].splice(3, this.corrections[key].length);
        }
        
        //const regex = new RegExp(`\\b(?<!-)${key}(?!-)\\b`, 'gm');
        const regex = new RegExp(`\\b${key}\\b`, 'gm');
        let matches = text.match(regex);
        text = text.replace(
          regex,
          //`<span style="border-bottom: 2px solid blue;" data-word="${key}">${key}</span>`
          
          (match, p1, string) => {
            const html = `<span style="border-bottom: 2px solid blue;" data-word="${key}">${key}</span>`;
            if (string.charAt(p1 - 1) === '-' || string.charAt(p1 + match.length) === '-') {
              return key;
            }
            return html;
          }
          
        );
        this.text = text;
      });

      this.editorKey = `editor_${new Date().getTime()}`;
      document.getElementById('spellcheck').focus();
      
    },
    sanitize() {
      const editor = document.getElementById('editor');
      editor.innerHTML = editor.innerHTML.replace(/<(?:.|\n)*?>/gm, '');
      return editor.textContent;
    },
    handleChange() {
      const text = document.getElementById('editor').textContent;
      console.log('input');
      this.$emit('spellcheckText', text);
    },
    handleReplace(word) {
      this.fixWord(word);
      this.replaceWord = '';
    },
    handleMouseOver(event) {
      const el = event.target;
      
      if (!this.overBox && this.selectedEl !== el) {
        this.resetBox();
      }

      const word = event.target.getAttribute("data-word");
      if (!word) return;

      this.showbox = true;
      this.selectedEl = event.target;
      this.selectedEl.style.backgroundColor = "lightblue";
      this.selectedEl.setAttribute("data-status", "active");
      this.boxY = event.target.offsetTop + 18;
      this.boxX = event.target.offsetLeft - 5;
      this.boxText = word;
      
    },
    handleMouseLeave(event) {
      const el = event.target;

      if (!this.overBox && this.showbox && this.selectedEl !== el) {
        this.resetBox();
      }
    },
    handleTab(event) {
      if (event.keyCode !== 9) return;
      if (!this.isChecking) return;
      document.getElementById('spellcheck').focus();
      const els = document.querySelectorAll('[data-word]');
      if (!els.length) {
        return;
      } else if (this.lastIndex === null || (this.lastIndex + 1 >= els.length)) {
        this.lastIndex = 0;
      } else  {
        this.lastIndex = this.lastIndex + 1;
      }
      Array.prototype.forEach.call(els, (el => el.style.backgroundColor = ''));
      this.handleMouseOver({ target: els[this.lastIndex] });

    },
    fixWord(suggestion) {
      const textNode = document.createTextNode(suggestion);
      document.getElementById('editor').replaceChild(textNode, this.selectedEl);
      this.text = document.getElementById("editor").innerHTML;
      this.resetBox();
    },
    ignoreWord(word) {
      const els = document.querySelectorAll(`[data-word="${word}"]`);
      Array.prototype.forEach.call(els, (el) => {
        el.removeAttribute("data-word");
        el.style.borderBottom = "";
      });
      //this.selectedEl.removeAttribute("data-word");
      //this.selectedEl.style = "";
      this.text = document.getElementById("editor").innerHTML;
      delete this.corrections[word];
      this.ignoreList.add(word);
      this.resetBox();
    },
    fixAll(word) {
      const els = document.querySelectorAll(
        `span[data-word="${this.boxText}"]`
      );
      Array.prototype.forEach.call(
        els,
        (el, index) => {
          const textNode = document.createTextNode(word);
          el.parentNode.replaceChild(textNode, el);
        }
      );
      this.text = document.getElementById("editor").innerHTML;
      this.resetBox();
    },
    resetBox() {    
      this.showbox = false;
      this.overBox = false;
      this.boxY = 0;
      this.boxX = 0;
      this.boxText = "";
      if (this.selectedEl) {
        this.selectedEl.style.backgroundColor = "";
        this.selectedEl.removeAttribute("data-status");
        this.selectedEl = null;
      }
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wrapper {
  position: relative;
}

.underline {
  border-bottom: 2px solid blue;
}
#editor {
  text-align: left;
  border: 2px solid black;
  height: 200px;
}

.noselect {
  user-select: none; 
}

</style>
