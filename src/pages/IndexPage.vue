<template>
  <q-page class="q-pa-md column items-center justify-center">
    <div class="column items-center">
      <h1 class="text-h4 text-weight-bold text-primary q-mb-lg">To Do List</h1>
    </div>
    <div class="row items-center justify-center q-gutter-md" style="max-width: 400px; width: 100%;">
      <q-input
        v-model="inputValue"
        outlined
        placeholder="Digite algo..."
        class="col"
      />
      <q-btn
        label="Enviar"
        unelevated
        @click="handleButtonClick"
        color="deep-purple-9"
        text-color="white"
        rounded
      />
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const inputValue = ref<string>('');

const handleButtonClick = () => {
  if (inputValue.value.trim() !== '') {
    console.log('Valor digitado:', inputValue.value);
    inputValue.value = '';
  } else {
    console.log('O campo está vazio!');
  }
};
</script>
