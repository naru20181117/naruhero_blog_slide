<template>
  <div id="app">
    <div class="reveal">
      <div class="slides">
        <section>
          <h2>{{title }}</h2>
          <img :src="image">
        </section>
        <section v-for="line in lines" :key="line">
          <section><vue-markdown-it :source="line" /></section>
        </section>
        <end></end>
      </div>
    </div>
  </div>
</template>

<script>
import Reveal from 'reveal.js'
import Markdown from 'reveal.js/plugin/markdown/markdown.esm.js';
import End from './components/End';
import { createClient } from "@/plugins/contentful";
import VueMarkdownIt from 'vue3-markdown-it';

export default {
  name: 'app',
  props: {
    content: {
      type: String,
    }
  },
  components: {
    End,
    VueMarkdownIt
  },
  data() {
    return {
      title: null,
      body: null,
      lines: [],
    };
  },
  mounted() {
    Reveal.initialize({
      plugins: [ Markdown ],
      markdown: {
        smartypants: true
      }
    })
  },
  async created() {
    let uri = window.location.search.substring(1)
    let params = new URLSearchParams(uri)
    await createClient()
      .getEntry(
        params.get("entry_id")
      )
      .then(post => {
        this.post = post
        this.title = post.fields.title
        this.body = post.fields.body
        this.lines = this.body.split(/\r?\n---\r?\n/)
        this.image = post.fields.postImage.fields.file.url
      });
  }
}
</script>

<style>
@import url('../node_modules/reveal.js/dist/reveal.css');
@import url('../node_modules/reveal.js/dist/theme/black.css');
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
}

#app section {
  max-height: 100vh !important;
}

img {
  width: 250px;
}
</style>
