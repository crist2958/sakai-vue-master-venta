<template>
  <div class="flex items-start justify-start w-full">
    <div class="bg-white p-6 rounded-lg shadow-md w-full">
      <h2 class="text-2xl font-semibold mb-6">GENERAR EL RFC DE UNA PERSONA</h2>
      <form @submit.prevent="generarRFC" class="space-y-4">
        <div class="flex flex-col">
          <label for="nombre" class="text-gray-700 font-medium">Nombre(s):</label>
          <input type="text" id="nombre" v-model="nombre" class="border p-2 rounded" required />
        </div>
        <div class="flex flex-col">
          <label for="apellidoPaterno" class="text-gray-700 font-medium">Apellido Paterno:</label>
          <input type="text" id="apellidoPaterno" v-model="apellidoPaterno" class="border p-2 rounded" required />
        </div>
        <div class="flex flex-col">
          <label for="apellidoMaterno" class="text-gray-700 font-medium">Apellido Materno:</label>
          <input type="text" id="apellidoMaterno" v-model="apellidoMaterno" class="border p-2 rounded" required />
        </div>
        <div class="flex flex-col">
          <label for="fechaNacimiento" class="text-gray-700 font-medium">Fecha de Nacimiento:</label>
          <input type="date" id="fechaNacimiento" v-model="fechaNacimiento" class="border p-2 rounded" required />
        </div>
        <div class="flex flex-col">
          <label for="rfc" class="text-gray-700 font-medium">RFC</label>
          <input type="text" id="rfc" v-model="rfc" class="border p-2 rounded bg-gray-100" readonly />
        </div>
        <button 
          type="submit"
          class="w-full bg-green-500 text-white py-2 rounded hover:bg-green-600 transition">
          GENERAR
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const nombre = ref('');
const apellidoPaterno = ref('');
const apellidoMaterno = ref('');
const fechaNacimiento = ref('');
const rfc = ref('');

const generarRFC = () => {
  if (!nombre.value || !apellidoPaterno.value || !apellidoMaterno.value || !fechaNacimiento.value) {
    alert('Por favor, llena todos los campos antes de generar el RFC.');
    return;
  }

  // Extrae los caracteres
  const dia = fechaNacimiento.value.slice(8, 10); 
  const mes = fechaNacimiento.value.slice(5, 7); 
  const anio = fechaNacimiento.value.slice(2, 4); 

  // Esto genera el rfc pro
  rfc.value = `${apellidoPaterno.value.slice(0, 2)}${apellidoMaterno.value.charAt(0)}${nombre.value.charAt(0)}${anio}${mes}${dia}`.toUpperCase();
};
</script>
