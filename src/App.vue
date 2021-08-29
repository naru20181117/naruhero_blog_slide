<template>
  <div id="app">
    <div class="reveal">
      <div class="slides">
        <section>
          <h2>{{ title }}</h2>
          <img :src="image">
        </section>
        <section v-for="line in lines" :key="line">
          <section>
            <vue-markdown-it
              :html="true"
              :breaks="true"
              :linkify="true"
              :source="line"
            />
          </section>
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
  async created() {
    let uri = window.location.search.substring(1)
    let params = new URLSearchParams(uri)
    await createClient()
      .getEntry(
        // ブログからパラメーターを投げて表示
        params.get("entry_id")
      )
      .then(post => {
        this.title = post.fields.title
        this.body = post.fields.body
        this.lines = this.body.split(/\r?\n---\r?\n/)
        this.image = post.fields.postImage.fields.file.url
      });
  },
  mounted() {
    Reveal.initialize({
      plugins: [ Markdown ],
      markdown: {
        smartypants: true
      }
    })
  },
}
</script>

<style>
@import url('../node_modules/reveal.js/dist/reveal.css');
/* ここでテーマを選ぶ */
@import url('../node_modules/reveal.js/dist/theme/beige.css');

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
  max-height: 350px !important;
  max-width: 450px !important;
}

/* 部分的に大きくしたい画像にはimg-largeを仕込む */
.img-large {
  max-height: 550px !important;
  max-width: 850px !important;
}
</style>
