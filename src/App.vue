<template>
  <div id="app">
    <!-- <input type="text" v-model="js" /> -->
    <input placeholder="input" v-model="testStr" @keyup.enter="setInput" />
    <div style="background: #242424" v-if="logs.length">
      <console-feed
        :logs="logs"
        variant="dark"
        :styles="{ BASE_FONT_SIZE: '20px' }"
      />
    </div>
  </div>
</template>

<script>
/*eslint-disable*/
import { ReactInVue } from "vuera";
import { Hook, Decode } from "console-feed";
import { Console } from "console-feed";
export default {
  name: "App",
  components: {
    "console-feed": ReactInVue(Console),
  },
  data() {
    return {
      iframe: null,
      logs: [],
      js: "console.log('hey')",
      testStr: "",
      debounceFn: null,
    };
  },
  watch: {
    js: {
      handler() {
        this.debounceFn();
      },
    },
  },
  mounted() {
    this.iframe = document.createElement("iframe");
    this.debounceFn = this.debounce(() => {
      this.updateIframe();
    }, 2000);
    this.debounceFn();
  },
  methods: {
    debounce(func, wait, immediate) {
      var timeout;

      return function executedFunction() {
        var context = this;
        var args = arguments;

        var later = function() {
          timeout = null;
          if (!immediate) func.apply(context, args);
        };

        var callNow = immediate && !timeout;

        clearTimeout(timeout);

        timeout = setTimeout(later, wait);

        if (callNow) func.apply(context, args);
      };
    },
    updateIframe() {
      this.iframe.srcdoc = "<script>" + this.js + "<\/script>";
      document.body.appendChild(this.iframe);

      Hook(this.iframe.contentWindow.console, (log) => {
        this.logs = [...this.logs, Decode(log)];
        // window.postMessage(log, "*");
      });
      // window.addEventListener("message", (log) => {
      //   Decode(log);
      // });
    },
    setInput() {
      let template = `\nconsole.log(${this.testStr})`;
      this.js += template;
      this.updateIframe();
      this.testStr = "";
    },
  },
};
</script>

<style></style>
