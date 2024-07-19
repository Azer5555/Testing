<template>
  <div class="pagination">
    <button @click="changePage(currentPage - 1)" :disabled="currentPage === 1">
      Назад
    </button>
    <button
      v-for="page in totalPages"
      :key="page"
      @click="changePage(page)"
      :class="{ active: page === currentPage }"
    >
      {{ page }}
    </button>
    <button
      @click="changePage(currentPage + 1)"
      :disabled="currentPage === totalPages"
    >
      Вперед
    </button>
  </div>
</template>

<script>
export default {
  props: {
    total: {
      type: Number,
      required: true,
    },
    pageSize: {
      type: Number,
      required: true,
    },
  },
  data() {
    return {
      currentPage: 1,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.total / this.pageSize);
    },
  },
  methods: {
    changePage(page) {
      if (page >= 1 && page <= this.totalPages) {
        this.currentPage = page;
        this.$emit("page-changed", page);
      }
    },
  },
};
</script>

<style scoped>
.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

button {
  padding: 10px;
  margin: 0 5px;
  background-color: #008000;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

button.active {
  background-color: #006400;
}
</style>
