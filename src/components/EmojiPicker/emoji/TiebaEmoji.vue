<template>
  <span class="emoji-item" :title="data.description">
    <img :src="tiebaSrc" />
  </span>
</template>

<script>
export default {
  name: "TiebaEmoji",
  props: {
    data: {
      type: Object,
    },
    url: {
      type: String,
      required: false,
      default:
        `${process.env.BASE_URL}assets/emoji/smilies/`,
    },
  },
  computed: {
    tiebaSrc() {
      return this.url + "icon_" + this.data.name + ".gif";
    },
  },
};
</script>

