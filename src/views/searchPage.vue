<template>
  <div class="search-container">
    <input
      type="text"
      id="searchInput"
      v-model="keyword"
      placeholder="搜索"
      @keypress.enter="search"
    />
  </div>
</template>

<script>
export default {
  name: 'Search',
  components: {},
  data() {
    return { keyword: '' };
  },
  created() {},
  methods: {
    search() {
      this.$router.push({ name: 'search', params: { keywords: this.keyword } });
    },
  },
};
</script>

<style lang="scss" scoped>
h1 {
  margin-top: 32px;
  margin-bottom: 28px;
  color: var(--color-text);

  span {
    opacity: 0.58;
  }
}

.search-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 80vh;
  width: 70vw;
  margin-left: 10vw;
}

#searchInput {
  // color: #4c0013;
  padding-left: 50px;
  width: 100%;
  height: 80px;
  border: none;
  border-radius: 30px;
  outline: none;
  font-size: 26px;
  font-weight: bold;
  color: #15080b;
  background-color: #e8f0fe;
  // box-shadow: 0 2px 2px rgba(0, 0, 0, 0.1);
}

#searchInput::placeholder {
  color: #444746;
  /* Placeholder text color */
  font-style: italic;
  /* Placeholder text style, e.g., italic */
  font-weight: bold;
}

#searchInput:focus {
  // background-color: #444746;
  box-shadow: 0 1px 1px rgba(0, 0, 0, 0.1);
  // outline: olive;
}
</style>
