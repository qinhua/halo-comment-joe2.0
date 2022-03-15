<template>
  <span class="emoji-item" :title="data.description">
    <img :src="aluSrc" />
  </span>
</template>

<script>
export default {
  name: "GuluEmoji",
  props: {
    data: {
      type: Object,
    },
    url: {
      type: String,
      required: false,
      default:
        `${process.env.BASE_URL}assets/emoji/gulu/`,
    },
  },
  computed: {
    aluSrc() {
      return this.url + "icon_" + this.data.name.replace('gulu-','') + ".png";
    },
  },
};
</script>

