<template>
  <span :title="data.description">
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
        "https://cdn.jsdelivr.net/gh/qinhua/cdn_assets@master/comment/emoji/smilies/",
    },
  },
  computed: {
    tiebaSrc() {
      return this.url + "icon_" + this.data.name + ".gif";
    },
  },
};
</script>

