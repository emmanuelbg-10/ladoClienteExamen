<template>
  <div v-if="countryData">
    <table>
      <tr>
        <th>Nombre:</th>
        <td>{{ countryData.name }}</td>
        <th>Población:</th>
        <td>{{ countryData.population }} millones</td>
      </tr>
      <tr>
        <th>PIB:</th>
        <td>{{ countryData.pib }} millones</td>
        <th>Área:</th>
        <td>{{ countryData.area }}km<sup>2</sup></td>
      </tr>
      <tr>
        <th>Renta:</th>
        <td>{{ countryData.income }}</td>
        <th>PIB/habitante:</th>
        <td :style="{ color: pibPerCapita > countryData.income ? 'red' : 'black' }">
          {{ pibPerCapita }}
        </td>
      </tr>
    </table>
    <button>Eliminar</button>
  </div>
  <div v-else>
    <p>Selecciona un país para ver los datos.</p>
  </div>
  <div v-if="error">
    <p style="color: red;">No se pueden mostrar los datos.</p>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue';

const props = defineProps({
  codigo: String
});

const countryData = ref(null);
const error = ref(false);

const pibPerCapita = computed(() => {
  if (countryData.value && countryData.value.population) {
    return (countryData.value.pib / countryData.value.population).toFixed(2);
  }
  return 0;
});

watch(() => props.codigo, async (newCodigo) => {
  if (newCodigo) {
    try {
      const response = await fetch(`http://localhost:3000/api/country/${newCodigo}`);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      countryData.value = await response.json();
      error.value = false;
    } catch (err) {
      console.log('Error fetching country data:', err);
      error.value = true;
      countryData.value = null;
    }
  }
});
</script>

<style scoped>
h2 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
}
p {
  margin: 0.5rem 0;
}
</style>