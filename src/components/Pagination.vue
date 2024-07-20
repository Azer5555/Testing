<template>
  <div class="pagination-controls">
    <button
      @click="goToPage(currentPageNumber - 1)"
      :disabled="currentPageNumber === 1"
    >
      Назад
    </button>
    <button
      v-for="page in totalPagesCount"
      :key="page"
      @click="goToPage(page)"
      :class="{ active: page === currentPageNumber }"
    >
      {{ page }}
    </button>
    <button
      @click="goToPage(currentPageNumber + 1)"
      :disabled="currentPageNumber === totalPagesCount"
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
      currentPageNumber: 1, // Страница по умолчанию
    };
  },
  computed: {
    totalPagesCount() {
      return Math.ceil(this.total / this.pageSize); //Сколько нам потребуется страниц чтобы заполнить
    },
  },
  methods: {
    goToPage(page) {
      if (page >= 1 && page <= this.totalPagesCount) {
        this.currentPageNumber = page;
        this.$emit("page-changed", page);
      }
    },
  },
};
</script>

<style scoped>
.pagination-controls {
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
