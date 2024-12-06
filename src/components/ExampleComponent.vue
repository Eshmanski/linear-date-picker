<template>
  <div>
    <LinearDatePicker :value="value" />
  </div>
</template>

<script setup lang="ts">
import LinearDatePicker from './LinearDatePicker.vue';

import { ref } from 'vue';


const value = ref([new Date().getTime(), new Date().getTime()]);
</script>
