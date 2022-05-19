<template>
  <div
    ref="modal"
    v-on:keyup.ctrl.space="(nav = !nav), (option_4 = false)"
    v-on:click="
      (option_4 = false), (nav = false), (option_1 = false), (option_2 = false)
    "
    tabindex="-1"
  >
    <textarea
      id="editor"
      placeholder="# Start Typing..."
      v-model="input"
      @input="render"
      spellcheck="false"
    />
    <div class="result" v-html="output" />
  </div>

  <div class="nav" v-if="nav">
    <a class="option" v-on:click="save">Save When Done</a>
    <a class="option" v-on:click="(option_2 = !option_2), (nav = false)"
      >Load</a
    >
    <a class="option">Change Theme</a>
    <a class="option" v-on:click="(option_4 = !option_4), (nav = false)">
      Import Raw Files From Github
    </a>
    <a class="option" v-on:click="guide">Guide</a>
    <p class="footer">Made with ❤️  VueJS, Source code availabe on github.</p>
  </div>

  <div class="option_2" v-if="option_2">
    <vSelect
      class="vselect"
      :options="loadLink"
      placeholder="Choose Files"
      v-model="selected"
    />
    <button class="button-8" role="button" @click="load()">Load</button>
  </div>

  <div class="option_4" v-if="option_4">
    <input
      placeholder="Enter URL"
      v-model="File_URL"
      v-on:keyup.enter="getURL"
      spellcheck="false"
    />
  </div>
</template>

<script>
import { marked } from "marked";
const pastemyst = require("pastemyst-js");
const expirationTime = "never";
const language = "markdown";
import vSelect from "vue-select";
import "vue-select/dist/vue-select.css";
import fetch from "node-fetch";

export default {
  name: "App",
  components: {
    vSelect,
  },
  data() {
    return {
      input: `# _Goo Guide_:
- Goo is an open-source and mini web-based markdown editor made with VueJs.

- Goo uses a command palate as a navbar. The command palate can be viewed by combining **CTRL + SPACE** keys.

- Save the file once when you finish it.`,
      output: "",
      nav: false,
      option_4: false,
      File_URL: "",
      option_1: false,
      loadLink: [],
      option_2: false,
      selected: "",
    };
  },
  methods: {
    render: function () {
      this.output = marked(this.input);
    },
    getURL: function () {
      try {
        fetch(this.File_URL)
          .then((res) => res.text())
          .then((data) => {
            this.input = data;
            this.output = marked(this.input);
            this.option_4 = !this.option_4;
          });
      } catch (err) {
        console.log("fetch error");
        this.File_URL = "ERROR";
      }
    },
    async load() {
      const response = await fetch(
        `https://paste.myst.rs/api/v2/paste/${this.selected}`
      );
      const data = await response.json();
      this.input = data.pasties[0].code;
      this.output = marked(this.input);
    },
    save: function () {
      pastemyst
        .createPasteMyst(this.input, expirationTime, language)
        .then((pasteMystInfo) => {
          console.log("success");
          console.log(pasteMystInfo.id);
          console.log(pasteMystInfo.link); // URL to the PasteMyst entry
          console.log(pasteMystInfo.createdAt); // Creation date Linux seconds
          console.log(pasteMystInfo.date); // 'Created at' date, UTC
          console.log(pasteMystInfo.code);
          console.log(pasteMystInfo.expiresIn);
          console.log(pasteMystInfo.language);
          this.loadLink.push(pasteMystInfo.id);
          this.input += "\r\n" + "FileName:" + pasteMystInfo.id;
          this.output = marked(this.input);
        })
        .catch((err) => {
          console.log("failure");
          console.error(err);
        });
    },
    guide: function () {
      this.input = `# _Goo Guide_:
- Goo is an open-source and mini web-based markdown editor made with VueJs.

- Goo uses a command palate as a navbar. The command palate can be viewed by combining **CTRL + SPACE** keys.

- Save the file once when you finish it.`;
      this.output = marked(this.input);
    },
  },
  mounted() {
    this.$refs.modal.focus();
    if (localStorage.getItem("ID"))
      this.loadLink = JSON.parse(localStorage.getItem("ID"));
    this.output = marked(this.input);
  },
  watch: {
    loadLink: {
      handler() {
        localStorage.setItem("ID", JSON.stringify(this.loadLink));
      },
      deep: true,
    },
  },
};
</script>
