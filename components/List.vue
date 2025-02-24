<template>
  <div class="currency-list-container">
    <h1 class="title">📊 Currency List</h1>

    <table class="currency-table">
      <thead>
        <tr>
          <th>Currency</th>
          <th>Value (relative to USD)</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(value, code) in exchangeRates" :key="code">
          <td>{{ code }}</td>
          <td>{{ value }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
const exchangeRates = ref({});
const allCurrencies = ['USD', 'EUR', 'RUB', 'BYN'];

onMounted(async () => {
  try {
    const response = await $fetch('http://localhost:8000/api/convert/', {
      method: 'POST',
      body: {
        base_currency: 'USD',
        amount: 1,
        target_currencies: allCurrencies,
      },
    });
    exchangeRates.value=response.converted_values

  console.log('api response:', response);
} catch (error) {
  console.error('Error fetching exchange rates:', error);
}
});
</script>


  <style scoped>
  .currency-list-container {
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(10px);
    padding: 2rem;
    border-radius: 15px;
    width: 90%;
    max-width: 400px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
    text-align: center;
  }
  
  .title {
    font-size: 1.8rem;
    font-weight: bold;
    text-align: center;
    color: #facc15;
    margin-bottom: 1.5rem;
  }
  
  .currency-table {
    width: 100%;
    border-collapse: collapse;
  }
  
  .currency-table th,
  .currency-table td {
    padding: 12px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    text-align: center;
    font-size: 1rem;
  }
  
  .currency-table th {
    background: rgba(255, 255, 255, 0.15);
    font-weight: bold;
    color: white;
  }
  
  .currency-table td {
    background: rgba(255, 255, 255, 0.1);
    color: white;
  }
  </style>
  