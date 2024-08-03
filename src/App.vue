<template>
  <div>
    <input v-model="filterQuery" placeholder="Suche..." />
    <table class="table">
      <thead>
        <tr>
          <th @click="sortBy('country')">Land</th>
          <th @click="sortBy('company')">Unternehmen</th>
          <th @click="sortBy('emissions')">Emissionen (in Tonnen CO2)</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="record in filteredAndSortedData" :key="record.id">
          <td>{{ record.country }}</td>
          <td>{{ record.company }}</td>
          <td>{{ record.emissions }}</td>
        </tr>
        <tr>
          <td colspan="2"><strong>Gesamtemissionen</strong></td>
          <td><strong>{{ totalEmissions }}</strong></td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  data() {
    return {
      records: [],
      filterQuery: '',
      sortKey: '',
      sortOrders: {
        country: 1,
        company: 1,
        emissions: 1
      }
    };
  },
  computed: {
    filteredAndSortedData() {
      let filteredData = this.records.filter(record =>
        record.country.toLowerCase().includes(this.filterQuery.toLowerCase()) ||
        record.company.toLowerCase().includes(this.filterQuery.toLowerCase())
      );
      return filteredData.sort((a, b) => {
        const modifier = this.sortOrders[this.sortKey] || 1;
        if (a[this.sortKey] < b[this.sortKey]) return -1 * modifier;
        if (a[this.sortKey] > b[this.sortKey]) return 1 * modifier;
        return 0;
      });
    },
    totalEmissions() {
      return this.filteredAndSortedData.reduce((sum, record) => sum + record.emissions, 0);
    }
  },
  methods: {
    sortBy(key) {
      this.sortKey = key;
      this.sortOrders[key] = this.sortOrders[key] * -1;
    }
  },
  methods: {
    sanitizeInput() {
      this.sanitizedFilter = this.escapeHTML(this.filter);
    },
    escapeHTML(str) {
      return str.replace(/[&<>"'`=\/]/g, function(s) {
        return {
          '&': '&amp;',
          '<': '&lt;',
          '>': '&gt;',
          '"': '&quot;',
          "'": '&#39;',
          '/': '&#x2F;',
          '`': '&#x60;',
          '=': '&#x3D;'
        }[s];
      });
    }
  },
  mounted() {
    fetch('co2-data.json')
      .then(response => response.json())
      .then(data => {
        this.records = data;
      })
      .catch(error => console.error('Error loading data:', error));
  }
};
</script>
