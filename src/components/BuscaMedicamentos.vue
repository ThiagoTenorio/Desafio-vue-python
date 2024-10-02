<template>
  <div>
    <h1>Busca de Medicamentos</h1>

    <!-- Formulário de Filtros -->
    <form @submit.prevent="fetchData">
      <select v-model="selectedFilter">
        <option value="all">Todos</option>
        <option value="substance">Substância</option>
        <option value="cnpj">CNPJ</option>
        <option value="laboratory">Laboratório</option>
      </select>
      <input type="text" v-model="searchTerm" placeholder="Buscar..." />

      <!-- Botão de Buscar -->
      <button type="submit">Buscar</button>
    </form>

    <!-- Listagem de Resultados Paginada -->
    <div v-if="results.length > 0">
      <ul>
        <li v-for="(item, index) in paginatedResults" :key="index">
          {{ item.name }} - {{ item.substance }} - {{ item.cnpj }} -
          {{ item.laboratory }}
        </li>
      </ul>

      <!-- Paginação -->
      <div>
        <button @click="previousPage" :disabled="currentPage === 1">
          Anterior
        </button>
        <span>Página {{ currentPage }} de {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage === totalPages">
          Próxima
        </button>
      </div>
    </div>

    <!-- Mensagem de sem resultados -->
    <p v-else>Nenhum resultado encontrado</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchTerm: "",
      selectedFilter: "all", // Valor inicial do select
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

      // Parâmetros de busca e filtros
      const params = {
        search: this.searchTerm,
      };

      // Adiciona o filtro selecionado aos parâmetros
      if (this.selectedFilter !== "all") {
        params.filter = this.selectedFilter; // Envia o tipo de filtro
      }

      try {
        const response = await fetch(
          `${apiUrl}?${new URLSearchParams(params)}`
        );
        const data = await response.json();
        this.results = data;
        this.currentPage = 1; // Resetar para a primeira página após a busca
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
/* Adicione estilos se necessário */
</style>
