<template>
  <div class="container">
    <div>
      <logo />
      <div class="my-container">
        <div class="backdrop">
          <HighlightText
            class="my-highlight"
            highlightClassName="highlight"
            :queries="list"
          >{{text}}</HighlightText>
        </div>

        <b-form-textarea
          type="text"
          ref="form"
          class="myForm"
          id="textarea"
          v-model="text"
          placeholder="Enter something..."
          rows="6"
          cols="12"
        ></b-form-textarea>

        <div
          class="suggest-list"
          v-if="listFilter.word"
          :style="{top: top + 'em', left: left + 'em'}"
        >
          <scrollBar class="wrap" barYMinHeight="2" :scrollTrackStyle="scrollTrackStyle">
            <b-list-group>
              <b-list-group-item
                class="suggest-word"
                v-for="(item, index) in listFilter.list"
                :key="index"
                @click="chooseWord(item)"
              >{{item}}</b-list-group-item>
            </b-list-group>
          </scrollBar>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Logo from "~/components/Logo.vue";

export default {
  components: {
    Logo
  },
  data() {
    return {
      isMounted: false,
      text: "",
      list: ["full name", "name", "last name", "phone", "email"],
      scrollTrackStyle: {
        backgroundColor: "white"
      },
      top: 2,
      left: 0
    };
  },
  computed: {
    // startCursor() {

    // }
    listFilter() {
      if (!this.isMounted)
        return {
          word: "",
          list: this.list
        };
      else if (!this.text)
        return {
          word: "",
          list: this.list
        };
      else {
        //insert word before cursor
        let beforeWord = "";
        //insert word after cursor
        let afterWord = "";

        // cursor position
        let startCursor = this.$refs.form.selectionStart;

        //change position menu
        this.left = 0.45 * (startCursor % 135);
        this.top = 2 + startCursor / 135;

        //Last space before start cursor
        let lastSpace = this.text.lastIndexOf(" ", startCursor - 1);
        //last line break before start cursor
        let lastLineBreak = this.text.lastIndexOf("\n", startCursor - 1);
        // first space after start cursor
        let firstSpace = this.text.indexOf(" ", startCursor);
        //first line break after start cursor
        let firstLineBreak = this.text.indexOf("\n", startCursor);

        //find insert word before cursor
        if (lastSpace === -1 && lastLineBreak === -1)
          beforeWord = this.text.slice(0, startCursor);
        else if (lastSpace === -1) {
          beforeWord = this.text.slice(lastLineBreak + 1, startCursor);
        } else if (lastLineBreak === -1) {
          beforeWord = this.text.slice(lastSpace + 1, startCursor);
        } else {
          beforeWord =
            lastSpace < lastLineBreak
              ? this.text.slice(lastLineBreak + 1, startCursor)
              : this.text.slice(lastSpace + 1, startCursor);
        }

        //find insert word after cursor
        if (firstSpace === -1 && firstLineBreak === -1)
          afterWord = this.text.slice(startCursor);
        else if (firstSpace === -1) {
          afterWord = this.text.slice(startCursor, firstLineBreak);
        } else if (firstLineBreak === -1) {
          afterWord = this.text.slice(startCursor, firstSpace);
        } else {
          afterWord =
            firstSpace < firstLineBreak
              ? this.text.slice(startCursor, firstSpace)
              : this.text.slice(startCursor, firstLineBreak);
        }
        let word = beforeWord + afterWord;
        return {
          word: word,
          list: this.list.filter(item => item.includes(word)),
          beforeWord: beforeWord,
          afterWord: afterWord,
          cursor: startCursor
        };
      }
    }
  },

  methods: {
    chooseWord(word) {
      let beforeInsertWord = this.text.slice(
        0,
        this.listFilter.cursor - this.listFilter.beforeWord.length
      );
      let afterInsertWord = this.text.slice(
        this.listFilter.cursor + this.listFilter.afterWord.length
      );
      this.text = beforeInsertWord + word + afterInsertWord;
      this.$refs.form.focus();
    }
  },
  mounted() {
    this.isMounted = true;
  }
};
</script>

<style>
.my-container {
  position: relative;
}

.my-container .backdrop {
  display: block;
  position: absolute;
  left: 0.8em;
  top: 0.4em;
  color: transparent;
  background-color: transparent;
  font-size: 1em;
  white-space: pre-wrap;
  /* z-index: 1; */
}

.my-container .myForm {
  display: inline-block;
  position: absolute;
  left: 0;
  top: 0;
  color: black;
  /* color: transparent; */
  background-color: transparent;
  font: 1em sans-serif;
  line-height: 1.45em;
  /* z-index: 1; */
  letter-spacing: 0;
  word-spacing: normal;
}

.my-highlight,
.my-highlight * {
  padding: 0;
  margin: 0;
  font: 1em;
  color: transparent;
  word-spacing: normal;
  letter-spacing: 0;
  white-space: pre-wrap;
}

.highlight {
  letter-spacing: 0;
  color: transparent;
  background-color: yellow;
  padding: 0;
  margin: 0;
  word-spacing: normal;
  white-space: pre-wrap;
}

.suggest-list {
  position: absolute;
}

.suggest-word {
  height: 2em;
}

.suggest-word:hover {
  height: 2em;
  background-color: rgb(82, 255, 189);
}

.wrap {
  height: 6em;
  width: 8em;
}
</style>
