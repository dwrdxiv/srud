<script setup>
import { RouterView } from 'vue-router';
import NavBar from './components/NavBar.vue';
import Encabezado from './components/Encabezado.vue';
import Footer from './components/Footer.vue';
</script>

<template>
  <div>
    <Encabezado />
    <NavBar />
  </div>
  <RouterView/>
  <Footer />
</template>

<style scoped>


</style>
