<template>
    <div>
      <div class="actions">
        <!-- Refresh Button -->
        <button @click="refreshTable" class="button-refresh">
          <i class="fa fa-refresh"></i> Refresh
        </button>

        <!-- Export Button -->
        <button class="button-download" @click="exportToExcel">Export</button>

        <!-- Advance Search Button -->
        <button @click="toggleSearch" class="button-search">
          <i :class="{'fa-chevron-up': showSearch, 'fa-chevron-down': !showSearch}"></i> Advance Search
        </button>
      </div>

      <!-- Advance Search (Collapsible) -->
      <div v-if="showSearch" class="advance-search">
        <div>
          <label class="advance-label">Price Range:</label>
          <input class="advance-input" type="number" v-model="searchFilters.priceFrom" placeholder="From" />
          <input class="advance-input" type="number" v-model="searchFilters.priceTo" placeholder="To" />
        </div>
        <div>
          <label class="advance-label">Date Range:</label>
          <input class="advance-input" type="date" v-model="searchFilters.dateFrom" placeholder="From" />
          <input class="advance-input" type="date" v-model="searchFilters.dateTo" placeholder="To" />
        </div>
        <div>
          <label>Status:</label>
          <div>
            <label class="advance-label"><input type="checkbox" v-model="searchFilters.status" value="Konfirmasi Pembayaran" /> Konfirmasi Pembayaran</label>
            <label class="advance-label"><input type="checkbox" v-model="searchFilters.status" value="Lunas" /> Lunas</label>
            <label class="advance-label"><input type="checkbox" v-model="searchFilters.status" value="Proses Pembayaran" /> Proses Pembayaran</label>
          </div>
        </div>
        <button class="button-filter" @click="applyFilters">Apply</button>
      </div>

      <!-- Table -->
      <table id="data-table">
        <thead>
          <tr>
            <th><input type="checkbox" v-model="selectAll" @change="checkAll" /></th>
            <th @click="sort('noKewajiban')">Nomor Kewajiban</th>
            <th @click="sort('noPolisi')">Nomor Polisi</th>
            <th @click="sort('pemilik')">Pemilik</th>
            <th @click="sort('peserta')">Peserta</th>
            <th @click="sort('nomorVA')">Nomor VA</th>
            <th @click="sort('hargaTerbentuk')">Harga Terbentuk</th>
            <th @click="sort('biayaAdmin')">Biaya Admin ex PPN</th>
            <th @click="sort('ppn')">PPN</th>
            <th @click="sort('total')">Total</th>
            <th @click="sort('tanggalLelang')">Tanggal Lelang</th>
            <th @click="sort('tanggalJatuhTempo')">Tanggal Jatuh Tempo</th>
            <th @click="sort('tanggalLunas')">Tanggal Lunas</th>
            <th @click="sort('status')">Status</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in paginatedData" :key="index">
            <td><input type="checkbox" v-model="item.checked" /></td>
            <td>{{ item.noKewajiban }}</td>
            <td>{{ item.noPolisi }}</td>
            <td>{{ item.pemilik }}</td>
            <td>{{ item.peserta }}</td>
            <td>{{ item.nomorVA }}</td>
            <td>{{ item.hargaTerbentuk }}</td>
            <td>{{ item.biayaAdmin }}</td>
            <td>{{ item.ppn }}</td>
            <td>{{ item.total }}</td>
            <td>{{ item.tanggalLelang }}</td>
            <td>{{ item.tanggalJatuhTempo }}</td>
            <td>{{ item.tanggalLunas }}</td>
            <td>{{ item.status }}</td>
          </tr>
        </tbody>
      </table>
    
      <!-- Pagination -->
      <div style="text-align: center;">
        <button class="button-page" @click="prevPage" :disabled="currentPage === 1">Prev</button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button class="button-page" @click="nextPage" :disabled="currentPage === totalPages">Next</button>
      </div>
    </div>
</template>

<script>
import * as XLSX from 'xlsx';

