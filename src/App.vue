<template>
  <div id="app-container">
    <h1>Справочник организаций</h1>
    <div class="controls">
      <input v-model="query" placeholder="Найти по ФИО" class="search-input" />
      <button @click="isAddModalVisible = true" class="add-button">
        Добавить
      </button>
    </div>
    <table class="org-table">
      <thead>
        <tr>
          <th @click="sortTableBy('name')">Название</th>
          <th
            @click="sortTableBy('director')"
            :class="{
              'sorted-asc': sortColumn === 'director' && sortDirection === 1,
              'sorted-desc': sortColumn === 'director' && sortDirection === -1,
            }"
          >
            ФИО директора
            <span v-if="sortColumn === 'director'">
              <span v-if="sortDirection === 1"> ▲</span>
              <span v-else> ▼</span>
            </span>
          </th>
          <th>Номер телефона</th>
          <th>Действия</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(org, index) in paginatedOrgs" :key="org.id">
          <td>{{ org.name }}</td>
          <td>{{ org.director }}</td>
          <td>{{ org.phone }}</td>
          <td>
            <button @click="removeOrg(index)" class="delete-button">✖</button>
          </td>
        </tr>
      </tbody>
    </table>
    <pagination
      :total="filteredOrgs.length"
      :page-size="itemsPerPage"
      @page-changed="handlePageChange"
    />
    <modal
      v-if="isAddModalVisible"
      @close="isAddModalVisible = false"
      @add="addOrg"
    />
  </div>
</template>

<script>
import Pagination from "./components/Pagination.vue";
import Modal from "./components/Modal.vue";

export default {
  components: {
    Pagination,
    Modal,
  },
  data() {
    return {
      orgs: [], // Данные организации
      query: "",
      sortColumn: "", // Ключ сортировки
      sortDirection: 1, // Первоначально в порядке возрастания
      currentPageNumber: 1, // Страница по умолчанию
      itemsPerPage: 5, // Количество элементов на странице
      isAddModalVisible: false,
    };
  },
  mounted() {
    //  Загрузка данных организации из localStorage при монтировании компонента
    const savedOrgs = localStorage.getItem("organizations");
    if (savedOrgs) {
      this.orgs = JSON.parse(savedOrgs);
    }
  },
  computed: {
    filteredOrgs() {
      const searchQuery = this.query.trim().toLowerCase();
      if (!searchQuery) {
        // Проверка наличия поискового запроса
        return this.orgs; // Вернуть все организации, если нет поискового запроса
      }
      return this.orgs.filter((org) =>
        org.director.toLowerCase().includes(searchQuery)
      );
    },
    sortedOrgs() {
      const orgs = [...this.filteredOrgs];
      if (this.sortColumn === "director") {
        orgs.sort((a, b) => {
          // сортировка по возрастанию и убыванию
          const directorA = a.director.toLowerCase();
          const directorB = b.director.toLowerCase();
          return this.sortDirection === 1
            ? directorA.localeCompare(directorB)
            : directorB.localeCompare(directorA);
        });
      }
      return orgs;
    },
    paginatedOrgs() {
      const startIndex = (this.currentPageNumber - 1) * this.itemsPerPage;
      return this.sortedOrgs.slice(startIndex, startIndex + this.itemsPerPage);
    },
    totalPagesCount() {
      return Math.ceil(this.filteredOrgs.length / this.itemsPerPage); //Вычисление общего количества страниц
    },
  },
  methods: {
    sortTableBy(column) {
      if (this.sortColumn === column) {
        this.sortDirection = -this.sortDirection; // Переключить направление сортировки, если щелкнуть тот же столбец
      } else {
        this.sortColumn = column; // Изменить столбец сортировки
        this.sortDirection = 1; // По умолчанию - восходящее направление
      }
    },
    handlePageChange(page) {
      this.currentPageNumber = page; // Обновить текущую страницу при изменении нумерации страниц
    },
    addOrg(newOrg) {
      this.orgs.push(newOrg); // Добавление новой организации
      this.saveOrgs(); // Сохранение организации в localStorage
      this.isAddModalVisible = false; // Закрытие модального окна
    },
    removeOrg(index) {
      const actualIndex =
        (this.currentPageNumber - 1) * this.itemsPerPage + index;
      this.orgs.splice(actualIndex, 1); // Удалить организацию, используя фактический индекс
      this.saveOrgs();
      if (this.paginatedOrgs.length === 0 && this.currentPageNumber > 1) {
        this.currentPageNumber--; // Перейти на предыдущую страницу, если на текущей странице не осталось элементов
      }
      if (this.currentPageNumber > this.totalPagesCount) {
        this.currentPageNumber = this.totalPagesCount; // Текущую страница, если она превышает общее количество страниц.
      }
    },
    saveOrgs() {
      localStorage.setItem("organizations", JSON.stringify(this.orgs));
    },
  },
};
</script>

<style scoped>
#app-container {
  font-family: Arial, sans-serif;
  margin: 20px;
}

h1 {
  color: #333;
}
.controls {
  display: flex;
  justify-content: space-between;
}

.search-input {
  margin-bottom: 20px;
  padding: 10px;
  width: 200px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.add-button {
  padding: 10px 20px;
  margin-bottom: 20px;
  background-color: #0000ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.add-button:hover {
  background-color: #00008b;
}

.org-table {
  width: 100%;
  border-collapse: collapse;
}

.org-table th,
.org-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.org-table th {
  background-color: #f2f2f2;
  cursor: pointer;
  text-align: center;
}

.org-table th:nth-child(4),
.org-table td:nth-child(4) {
  width: 50px;
  text-align: center;
}
.delete-button {
  padding: 5px 10px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.delete-button:hover {
  background-color: #c82333;
}
</style>
