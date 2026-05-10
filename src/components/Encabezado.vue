<script setup>
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();

const pageData = computed(() => {
  switch (route.path) {
    case '/inventario':
      return {
        titulo: 'Inventory',
        gif: '/bannerinventario.png',
        logotitulo: '/logoinventory.svg'
      };
    case '/':
    default:
      return {
        titulo: 'Home - Shoe Catalog',
        gif: '/bannerhome.png',
        logotitulo: '/logocatalog.svg'
      };

  }
});
</script>

<template>
    <header class="main-header">
    <nav class="top-nav">
      <div class="brand">SHRUD</div>
      <div class="botonesUser">
        <button class="loginButton">Login</button>
        <button class="registerButton">Register</button>
      </div>
    </nav>

    <div class="banner" :style="{ backgroundImage: `url(${pageData.gif})` }">
      <div class="overlay">
        <img :src="pageData.logotitulo"  :alt="pageData.titulo" class="header-svg-titulo" style="color: aliceblue; height: 100px; margin-left: 85%; transition: all 0.2s ease-in-out;">
      </div>
    </div>
  </header>
</template>

<style scoped>
.top-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 5%;
  background: white;
  border-bottom: 1px solid #eee;
}

.brand { font-weight: bold; font-family: Arial, Helvetica, sans-serif; font-size: 1.5rem; }

.auth-buttons { display: flex; gap: 10px; }
.loginButton {
  background-color: #42a846;
  color: white;
  font-weight: bold;
  border: none;
  height: 30px;
  padding: 5px 10px;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.loginButton:hover {
  background-color: #0ab833;
  transform: scale(1.05);
  }
.loginButton:active{
  transform: scale(1);
}

.registerButton {
  background-color: #3b7ee2;
  color: white;
  font-weight: bold;
  border: none;
  height: 30px;
  padding: 5px 10px;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.registerButton:hover {
  background-color: #48aaeb;
  transform: scale(1.05);
  }
.registerButton:active{
  transform: scale(1);
}


.banner {
  height: 350px;
  background-size: cover;
  background-position: center;
  position: relative;
  transition: all 0.5s ease;
}

.overlay {
  background: rgba(0, 0, 0, 0); 
  color: white;
  font-family: Arial, Helvetica, sans-serif;
  font-size: 1.2rem;
  width: 100%;
  height: 100%;
  display: flex;
  align-items:end;
  justify-content:last baseline;
}
.header-svg-titulo:hover{
  transform: scale(1.05);
  fill: red;
}


</style>