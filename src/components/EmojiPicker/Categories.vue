<template>
  <transition name="category">
    <table class="motion-switcher-table">
      <tbody>
        <tr>
          <th
            v-for="(categorie, index) in categories"
            :class="[
              'category',
              categorie.name + '-box',
              { active: index === active },
              { 'on-hover': index === active },
            ]"
            :key="index"
            @click="onSelect(index)"
          >
            {{ categorie.title }}
          </th>
        </tr>
      </tbody>
    </table>
  </transition>
</template>

<script>
export default {
  name: "Categories",
  data: () => ({
    categories: [
      { name: 'haha', title: '泡泡' },
      { name: 'bilibili', title: 'B站' },
      // { name: 'gulu', title: '咕噜' },
      // { name: 'tieba', title: '贴吧' },
      { name: 'menhera', title: '颜文字' },
    ],
    active: 0,
  }),
  methods: {
    onSelect(index) {
      this.active = index;
      const _category = this.categories[index];
      this.$emit("select", _category);
    },
  },
};
</script>
