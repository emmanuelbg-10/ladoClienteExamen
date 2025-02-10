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
    <button @click="eliminarPais">Eliminar</button>
  </div>
    <h4 v-else>No hay datos que mostrar</h4>
</template>

<script setup>
import { ref, watch, computed } from 'vue';

const props = defineProps({
  codigo: String
});

const emit = defineEmits(['paisEliminado']);

const countryData = ref(null);


const pibPerCapita = computed(() => {
  if (countryData.value && countryData.value.population) {
    return (countryData.value.pib / countryData.value.population).toFixed(2);
  }
  return 0;
});

const eliminarPais = async () => {
  if (props.codigo) {
    try {
      const response = await fetch(`http://localhost:3000/api/country/${props.codigo}`, {
        method: 'DELETE'
      });
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      emit('paisEliminado', props.codigo);
      countryData.value = '';
    } catch (err) {
      console.log('Error deleting country data:', err);
    }
  }
};

watch(() => props.codigo, async (newCodigo) => {
  if (newCodigo) {
    try {
      const response = await fetch(`http://localhost:3000/api/country/${newCodigo}`);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      countryData.value = await response.json();
    } catch (err) {
      console.log('Error fetching country data:', err);
      countryData.value = null;
    }
  }
});
</script>

<style scoped>

</style>