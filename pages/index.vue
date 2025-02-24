
<template>
  <div class="app-container">
    <div class="nav-tabs">
      <button :class="{ active: activeTab === 'converter' }" @click="activeTab = 'converter'">
        💰 Converter
      </button>
      <button :class="{ active: activeTab === 'currencies' }" @click="activeTab = 'currencies'">
        📄 Currencies
      </button>
    </div>

    <div v-if="activeTab === 'converter'">
      <div class="currency-card">
        <h1 class="title">💰 Currency Converter</h1>

        <div class="currency-list">
          <div v-for="(value, code) in currencies" :key="code" class="currency-item">
            <span class="currency-label">{{ code }}</span>
            <input 
              type="number" 
              v-model.number="currencies[code]" 
              @input="onCurrencyInput(code)"
              class="currency-input"
            />
          </div>
        </div>

        <div class="button-container">
          <button @click="showAddCurrency = !showAddCurrency" class="add-currency-btn">
            ➕ Add Currency
          </button>
        </div>

        <div v-if="showAddCurrency" class="dropdown-container">
          <select v-model="selectedCurrency" class="dropdown">
            <option disabled value="">Select Currency</option>
            <option v-for="option in availableCurrencies" :key="option" :value="option">
              {{ option }}
            </option>
          </select>
          <button @click="addCurrency" class="confirm-btn">✅ Add</button>
        </div>
      </div>
    </div>

    <div v-if="activeTab === 'currencies'">
      <List/>
    </div>
  </div>
</template>

<script setup>

const activeTab = ref('converter');

const currencies = useState('currencies', () => ({
  USD: 1,
  EUR: 0,
}));

const allCurrencies = ['USD', 'EUR', 'RUB', 'BYN'];
const selectedCurrency = ref('');
const showAddCurrency = ref(false);

const availableCurrencies = computed(() =>
  allCurrencies.filter((code) => !(code in currencies.value))
);


const onCurrencyInput = async (baseCurrency) => {
  const amount = currencies.value[baseCurrency];
  if (!amount || amount <= 0) return;

  try {
    const response = await $fetch('http://localhost:8000/api/convert/', {
      method: 'POST',
      body: {
        base_currency: baseCurrency,
        amount: amount,
        target_currencies: Object.keys(currencies.value).filter(
          (c) => c !== baseCurrency
        ),
      },
    });

    if (response) {
      const updatedValues = { ...currencies.value };
      Object.keys(response.converted_values).forEach((code) => {
        updatedValues[code] = response.converted_values[code];
      });
      currencies.value = updatedValues;
    } else {
      console.error('Invalid response:', response);
    }
  } catch (error) {
    console.error('Error converting currency:', error);
  }
};

const addCurrency = () => {
  if (selectedCurrency.value) {
    currencies.value[selectedCurrency.value] = 0;
    selectedCurrency.value = '';
    showAddCurrency.value = false;
  }
};
</script>

<style scoped>

.app-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  background: linear-gradient(to right, #1e293b, #0f172a);
  z-index: 1;
  padding-top: 60px;
}

.nav-tabs {
  display: flex;
  gap: 15px;
  margin-bottom: 10px;
}

.nav-tabs button {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  padding: 12px 20px;
  font-size: 1rem;
  font-weight: bold;
  color: white;
  border-radius: 8px;
  cursor: pointer;
  transition: 0.3s;
}

.nav-tabs button.active {
  background: rgba(255, 255, 255, 0.5);
}

.currency-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  padding: 2rem;
  border-radius: 15px;
  width: 90%;
  max-width: 400px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
}

.title {
  font-size: 1.8rem;
  font-weight: bold;
  text-align: center;
  color: #facc15;
  margin-bottom: 1.5rem;
}


.currency-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.currency-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.15);
  padding: 0.8rem;
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.currency-label {
  font-weight: bold;
  font-size: 1.1rem;
  color: #facc15;
}

.currency-input {
  width: 100px;
  padding: 0.5rem;
  font-size: 1rem;
  border: none;
  outline: none;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.2);
  color: white;
  text-align: right;
}

.button-container {
  margin-top: 1.5rem;
  text-align: center;
}

.add-currency-btn {
  width: 100%;
  background: linear-gradient(45deg, #22c55e, #16a34a);
  color: white;
  font-size: 1.1rem;
  font-weight: bold;
  padding: 0.8rem;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  transition: 0.3s ease;
}

.add-currency-btn:hover {
  background: linear-gradient(45deg, #16a34a, #15803d);
}

.dropdown-container {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  position: relative;
  z-index: 999;
}

.dropdown {
  padding: 0.7rem;
  border-radius: 10px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.9);
  color: black;
  border: 1px solid rgba(0, 0, 0, 0.2);
  width: 100%;
}

.confirm-btn {
  width: 100%;
  background: linear-gradient(45deg, #6366f1, #4338ca);
  color: white;
  font-size: 1.1rem;
  font-weight: bold;
  padding: 0.8rem;
  border-radius: 10px;
  border: none;
}
</style>
