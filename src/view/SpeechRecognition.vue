<template>
  <i
    class="material-icons"
    @click="recognize"
  >keyboard_voice</i>
</template>

<script>
import Modernizr from 'modernizr';
import Materialize from 'materialize-css';

/**
 * Workaround the issue that Modernizr return a function instead of a class
 * @param {string} prop - the name of class to be prefixed
 * @param {Object} obj - object to be checked
 * @return {class} prefixed class
 */
function prefixedClass(prop, obj) {
  return obj[Modernizr.prefixed(prop, obj, false)];
}

export default {
  name: 'SpeechRecognition',
  props: {
    options: Array,
  },
  methods: {
    recognize() {
      const SpeechRecognition = prefixedClass('SpeechRecognition', window);
      const SpeechGrammarList = prefixedClass('SpeechGrammarList', window);

      const map = new Map(this.options);
      const names = [...map.keys()];
      const grammar = `
      #JSGF V1.0;
      grammar option;
      public <option> = '${names.join('|')}';
      `;

      const list = new SpeechGrammarList();
      list.addFromString(grammar, 1);

      const recognition = new SpeechRecognition();
      recognition.lang = 'cmn-Hans-CN';
      recognition.interimResults = false;
      recognition.maxAlternatives = 1;
      // It seems not working, maybe because non-ascii?
      // recognition.grammars = list;

      recognition.onresult = (event) => {
        const r = event.results[event.results.length - 1][0];
        this.$emit('result', r.transcript);
        console.warn(`${r.transcript} with confidence ${r.confidence}`);
      };

      recognition.onspeechend = () => {
        recognition.stop();
      };

      recognition.onnomatch = (event) => {
        Materialize.toast({html: '识别失败'});
      };

      recognition.onerror = (event) => {
        Materialize.toast({html: event.error});
      };

      recognition.start();
    },
  },
};
</script>
