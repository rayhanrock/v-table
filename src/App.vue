<template>
  <div style="width: 80%; margin: 10px auto">
    <div class="ui icon input">
      <input
        @input="search"
        v-model="searchText"
        type="text"
        placeholder="Search..." />
      <i class="search icon"></i>
    </div>

    <table class="ui celled table">
      <thead>
        <tr>
          <th>রশিদ নং</th>
          <th>কাষ্টমার</th>
          <th>পন্যসমুহ</th>
          <th @click="shortByCost">হিসাব</th>
          <th @click="shortBydate">সময়</th>
          <th id="actionHeader">ক্রিয়াকলাপ</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in filteredRows">
          <td>{{ row.id }}</td>
          <td>{{ row.customer }}</td>
          <td>
            <div class="ui list">
              <div
                v-for="product in row.products"
                class="item">
                <div class="content">
                  <div class="header">{{ product.name }}</div>
                  {{ product.price }}
                </div>
              </div>
            </div>
          </td>
          <td>{{ calculateBill(row.products) }}</td>
          <td>{{ row.date.toLocaleDateString() }}</td>

          <td>
            <button @click="deleteRow(row.id)">delete</button
            ><button @click="printRow(row)">print</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'App',
  data() {
    return {
      rows: [],
      filteredRows: [],
      searchText: '',
      isAscendingDate: true,
      isAscendingCost: true,
    };
  },
  computed: {
    calculateBill() {
      return (products) => {
        return products.reduce((acc, cur) => acc + cur.price, 0);
      };
    },
  },
  async created() {
    try {
      const res = await axios.get(`http://localhost:3000/rows`);
      console.log(res);

      this.rows = res.data.map((row) => {
        return {
          ...row,
          date: new Date(row.date),
        };
      });

      this.filteredRows = this.rows;
    } catch (error) {
      console.log(error);
    }
  },

  methods: {
    shortBydate() {
      if (this.isAccendingDate) {
        this.rows.sort((a, b) => b.date - a.date);
        this.isAccendingDate = false;
      } else {
        this.rows.sort((a, b) => a.date - b.date);
        this.isAccendingDate = true;
      }
    },
    shortByCost() {
      if (this.isAscendingCost) {
        this.rows.sort(
          (a, b) =>
            this.calculateBill(b.products) - this.calculateBill(a.products)
        );
        this.isAscendingCost = false;
      } else {
        this.rows.sort(
          (a, b) =>
            this.calculateBill(a.products) - this.calculateBill(b.products)
        );
        this.isAscendingCost = true;
      }
    },

    search() {
      console.log(this.searchText);
      var searhText = this.searchText.toLowerCase();
      this.filteredRows = this.rows.filter((row) => {
        return (
          row.customer.toLowerCase().includes(searhText) ||
          row.id.toString().toLowerCase().includes(searhText)
        );
      });
    },

    deleteRow(id) {
      this.rows = this.rows.filter((row) => row.id !== id);
      this.filteredRows = this.rows;
    },

    printRow(row) {
      const printWindow = window.open('', '_blank');

      const rowContent = this.getRowContent(row);

      // Customize the content and styling for printing
      const printHtml = `
        <!DOCTYPE html>
        <html>
        <head>
          <style>
            /* Customize your print styles here */
            body {
              font-family: Arial, sans-serif;
            }
            /* ... other styles ... */
          </style>
        </head>
        <body>
          ${rowContent}
        </body>
        </html>
      `;

      // Write the content to the print window
      printWindow.document.open();
      printWindow.document.write(printHtml);
      printWindow.document.close();

      // Trigger the print dialog
      printWindow.print();
    },

    getRowContent(row) {
      // Create a string containing product names and prices
      const productDetails = row.products
        .map((product) => `${product.name} - ${product.price}`)
        .join('<br>');

      // Customize the content to match your row structure
      const rowHtml = `
        <h2>Invoice Details</h2>
        <p>ID: ${row.id}</p>
        <p>Customer: ${row.customer}</p>
        <p>Product Details: <p style="margin-left : 20px">${productDetails}</p></p>
        <p>Bill: ${this.calculateBill(row.products)}</p>
        <p>Date: ${row.date.toLocaleDateString()}</p>
      `;

      return rowHtml;
    },
  },
};
</script>