export default {
  data() {
    return {
      items: [
        {
          noKewajiban: 'B20116005069',
          noPolisi: 'KT 3089 LS',
          pemilik: 'PT OLX',
          peserta: 'Suyono',
          nomorVA: '5621000012456',
          hargaTerbentuk: 93850000,
          biayaAdmin: 0,
          ppn: 0,
          total: 93850000,
          tanggalLelang: '11/11/2020',
          tanggalJatuhTempo: '17/11/2020',
          tanggalLunas: '17/11/2020',
          status: 'Konfirmasi Pembayaran',
          checked: false
        },
        {
          noKewajiban: 'B20316005067',
          noPolisi: 'KT 3090 JSX',
          pemilik: 'PT OLX',
          peserta: 'Budiman',
          nomorVA: '95621000012456',
          hargaTerbentuk: 13950000,
          biayaAdmin: 0,
          ppn: 0,
          total: 13950000,
          tanggalLelang: '10/11/2020',
          tanggalJatuhTempo: '16/11/2020',
          tanggalLunas: '14/11/2020',
          status: 'Lunas',
          checked: false
        },
        {
          noKewajiban: 'B20116005065',
          noPolisi: 'B 1234 JEP',
          pemilik: 'PT OLX',
          peserta: 'Suherman',
          nomorVA: '95621000012459',
          hargaTerbentuk: 190000000,
          biayaAdmin: 0,
          ppn: 0,
          total: 190000000,
          tanggalLelang: '09/11/2020',
          tanggalJatuhTempo: '15/11/2020',
          tanggalLunas: '',
          status: 'Proses Pembayaran',
          checked: false
        },
        {
          noKewajiban: 'B33434343434',
          noPolisi: 'B 2243 LJD',
          pemilik: 'PT Tokopedia',
          peserta: 'Sunanta',
          nomorVA: '3003383992283000',
          hargaTerbentuk: 147340000,
          biayaAdmin: 0,
          ppn: 0,
          total: 147340000,
          tanggalLelang: '10/08/2020',
          tanggalJatuhTempo: '30/08/2020',
          tanggalLunas: '25/08/2020',
          status: 'Lunas',
          checked: false
        },
        {
          noKewajiban: 'B20116005069',
          noPolisi: 'KT 3089 LS',
          pemilik: 'PT OLX',
          peserta: 'Suyono',
          nomorVA: '5621000012456',
          hargaTerbentuk: 93850000,
          biayaAdmin: 0,
          ppn: 0,
          total: 93850000,
          tanggalLelang: '11/11/2020',
          tanggalJatuhTempo: '17/11/2020',
          tanggalLunas: '17/11/2020',
          status: 'Konfirmasi Pembayaran',
          checked: false
        },
        {
          noKewajiban: 'B20316005067',
          noPolisi: 'KT 3090 JSX',
          pemilik: 'PT OLX',
          peserta: 'Budiman',
          nomorVA: '95621000012456',
          hargaTerbentuk: 13950000,
          biayaAdmin: 0,
          ppn: 0,
          total: 13950000,
          tanggalLelang: '10/11/2020',
          tanggalJatuhTempo: '16/11/2020',
          tanggalLunas: '14/11/2020',
          status: 'Lunas',
          checked: false
        },
        {
          noKewajiban: 'B20116005065',
          noPolisi: 'B 1234 JEP',
          pemilik: 'PT OLX',
          peserta: 'Suherman',
          nomorVA: '95621000012459',
          hargaTerbentuk: 190000000,
          biayaAdmin: 0,
          ppn: 0,
          total: 190000000,
          tanggalLelang: '09/11/2020',
          tanggalJatuhTempo: '15/11/2020',
          tanggalLunas: '',
          status: 'Proses Pembayaran',
          checked: false
        },
        {
          noKewajiban: 'B33434343434',
          noPolisi: 'B 2243 LJD',
          pemilik: 'PT Tokopedia',
          peserta: 'Sunanta',
          nomorVA: '3003383992283000',
          hargaTerbentuk: 147340000,
          biayaAdmin: 0,
          ppn: 0,
          total: 147340000,
          tanggalLelang: '10/08/2020',
          tanggalJatuhTempo: '30/08/2020',
          tanggalLunas: '25/08/2020',
          status: 'Lunas',
          checked: false
        },
        {
          noKewajiban: 'B20116005069',
          noPolisi: 'KT 3089 LS',
          pemilik: 'PT OLX',
          peserta: 'Suyono',
          nomorVA: '5621000012456',
          hargaTerbentuk: 93850000,
          biayaAdmin: 0,
          ppn: 0,
          total: 93850000,
          tanggalLelang: '11/11/2020',
          tanggalJatuhTempo: '17/11/2020',
          tanggalLunas: '17/11/2020',
          status: 'Konfirmasi Pembayaran',
          checked: false
        },
        {
          noKewajiban: 'B20316005067',
          noPolisi: 'KT 3090 JSX',
          pemilik: 'PT OLX',
          peserta: 'Budiman',
          nomorVA: '95621000012456',
          hargaTerbentuk: 13950000,
          biayaAdmin: 0,
          ppn: 0,
          total: 13950000,
          tanggalLelang: '10/11/2020',
          tanggalJatuhTempo: '16/11/2020',
          tanggalLunas: '14/11/2020',
          status: 'Lunas',
          checked: false
        },
        {
          noKewajiban: 'B20116005065',
          noPolisi: 'B 1234 JEP',
          pemilik: 'PT OLX',
          peserta: 'Suherman',
          nomorVA: '95621000012459',
          hargaTerbentuk: 190000000,
          biayaAdmin: 0,
          ppn: 0,
          total: 190000000,
          tanggalLelang: '09/11/2020',
          tanggalJatuhTempo: '15/11/2020',
          tanggalLunas: '',
          status: 'Proses Pembayaran',
          checked: false
        },
        {
          noKewajiban: 'B33434343434',
          noPolisi: 'B 2243 LJD',
          pemilik: 'PT Tokopedia',
          peserta: 'Sunanta',
          nomorVA: '3003383992283000',
          hargaTerbentuk: 147340000,
          biayaAdmin: 0,
          ppn: 0,
          total: 147340000,
          tanggalLelang: '10/08/2020',
          tanggalJatuhTempo: '30/08/2020',
          tanggalLunas: '25/08/2020',
          status: 'Lunas',
          checked: false
        },
      ],
      selectAll: false,
      currentPage: 1,
      itemsPerPage: 10,
      sortKey: '',
      sortOrder: 1,
      showSearch: false,
      searchFilters: {
        priceFrom: null,
        priceTo: null,
        dateFrom: null,
        dateTo: null,
        status: [],
      },
    };
  },
  computed: {
    // Total Pages
    totalPages() {
      return Math.ceil(this.filteredData.length / this.itemsPerPage);
    },
    // Pagination
    paginatedData() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.sortedData.slice(start, end);
    },
    sortedData() {
      return this.filteredData.sort((a, b) => {
        if (this.sortKey) {
          let valueA = a[this.sortKey];
          let valueB = b[this.sortKey];

          if (typeof valueA === 'string') {
            valueA = valueA.toLowerCase();
            valueB = valueB.toLowerCase();
          }

          if (this.sortKey.includes('tanggal')) {
            valueA = new Date(valueA);
            valueB = new Date(valueB);
          }

          return (valueA > valueB ? 1 : valueA < valueB ? -1 : 0) * this.sortOrder;
        }
        return 0;
      });
    },
    // Filter Data
    filteredData() {
      return this.items.filter(item => {
        const withinPriceRange =
          (!this.searchFilters.priceFrom || item.hargaTerbentuk >= this.searchFilters.priceFrom) &&
          (!this.searchFilters.priceTo || item.hargaTerbentuk <= this.searchFilters.priceTo);
        
        const withinDateRange =
          (!this.searchFilters.dateFrom || new Date(item.tanggalLelang) >= new Date(this.searchFilters.dateFrom)) &&
          (!this.searchFilters.dateTo || new Date(item.tanggalLelang) <= new Date(this.searchFilters.dateTo));
        
        const matchesStatus =
          this.searchFilters.status.length === 0 || this.searchFilters.status.includes(item.status);
        
        return withinPriceRange && withinDateRange && matchesStatus;
      });
    },
  },
  methods: {
    // Checkbox All
    checkAll() {
      this.items.forEach(item => {
        item.checked = this.selectAll;
      });
    },
    sort(key) {
      if (this.sortKey === key) {
        this.sortOrder *= -1;
      } else {
        this.sortOrder = 1;
      }
      this.sortKey = key;
    },
    // Next Page
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    // Prev Page
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    // Clear filters
    refreshTable() {
      this.searchFilters = {
        priceFrom: null,
        priceTo: null,
        dateFrom: null,
        dateTo: null,
        status: [],
      };
      this.currentPage = 1;
    },
    // Toggle Search
    toggleSearch() {
      this.showSearch = !this.showSearch;
    },
    applyFilters() {
      this.currentPage = 1;
    },
    // Download Excel
    exportToExcel() {
      const wb = XLSX.utils.book_new();
      const ws = XLSX.utils.table_to_sheet(document.getElementById('data-table'));

      XLSX.utils.book_append_sheet(wb, ws, 'Daftar Piutang Unit');

      XLSX.writeFile(wb, 'daftar_piutang_unit.xlsx');
    }
  },
};
</script>


<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 8px;
  text-align: left;
  border-top: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
}

th {
  cursor: pointer;
  font-weight: bold;
  width: fit-content;
}

button {
  margin: 8px 12px;
  border: none;
  color: white;
  padding: 8px 16px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  border-radius: 10px;
  cursor: pointer;
}

.button-search{
  background-color: #04AA6D;
}

.button-refresh{
  background-color: #02a2fe;
}

.button-filter{
  background-color: #808080;
}

.button-page{
  background-color: #808080;
}

.button-download{
  background-color: #f6a83c;
}

.actions {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
}

.advance-search {
  display: block;
  flex-wrap: wrap;
  margin-bottom: 15px;
}

.advance-label {
  display: block;
  flex-wrap: wrap;
  margin-bottom: 6px;
}

.advance-input {
  display: inline-block;
  margin: 0px 12px 0px 0px;
  padding: 8px;
  border-radius: 8px;
  border: 1px solid;
}

.advance-search div {
  margin-right: 15px;
  margin-bottom: 10px;
}

.advance-search label {
  margin-right: 5px;
}
</style>