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
        />
        <select
          v-model="selectedFilter"
          class="form-select"
          aria-label="Selecione um filtro"
        >
          <option value="substance">Substância</option>
          <option value="cnpj">CNPJ</option>
          <option value="laboratory">Laboratório</option>
        </select>
        <button type="submit" class="btn btn-dark">Buscar</button>
        <button type="button" class="btn btn-secondary" @click="clearFilter">
          Limpar Filtro
        </button>
      </div>
    </form>

    <div v-if="results.length > 0" class="mt-4">
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
            <td>{{ item.substance }}</td>
            <td>{{ item.cnpj }}</td>
            <td>{{ item.laboratory }}</td>
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
      selectedFilter: "substance", // Filtro padrão
      results: [],
      currentPage: 1,
      perPage: 25,
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
      const apiUrl = "http://localhost:3000/medicines";
      const params = {
        filter: this.selectedFilter,
      };

      if (this.searchTerm) {
        params[this.selectedFilter] = this.searchTerm; // Filtrando pela chave correta
      }

      console.log("Parâmetros enviados:", params); // Log para depuração

      try {
        const response = await fetch(
          `${apiUrl}?${new URLSearchParams(params)}`
        );

        if (!response.ok) {
          throw new Error("Erro na resposta da API"); // Tratamento de erro mais claro
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
      this.selectedFilter = "substance"; // Reseta o filtro para 'substance'
      this.fetchAllData(); // Busca todos os dados novamente
    },
    async fetchAllData() {
      const apiUrl = "http://localhost:3000/medicines";

      try {
        const response = await fetch(apiUrl);

        if (!response.ok) {
          throw new Error("Erro na resposta da API");
        }

        const data = await response.json();
        this.results = data; // Define todos os dados
        this.currentPage = 1; // Resetar a página para 1
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
