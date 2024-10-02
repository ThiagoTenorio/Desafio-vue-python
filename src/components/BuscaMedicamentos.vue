<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Busca de Medicamentos</h1>

    <!-- Formulário de Filtros -->
    <form @submit.prevent="fetchData" class="d-flex align-items-center">
      <div class="input-group flex-grow-1">
        <input
          type="text"
          v-model="searchTerm"
          placeholder="Buscar..."
          class="form-control"
          aria-label="Buscar"
        />
        <select
          v-model="selectedFilter"
          class="form-select"
          aria-label="Selecione um filtro"
        >
          <option value="all">Todos</option>
          <option value="substance">Substância</option>
          <option value="cnpj">CNPJ</option>
          <option value="laboratory">Laboratório</option>
        </select>
        <button type="submit" class="btn btn-primary">Buscar</button>
      </div>
    </form>

    <!-- Listagem de Resultados Paginada -->
    <div v-if="results.length > 0" class="mt-4">
      <ul class="list-group">
        <li
          class="list-group-item"
          v-for="(item, index) in paginatedResults"
          :key="index"
        >
          {{ item.name }} - {{ item.substance }} - {{ item.cnpj }} -
          {{ item.laboratory }}
        </li>
      </ul>

      <!-- Paginação -->
      <div class="d-flex justify-content-between mt-3">
        <button
          @click="previousPage"
          class="btn btn-secondary"
          :disabled="currentPage === 1"
        >
          Anterior
        </button>
        <span>Página {{ currentPage }} de {{ totalPages }}</span>
        <button
          @click="nextPage"
          class="btn btn-secondary"
          :disabled="currentPage === totalPages"
        >
          Próxima
        </button>
      </div>
    </div>

    <!-- Mensagem de sem resultados -->
    <p v-else class="mt-3 text-center">Nenhum resultado encontrado</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchTerm: "",
      selectedFilter: "all",
      results: [],
      currentPage: 1,
      perPage: 10,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.results.length / this.perPage);
    },
    paginatedResults() {
      const start = (this.currentPage - 1) * this.perPage;
      const end = start + this.perPage;
      return this.results.slice(start, end);
    },
  },
  methods: {
    async fetchData() {
      const apiUrl = "https://fakeapi.com/medicines"; // Substitua pela URL da sua API

      const params = {
        search: this.searchTerm,
      };

      if (this.selectedFilter !== "all") {
        params.filter = this.selectedFilter;
      }

      try {
        const response = await fetch(
          `${apiUrl}?${new URLSearchParams(params)}`
        );
        const data = await response.json();
        this.results = data;
        this.currentPage = 1;
      } catch (error) {
        console.error("Erro ao buscar dados:", error);
        this.results = [];
      }
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    previousPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
  },
};
</script>

<style>
.container {
  max-width: 600px;
}

.list-group-item {
  transition: background-color 0.3s;
}

.list-group-item:hover {
  background-color: #f1f1f1;
}
</style>
