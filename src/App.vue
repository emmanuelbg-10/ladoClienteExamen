<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import CountryData from './components/CountryData.vue';
import GoogleChart from './components/GoogleChart.vue';
import codes from './components/codes.json';

const name = ref('');
const codigo = ref('');
const selectedCountries = ref([]);
const countryNames = ref({});
const dataTypes = ['population', 'pib', 'area', 'income']; // Array estático con las opciones
const selectedDataType = ref(dataTypes[0]); // Valor seleccionado por defecto

const fetchCountryNames = async () => {
  try {
    const response = await fetch("http://localhost:3000/api/names");
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    countryNames.value = data;
  } catch (error) {
    console.log('Pais no encontrado');
  }
};

onMounted(() => {
  fetchCountryNames();
});

const selectCode = (code) => {
  if (!countryNames.value[code]) {
    console.log('pais no encontrado');
    return;
  }
  if (!selectedCountries.value.includes(code)) {
    selectedCountries.value.push(code);
    codigo.value = code;
  } else {
    console.log('pais no encontrado');
  }
};

const deselectCode = (code) => {
  selectedCountries.value = selectedCountries.value.filter(country => country !== code);
};

const handlePaisEliminado = (codigoEliminado) => {
  selectedCountries.value = selectedCountries.value.filter(country => country !== codigoEliminado);
  codigo.value = '';
};

const sortedSelectedCountries = computed(() => {
  return selectedCountries.value.slice().sort((a, b) => {
    const nameA = countryNames.value[a] || a;
    const nameB = countryNames.value[b] || b;
    return nameA.localeCompare(nameB);
  });
});

const generateChartData = computed(() => {
  const data = [[ 'país', selectedDataType.value ]];
  selectedCountries.value.forEach(countryCode => {
    const countryName = countryNames.value[countryCode];
    const countryData = countryDataMap[countryCode];
    if (countryName && countryData) {
      data.push([countryName, countryData[selectedDataType.value]]);
    }
  });
  return data;
});

watch(codigo, (newCodigo) => {
  if (!countryNames.value[newCodigo]) {
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
      .then(data => {
        countryDataMap[newCodigo] = data;
        console.log(data);
      })
      .catch(error => console.log('error', error));
  }
});

const countryDataMap = ref({});
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
          {{ countryNames[code] || '' }}
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
    <div class="country-data">
      <CountryData :codigo="codigo" @paisEliminado="handlePaisEliminado" />
    </div>
    <div class="options">
      <ul>
        <li v-for="option in dataTypes" :key="option">
          <input type="radio" :id="option" :value="option" v-model="selectedDataType">
          <label :for="option">{{ option }}</label>
        </li>
      </ul>
    </div>
    <div class="chart">
      <GoogleChart :data="generateChartData" />
    </div>
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