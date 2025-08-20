<script>
import Header from './components/Header.vue'

export default {
  name: 'Parties',
  components: { Header },
  data() {
    return {
      parties: [],
      currentPage: 1,
      itemsPerPage: 10,
      partySearch: '',
      nonPartySearch: ''
    }
  },
  computed: {
    filteredPartyCountries() {
      if (!this.partySearch) return this.paginatedPartyCountries;
      const search = this.partySearch.toLowerCase();
      return this.paginatedPartyCountries.filter(country => 
        country.name.en.toLowerCase().includes(search)
      );
    },
    filteredNonPartyCountries() {
      if (!this.nonPartySearch) return this.nonPartyCountries;
      const search = this.nonPartySearch.toLowerCase();
      return this.nonPartyCountries.filter(country => 
        country.name.en.toLowerCase().includes(search)
      );
    },
    partyCountries() {
      const today = new Date();
      return this.parties.filter(country => {
        const partyDate = country.treaties?.XXVII8?.party;
        return partyDate && new Date(partyDate) <= today;
      });
    },
    nonPartyCountries() {
      const today = new Date();
      return this.parties.filter(country => {
        const partyDate = country.treaties?.XXVII8?.party;
        return !partyDate || new Date(partyDate) > today;
      });
    },
    paginatedPartyCountries() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.partyCountries.slice(start, end);
    },
    totalPages() {
      return Math.ceil(this.partyCountries.length / this.itemsPerPage);
    },
    lastAccession() {
      const partiesWithDates = this.partyCountries.filter(country => country.treaties?.XXVII8?.party);
      if (partiesWithDates.length === 0) return 'N/A';
      const lastAccessionDate = partiesWithDates.reduce((latest, country) => {
        const date = new Date(country.treaties.XXVII8.party);
        return date > latest ? date : latest;
      }, new Date(0));
      return lastAccessionDate.toLocaleDateString();
    }
  },
  methods: {
    async getParties() {
      try {
        const response = await fetch('https://api.cbd.int/api/v2013/countries');
        const data = await response.json();
        return data;
      } catch (error) {
        console.error('Error fetching data:', error);
        return [];
      }
    },
    formatDate(dateString) {
      if (!dateString) return 'N/A';
      return new Date(dateString).toLocaleDateString();
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    }
  },
  async created() {
    this.parties = await this.getParties();
  }
}
</script>

<template>
  <div>
    <Header />
    <div class="container mt-4">
      <div class="row mb-4">
        <div class="col-md-6 d-flex align-items-stretch">
          <div class="card shadow-sm border-0 w-100">
            <div class="card-body d-flex align-items-center">
              <div class="me-3">
                <i class="bi bi-people-fill text-primary" style="font-size: 2rem;"></i>
              </div>
              <div>
                <h5 class="card-title mb-1">Parties Overview</h5>
                <p class="card-text mb-0"><strong>Number of Parties:</strong> {{ partyCountries.length }}</p>
                <p class="card-text mb-0"><strong>Last Accession:</strong> {{ lastAccession }}</p>
              </div>
            </div>
          </div>
        </div>
        <div class="col-md-6 d-flex align-items-stretch">
          <div class="card shadow-sm border-0 w-100">
            <div class="card-body d-flex align-items-center">
              <div class="me-3">
                <i class="bi bi-exclamation-triangle-fill text-warning" style="font-size: 2rem;"></i>
              </div>
              <div>
                <h5 class="card-title mb-1">Non-Parties Overview</h5>
                <p class="card-text mb-0"><strong>Number of Non-Parties:</strong> {{ nonPartyCountries.length }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-6">
          <div class="card mb-4">
            <div class="card-header bg-success text-white d-flex justify-content-between align-items-center">
              <h2 class="h4 mb-0">Party Countries</h2>
              <input type="text" v-model="partySearch" class="form-control form-control-sm w-auto" placeholder="Search countries...">
            </div>
            <div class="table-responsive">
              <table class="table table-hover mb-0">
                <thead class="table-light">
                  <tr><th>Country Name</th><th>Date</th></tr>
                </thead>
                <tbody>
                  <tr v-for="country in filteredPartyCountries" :key="country._id">
                    <td>{{ country.name.en }}</td>
                    <td>{{ formatDate(country.treaties?.XXVII8?.party) }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div class="pagination-controls d-flex justify-content-between align-items-center">
              <button class="btn btn-sm btn-outline-primary" @click="prevPage" :disabled="currentPage === 1">
                <i class="bi bi-chevron-left me-1"></i>Previous
              </button>
              <span class="text-muted">Page {{ currentPage }} of {{ totalPages }}</span>
              <button class="btn btn-sm btn-outline-primary" @click="nextPage" :disabled="currentPage === totalPages">
                Next<i class="bi bi-chevron-right ms-1"></i>
              </button>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <div class="card">
            <div class="card-header bg-warning text-dark d-flex justify-content-between align-items-center">
              <h2 class="h4 mb-0">Non-Party Countries</h2>
              <input type="text" v-model="nonPartySearch" class="form-control form-control-sm w-auto" placeholder="Search countries...">
            </div>
            <div class="table-responsive">
              <table class="table table-hover mb-0">
                <thead class="table-light">
                  <tr><th>Country Name</th><th>Date</th></tr>
                </thead>
                <tbody>
                  <tr v-for="country in filteredNonPartyCountries" :key="country._id">
                    <td>{{ country.name.en }}</td>
                    <td>{{ formatDate(country.treaties?.XXVII8?.party) }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.card {
  border: none;
  box-shadow: 0 10px 20px rgba(0,0,0,0.05);
  border-radius: 12px;
  transition: all 0.3s ease;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 12px 24px rgba(0,0,0,0.1);
}

.card-header {
  border-bottom: none;
  border-radius: 12px 12px 0 0 !important;
  padding: 1rem 1.25rem;
}

.table {
  margin-bottom: 0;
}

.table th {
  font-weight: 600;
  color: #4a5568;
  border-bottom-width: 2px;
}

.table td {
  padding: 1rem 1.25rem;
  vertical-align: middle;
}

.form-control {
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.2);
  background: rgba(255,255,255,0.1);
  color: inherit;
}

.form-control::placeholder {
  color: rgba(255,255,255,0.7);
}

.form-control:focus {
  box-shadow: none;
  border-color: rgba(255,255,255,0.5);
  background: rgba(255,255,255,0.2);
}

.btn {
  border-radius: 8px;
  padding: 0.5rem 1rem;
  font-weight: 500;
  transition: all 0.2s ease;
}

.btn:hover {
  transform: translateY(-1px);
}

.bg-success {
  background: linear-gradient(135deg, #0ea5e9 0%, #0284c7 100%) !important;
}

.bg-warning {
  background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%) !important;
}

.table-hover tbody tr:hover {
  background-color: rgba(0,0,0,0.02);
}

.container {
  padding: 2rem 1rem;
}

.card-body {
  padding: 1.5rem;
}

.table-responsive {
  border-radius: 0 0 12px 12px;
  overflow: hidden;
}

.pagination-controls {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  padding: 1rem;
  border-radius: 0 0 12px 12px;
}

.bi {
  opacity: 0.9;
}
</style>
