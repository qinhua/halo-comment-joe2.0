<template>
  <div id="container-emoji" class="motion-container" :class="categoryClass">
     <component
      :is="categoryEmoji" 
      v-for="(emoji, index) in data[category]"
      :key="index"
      :data="emoji"
      @click.native="onSelect(emoji, type)"
    />
  </div>
</template>

<script>
import HahaEmoji from './HahaEmoji'
import BilibiliEmoji from './BilibiliEmoji'
// import TiebaEmoji from './TiebaEmoji'
import MenheraEmoji from './MenheraEmoji'

export default {
  name: 'EmojiList',
  components: {
    HahaEmoji,
    BilibiliEmoji,
    // TiebaEmoji,
    MenheraEmoji
  },
  data: () => ({
    categories: [
      { name: 'haha', title: 'Haha' },
      { name: 'bilibili', title: 'Bilibili' },
      // { name: 'tieba', title: 'Tieba' },
      { name: 'menhera', title: '(✪ω✪)' },
    ]
  }),
  props: {
    data: {type: Object},
    category: { type: String }
  },
  methods: {
    onSelect(emoji, type) {
      this.$emit('select', emoji, type)
    },
  },
  computed: {
    categoryClass() {
      return this.category + "-container";
    },
    categoryEmoji() {
      return this.category + "Emoji";
    },
    type() {
      if(this.category === "bilibili") {
        return "Math";
      } else if(["tieba","haha"].includes(this.category)) {
        return "BBCode"
      }
      return "";
    }
  },
  watch: {
    data() {
      this.$refs['container-emoji'].scrollTop = 0
    }
  }
}
</script>