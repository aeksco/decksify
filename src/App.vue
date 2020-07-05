<template>
  <div id="app">
    <i class="fa fa-spin fa-spinner fa-lg" v-if="fetching"></i>
    <div class="reveal" v-else>
      <div class="slides">
        <section v-for="slide in slides" :key="slide.id">
          <div v-html="slide.content" v-if="slide.type === 'HTML'"></div>
          <div v-html="renderMarkdown(slide.content)" v-else></div>
        </section>
      </div>
    </div>
  </div>
</template>

<script>
import marked from 'marked'
import Reveal from 'reveal.js/js/reveal'
const uniqueId = require('lodash/uniqueId')
const htmlTag = '<!-- HTML -->'

export default {
  props: ['url', 'revealOptions'], // TODO - provide default prop values
  async created () {

    const url = 'http://localhost:3003/PITCHME.md'
    console.log(this)
    this.fetching = false
    let response = await fetch(url)
    let raw = await response.text()
    this.raw = raw
    this.fetching = false
    setTimeout(() => {
      Reveal.initialize(this.revealOptions)
    }, 500)
  },
  data () {
    return {
      fetching: true,
      raw: ''
    }
  },
  computed: {
    compiledMarkdown: function () {
      return marked(this.input, { sanitize: true })
    },
    slides () {
      return this.raw.split('---').map((content) => {
        let dirty = true

        while (dirty) {
          if (content.startsWith('\n')) {
            content = content.slice(1, content.length)
          } else if (content.startsWith('\n')) {
            content = content.slice(0, content.length - 2)
          } else {
            dirty = false
          }
        }

        // console.log(content[0])
        if (content.startsWith(htmlTag)) {
          return {
            type: 'HTML',
            content: content.slice(htmlTag.length + 2, content.length),
            id: uniqueId()
          }
        }

        return {
          id: uniqueId(),
          type: 'MD',
          content
        }
      }).filter((e) => {
        // console.log(e.content)
        return e.content !== ''
      })
    }
  },
  methods: {
    renderMarkdown (input) {
      return marked(input, { sanitize: true })
    }
  }
}
</script>

<style>
@import url('reveal.js/css/reveal.css');

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  /* margin-top: 60px; */
  height: 100vh;
}
</style>
