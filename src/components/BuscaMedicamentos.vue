<template>
  <div class="container mt-4 shadow p-4 bg-light rounded">
    <h1 class="text-center mb-4 text-dark fw-bold">Busca de Medicamentos</h1>

    <form @submit.prevent="fetchData" class="d-flex align-items-center mb-4">
      <div class="input-group flex-grow-1">
        <input
          type="text"
          v-model="searchTerm"
          placeholder="Buscar..."
          class="form-control"
          aria-label="Buscar"
          @input="fetchData"
        />
        <select
          v-model="selectedFilter"
          class="form-select"
          aria-label="Selecione um filtro"
        >
          <option value="SUBSTANCIA">Substância</option>
          <option value="CNPJ">CNPJ</option>
          <option value="LABORATORIO">Laboratório</option>
        </select>
        <button type="button" class="btn btn-secondary" @click="clearFilter">
          Limpar Filtro
        </button>
      </div>
    </form>

    <div v-if="filteredResults.length > 0" class="mt-4">
      <table class="table table-striped table-bordered">
        <thead class="table-dark">
          <tr>
            <th>Substância</th>
            <th>CNPJ</th>
            <th>Laboratório</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in paginatedResults" :key="index">
            <td>{{ item.SUBSTANCIA }}</td>
            <td>{{ item.CNPJ }}</td>
            <td>{{ item.LABORATORIO }}</td>
          </tr>
        </tbody>
      </table>

      <div class="d-flex justify-content-between mt-3 mb-4">
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

    <p v-else class="mt-3 text-center text-danger">
      Nenhum resultado encontrado
    </p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchTerm: "",
      selectedFilter: "SUBSTANCIA", // Filtro padrão em letras maiúsculas
      results: [],
      currentPage: 1,
      perPage: 25,
    };
  },
  computed: {
    totalPages() {
      return Math.ceil(this.filteredResults.length / this.perPage);
    },
    paginatedResults() {
      const start = (this.currentPage - 1) * this.perPage;
      const end = start + this.perPage;
      return this.filteredResults.slice(start, end);
    },
    filteredResults() {
      if (!this.searchTerm) {
        return this.results; // Se não há termo de busca, retorna todos os resultados
      }

      return this.results.filter((item) => {
        return item[this.selectedFilter]
          .toLowerCase()
          .includes(this.searchTerm.toLowerCase());
      });
    },
  },
  methods: {
    async fetchData() {
      const apiUrl = "http://localhost:8000/api/data-excel/list/"; // Atualize a URL aqui
      const params = {
        filter: this.selectedFilter,
        [this.selectedFilter]: this.searchTerm,
      };

      console.log("Parâmetros enviados:", params); // Log para depuração

      try {
        const response = await fetch(
          `${apiUrl}?${new URLSearchParams(params)}`
        );

        if (!response.ok) {
          throw new Error("Erro na resposta da API");
        }

        const data = await response.json();
        this.results = data;
        this.currentPage = 1; // Resetar a página para 1 após a busca
      } catch (error) {
        console.error("Erro ao buscar dados:", error);
        this.results = []; // Limpar resultados em caso de erro
      }
    },
    clearFilter() {
      this.searchTerm = ""; // Limpa o input de pesquisa
      this.selectedFilter = "SUBSTANCIA"; // Reseta o filtro para 'SUBSTANCIA'
      this.fetchAllData(); // Busca todos os dados novamente
    },
    async fetchAllData() {
      const apiUrl = "http://localhost:8000/api/data-excel/list/"; // Atualize a URL aqui

      console.log("Iniciando busca de todos os dados..."); // Log para depuração

      try {
        const response = await fetch(apiUrl);

        if (!response.ok) {
          throw new Error("Erro na resposta da API");
        }

        const data = await response.json();
        this.results = data; // Define todos os dados
        this.currentPage = 1; // Resetar a página para 1
        console.log("Dados recebidos:", data); // Log dos dados recebidos
      } catch (error) {
        console.error("Erro ao buscar dados:", error);
        this.results = []; // Limpar resultados em caso de erro
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
  mounted() {
    console.log("Componente montado. Carregando dados..."); // Log para depuração
    this.fetchAllData(); // Carregar todos os dados ao iniciar o componente
  },
};
</script>

<style>
.container {
  max-width: 800px;
}

.table {
  margin-top: 20px;
}

.table-striped tbody tr:nth-of-type(odd) {
  background-color: #f9f9f9; /* Cor de fundo para linhas ímpares */
}

.table-striped tbody tr:hover {
  background-color: #e9ecef; /* Cor de fundo ao passar o mouse */
}

.text-primary {
  color: #007bff; /* Azul padrão do Bootstrap */
}

.text-danger {
  color: #dc3545; /* Vermelho padrão do Bootstrap */
}

/* Transição suave nos botões */
.btn {
  transition: background-color 0.3s, transform 0.2s;
}

.btn:hover {
  transform: scale(1.05); /* Efeito de zoom ao passar o mouse */
}
</style>
