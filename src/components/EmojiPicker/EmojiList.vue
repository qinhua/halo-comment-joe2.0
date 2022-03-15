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
import HahaEmoji from './emoji/HahaEmoji'
import BilibiliEmoji from './emoji/BilibiliEmoji'
// import GuluEmoji from './emoji/GuluEmoji'
// import TiebaEmoji from './emoji/TiebaEmoji'
import MenheraEmoji from './emoji/MenheraEmoji'

export default {
  name: 'EmojiList',
  components: {
    HahaEmoji,
    BilibiliEmoji,
    // GuluEmoji,
    // TiebaEmoji,
    MenheraEmoji
  },
  data: () => ({
    categories: [
      { name: 'haha', title: '泡泡' },
      { name: 'bilibili', title: 'B站' },
      { name: 'gulu', title: '咕噜' },
      // { name: 'tieba', title: '贴吧' },
      { name: 'menhera', title: '颜文字' },
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
      } else if(["haha","gulu","tieba"].includes(this.category)) {
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