<script setup>
import { ref, watch, onMounted, computed } from 'vue';
import Componente from './components/Componente.vue';
import codes from './components/codes.json';
import GoogleChart from './components/GoogleChart.vue';

const name = ref('');
const codigo = ref('');
const selectedCountries = ref([]);
const countryNames = ref({});

const fetchCountryNames = async () => {
  try {
    const response = await fetch("http://localhost:3000/api/names");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    countryNames.value = data;
  } catch (error) {
    console.log('Error fetching country names:', error);
  }
};

onMounted(() => {
  fetchCountryNames();
});

const selectCode = (code) => {
  if (code === 'XXX') {
    console.log('pais no encontrado');
    return;
  }
  if (!selectedCountries.value.includes(code)) {
    selectedCountries.value.push(code);
    codigo.value = code;
  }
};

const deselectCode = (code) => {
  selectedCountries.value = selectedCountries.value.filter(country => country !== code);
};

const sortedSelectedCountries = computed(() => {
  return selectedCountries.value.slice().sort((a, b) => {
    const nameA = countryNames.value[a] || a;
    const nameB = countryNames.value[b] || b;
    return nameA.localeCompare(nameB);
  });
});

watch(codigo, (newCodigo) => {
  if (newCodigo === 'XXX') {
    console.log('pais no mostrado');
    return;
  }

  if (newCodigo) {
    var requestOptions = {
      method: 'GET',
      redirect: 'follow'
    };

    fetch("http://localhost:3000/api/country/" + newCodigo, requestOptions)
      .then(response => {
        if (!response.ok) {
          console.log('Pais no encontrado');
        }
        return response.json();
      })
      .then(data => console.log(data))
      .catch(error => console.log('error', error));
  }
});
</script>

<template>
  <div class="parent">
    <div class="header">
      <h1>Datos mundiales</h1>
    </div>
    <div class="name">
      <p>Ingresa tu nombre:</p>
      <input type="text" v-model="name">
      <p v-if="name.trim() !== ''">Tu nombre es <strong>{{ name }}</strong> tiene {{ name.length }} letras</p>
    </div>
    <div class="div-codes">
      <h2>Codigos</h2>
      <ul>
        <li v-for="code in codes" :key="code" @click="selectCode(code)">
          {{ code !== 'XXX' ? (countryNames[code] || code) : '' }}
        </li>
      </ul>
    </div>
    <div class="selected">
      <h2 v-if="selectedCountries.length === 0">Debes seleccionar algún país en el panel de códigos</h2>
      <ul v-else>
        <h2>Seleccionados ({{ selectedCountries.length }}):</h2>
        <li v-for="country in sortedSelectedCountries" :key="country">
          {{ countryNames[country] || country }} <button @click="deselectCode(country)">Desmarcar</button>
        </li>
      </ul>
    </div>
    <div class="country-data"></div>
    <div class="options"></div>
    <div class="chart"></div>
  </div>
</template>

<style scoped>
.parent {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  grid-template-rows: auto repeat(3, 1fr);
  grid-column-gap: 0px;
  grid-row-gap: 0px;
  height: 100vh;
  margin: -2rem;
}

.header {
  grid-area: 1 / 1 / 2 / 6;
  background-color: aquamarine;
}

.header h1 {
  margin: 2px;
}

.div-codes {
  grid-area: 2 / 1 / 6 / 2;
  background-color: azure;
  overflow-y: auto;
}

.name {
  grid-area: 2 / 2 / 3 / 3;
  background-color: antiquewhite;
}

.selected {
  grid-area: 2 / 5 / 4 / 6;
  background-color: lightyellow;
  overflow-y: auto;
}

.country-data {
  grid-area: 2 / 3 / 3 / 5;
  background-color: lightseagreen;
}

.options {
  grid-area: 3 / 2 / 4 / 5;
  background-color: bisque;
}

.chart {
  grid-area: 4 / 2 / 5 / 6;
  background-color: aqua;
}

ul li {
  text-align: left;
  font-size: 0.9rem;
}

ul li:hover {
  font-weight: bold;
  cursor: pointer;
}

ul li button {
  font-size: 0.6rem;
  margin: 2px;
}

h2 {
  font-size: 1.2rem;
}
</style>