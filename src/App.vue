<template>
  <div id="app">
    <h1>Справочник организаций</h1>
    <div class="knop">
      <input
        v-model="searchQuery"
        placeholder="Найти по ФИО"
        class="search-input"
      />
      <button @click="showAddModal = true" class="add-button">Добавить</button>
    </div>
    <table class="organization-table">
      <thead>
        <tr>
          <th @click="sortBy('name')">Название</th>
          <th
            @click="sortBy('director')"
            :class="{
              'sorted-asc': sortKey === 'director' && sortOrder === 1,
              'sorted-desc': sortKey === 'director' && sortOrder === -1,
            }"
          >
            ФИО директора
            <span v-if="sortKey === 'director'">
              <span v-if="sortOrder === 1"> ▲</span>
              <span v-else> ▼</span>
            </span>
          </th>
          <th>Номер телефона</th>
          <th>Действия</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="(organization, index) in paginatedData"
          :key="organization.id"
        >
          <td>{{ organization.name }}</td>
          <td>{{ organization.director }}</td>
          <td>{{ organization.phone }}</td>
          <td>
            <button @click="removeOrganization(index)" class="delete-button">
              ✖
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <pagination
      :total="filteredData.length"
      :page-size="pageSize"
      @page-changed="onPageChange"
    />
    <add-modal
      v-if="showAddModal"
      @close="showAddModal = false"
      @add="addOrganization"
    />
  </div>
</template>

<script>
import Pagination from "./components/Pagination.vue";
import AddModal from "./components/Modal.vue";

export default {
  components: {
    Pagination,
    AddModal,
  },
  data() {
    return {
      organizations: [], // Initialize with your organizations data
      searchQuery: "",
      sortKey: "", // Initially no sorting key
      sortOrder: 1, // Initially ascending order
      currentPage: 1,
      pageSize: 2,
      showAddModal: false,
    };
  },
  mounted() {
    // Load organizations data from localStorage on component mount
    const savedOrganizations = localStorage.getItem("organizations");
    if (savedOrganizations) {
      this.organizations = JSON.parse(savedOrganizations);
    }
  },
  computed: {
    filteredData() {
      const query = this.searchQuery.trim().toLowerCase();
      if (!query) {
        return this.organizations; // Return all organizations if no search query
      }
      return this.organizations.filter((org) =>
        org.director.toLowerCase().includes(query)
      );
    },
    sortedData() {
      const data = [...this.filteredData]; // Copy to avoid mutating original array
      if (this.sortKey === "director") {
        data.sort((a, b) => {
          const nameA = a.director.toLowerCase();
          const nameB = b.director.toLowerCase();
          return this.sortOrder === 1
            ? nameA.localeCompare(nameB)
            : nameB.localeCompare(nameA);
        });
      }
      return data;
    },
    paginatedData() {
      const start = (this.currentPage - 1) * this.pageSize;
      return this.sortedData.slice(start, start + this.pageSize);
    },
    totalPages() {
      return Math.ceil(this.filteredData.length / this.pageSize);
    },
  },
  methods: {
    sortBy(key) {
      if (this.sortKey === key) {
        this.sortOrder = -this.sortOrder; // Toggle sort order if same key clicked
      } else {
        this.sortKey = key; // Change sort key
        this.sortOrder = 1; // Default to ascending order
      }
    },
    onPageChange(page) {
      this.currentPage = page; // Update current page on pagination change
    },
    addOrganization(organization) {
      this.organizations.push(organization); // Add new organization
      this.saveOrganizations(); // Save organizations to localStorage
      this.showAddModal = false; // Close modal after adding
    },
    removeOrganization(index) {
      const actualIndex = (this.currentPage - 1) * this.pageSize + index;
      this.organizations.splice(actualIndex, 1); // Remove organization using actual index
      this.saveOrganizations(); // Save organizations to localStorage
      if (this.paginatedData.length === 0 && this.currentPage > 1) {
        this.currentPage--; // Move to previous page if no items left on current page
      }
      if (this.currentPage > this.totalPages) {
        this.currentPage = this.totalPages; // Correct current page if it exceeds total pages
      }
    },
    saveOrganizations() {
      localStorage.setItem("organizations", JSON.stringify(this.organizations));
    },
  },
};
</script>

<style scoped>
#app {
  font-family: Arial, sans-serif;
  margin: 20px;
}

h1 {
  color: #333;
}
.knop {
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

.organization-table {
  width: 100%;
  border-collapse: collapse;
}

.organization-table th,
.organization-table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

.organization-table th {
  background-color: #f2f2f2;
  cursor: pointer;
  text-align: center;
}

.organization-table th:nth-child(4),
.organization-table td:nth-child(4) {
  width: 50px; /* Устанавливаем фиксированную ширину столбца */
  text-align: center; /* Выравниваем текст по центру */
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
