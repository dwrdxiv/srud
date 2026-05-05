<script setup>
import { ref, onMounted, computed } from 'vue';
import { collection, getDocs, updateDoc, increment, doc } from 'firebase/firestore';
import { db } from '../firebase';

const carrito = ref([]);
const mostrarCarrito = ref(false);
const zapatoSeleccionado = ref(null);
const tallaElegida = ref('');

const busqueda = ref('');

const zapatos = ref([]);
const cargando = ref(true);

const generarRecibo = () => {
  const productosHTML = carrito.value.map(item => `
    <tr>
      <td>${item.nombre} (Size: ${item.talla})</td>
      <td>$${item.precio}</td>
    </tr>
  `).join('');
  const total = carrito.value.reduce((sum, item) => sum + item.precio, 0);
  const ventanaPuntual = window.open('', '_blank');
  ventanaPuntual.document.write(`
    <html>
      <head>
        <title>Buy Invoice - SHRUD</title>
        <style>
          body { font-family: sans-serif; padding: 40px; text-align: center; }
          table { width: 100%; border-collapse: collapse; margin-top: 20px; }
          th, td { border: 1px solid #ddd; padding: 12px; text-align: left; }
          .total { font-size: 1.5rem; font-weight: bold; margin-top: 20px; }
          .footer { margin-top: 50px; font-size: 0.8rem; color: #777; }
        </style>
      </head>
      <body>
        <h1>SHRUD</h1>
        <p>Payment Receipt</p>
        <hr>
        <table>
          <thead><tr><th>Product</th><th>Price</th></tr></thead>
          <tbody>${productosHTML}</tbody>
        </table>
        <div class="total">Total: $${total}</div>
        <p class="footer">Thank you for your purchase. Date: ${new Date().toLocaleDateString()}</p>
      </body>
    </html>
  `);

  ventanaPuntual.document.close();
  ventanaPuntual.print(); // Lanza el diálogo de impresión
};

const prepararCarrito = (zapato) => {
  zapatoSeleccionado.value = zapato;
  tallaElegida.value = '';
  mostrarCarrito.value = true;
};

const anadiralCarrito = () => {
  if (!tallaElegida.value) {
    alert('Please, select a size before adding to cart.');
    return;
  }

  const stockDisponible = zapatoSeleccionado.value.tallas[tallaElegida.value];
  if (stockDisponible <= 0) {
    alert("We are sorry, size out of stock.")
    mostrarCarrito.value = true;
    return;
  } 

  carrito.value.push({
    id: zapatoSeleccionado.value.id,
    nombre: zapatoSeleccionado.value.nombre,
    marca: zapatoSeleccionado.value.marca,
    precio: zapatoSeleccionado.value.precio_venta,
    talla: tallaElegida.value,
    imagen_url: zapatoSeleccionado.value.imagen_url
  });

  mostrarCarrito.value = false;
  alert('Added '+ zapatoSeleccionado.value.nombre + ' size ' + tallaElegida.value + ' to your cart.');
};

const vaciarCarrito = () => {
  if (confirm('¿Are you sure you want to empty the cart?')) {
    carrito.value = [];
  }
};
const procesarCompra = async () => {
  try {
      for (const item of carrito.value) {
      const docRef = doc(db, "Zapatos", item.id);
      await updateDoc(docRef, {
        [`tallas.${item.talla}`]: increment(-1)
      });
  }
  alert('Purchase successful! Generating your receipt...');
  generarRecibo();
  obtenerZapatos();
  carrito.value = [];
  } catch (error) {
    console.error("Error processing purchase:", error);
    alert('There was an error processing the purchase. Please try again.'); 
  }
  
};

const obtenerZapatos = async () => {
  try {
    const querySnapshot = await getDocs(collection(db, "Zapatos"));
    zapatos.value = querySnapshot.docs.map(doc => ({
      id: doc.id,
      ...doc.data()
    }));
  } catch (error) {
    console.error("Error loading products:", error);
  } finally {
    cargando.value = false;
  }
};

const zapatosFiltrados = computed(() => {
  const termino = busqueda.value.toLowerCase().trim();
  if (!termino) return zapatos.value;

  return zapatos.value.filter(zapato => {
    return zapato.nombre.toLowerCase().includes(termino) || 
           zapato.marca.toLowerCase().includes(termino);
  })
});

onMounted(obtenerZapatos);
</script>

<template>
  <main >
    <section >
      <div class="catalogoTitle">
          <h2  style="font-size: 2rem; margin-bottom: 0px;">Available Products</h2>
      </div>
      <div class="searchDiv">
          <input v-model="busqueda" type="text" placeholder="Looking for your favorite?" class="search-input" />
      </div>
      
      <div v-if="cargando">
        <div ></div>
      </div>
      <div v-else class="catalogoDiv">
        <div v-for="zapato in zapatosFiltrados" :key="zapato.id" class="productoDiv">
          <div >
            <img 
              :src="zapato.imagen_url || 'https://images.unsplash.com/photo-1542291026-7eec264c27ff?q=80&w=600'"/>
          </div>

          <div class="productInfoDiv" >
            <div >
              <h3 >{{ zapato.nombre }}</h3>
              <p >{{ zapato.marca }}</p>
            </div>
            <span >${{ zapato.precio_venta }}</span>
            
            
            <button @click="prepararCarrito(zapato)">Add to Cart</button>
          </div>
        </div>
      </div>

      <div v-if="!cargando && zapatos.length === 0" >
        <p >No available shoes in inventory.</p>
        <RouterLink to="/admin">Go to Admin panel to add products</RouterLink>
      </div>
    </section>
  </main>


  <div v-if="mostrarCarrito" class="modal-overlay">
    <div class="modal-content">
      <h2>{{ zapatoSeleccionado.nombre }}</h2>
      <p>Select your size:</p>
      
      <div class="tallas-grid">
        <button class="tallas"
          v-for="(stock, talla) in zapatoSeleccionado.tallas" 
          :key="talla"
          @click="tallaElegida = talla"
          :class="{ 'talla-activa': tallaElegida === talla }"
          :disabled="stock <= 0"
        >
          {{ talla }} ({{ stock }} available)
        </button>
      </div>

      <div class="modal-acciones">
        <button @click="mostrarCarrito = false" class="btn-cancelar">Cancel</button>
        <button @click="anadiralCarrito" class="btn-confirmar">Confirm</button>
      </div>
    </div>
  </div>

  <div v-if="carrito.length > 0" class="resumen-carrito">
    <h3>Your Cart ({{ carrito.length }})</h3>
      <ul>
        <li v-for="(item, index) in carrito" :key="index">
          {{ item.nombre }} - Size: {{ item.talla }} - ${{ item.precio }}
        </li>
      </ul>
      <div class="carrito-acciones">
        <button @click="vaciarCarrito" class="btn-vaciar">Empty Cart</button>
        <button @click="procesarCompra" class="btn-pagar">Checkout</button>
      </div>
    
  </div>  
</template>

<style scoped>
.catalogoTitle {
  text-align: center;
  margin-bottom: 20px;
  font-family: Arial, Helvetica, sans-serif;
}

.catalogoDiv {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 55px;
  width: 85%;
  margin: 0 auto 3rem auto;
  border-radius: 20px;
  padding: 40px;
  background-image: linear-gradient(180deg, #ffffff 0%, #dbdfe7 50%);
}
.searchDiv {
  text-align: center;
  margin-bottom: 30px;
}
.search-input {
  width: 300px;
  padding: 10px 15px;
  border-radius: 20px;
  border: 2px solid #ccc;
  font-size: 1rem;
  transition: all 0.2s ease-in-out;
}
.productoDiv {
  background: rgb(255, 255, 255);
  border-radius: 10px;
  width: 300px;
  height: 450px;
  margin: 20px auto;
  overflow: hidden;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.274);
  display: flex;
  min-width: 0;
  flex-direction: column;
}
.productoDiv img {
  width: 100%;
  height: 300px;
  object-fit: cover;
}
.productInfoDiv {
  padding: 10px;
  display: flex;
  flex-direction:column;
  justify-content: space-between;
}
.productInfoDiv h3 {
  font-family: Arial, Helvetica, sans-serif;
  margin: 0;
  font-size: 1.2rem;
}
.productInfoDiv p {
  font-family: Arial, Helvetica, sans-serif;
  margin: 0 0;
  color: #555;
}
.productInfoDiv span {
  font-family: Arial, Helvetica, sans-serif;
  text-align: right;
  color: #000000;
  font-size: 1.5rem;
  margin-top: 10px;
}
.productInfoDiv button {
  margin-top: 15px;
  background-color: #42a846;
  color: white;
  font-size: 1rem;
  font-weight: bold;
  border-radius: 7px;
  border: none;
  height: 30px;
  padding: 5px 10px;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.productInfoDiv button:hover {
  background-color: #0ab833;
  transform: scale(1.03);
}
.productInfoDiv button:active{
  transform: scale(1);
}

.modal-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0,0,0,0.7);
  backdrop-filter: blur(3px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: rgba(255, 255, 255, 0.774);
  font-family: Arial, Helvetica, sans-serif;
  padding: 30px;
  border-radius: 15px;
  max-width: 400px;
  width: 90%;
  text-align: center;
}

.tallas-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
  margin: 20px 0;
}
.tallas {
  background: #f0f0f0;
  border: 3px solid #f0f0f0;
  padding: 10px;
  border-radius: 5px;
  cursor: pointer;
  transition: border 0.2s ease-in-out;
}
.tallas:hover {
  border: #000 solid 3px;
}
.talla-activa {
  background: transparent;
  background-color: #000 !important;
  color: white;
}

.modal-acciones {
  display: flex;
  justify-content: space-around;
  margin-top: 20px;
}
.btn-cancelar {
  background: #ffffff;
  font-weight: bold;
  color: rgb(182, 0, 0);
  border: 2px solid rgb(182, 0, 0);
  padding: 10px 20px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.btn-cancelar:hover {
  background-color: rgb(182, 0, 0);
  color: white;
}
.btn-confirmar {
  background: #28a745;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.btn-confirmar:hover {
  background-color: #0ab833;
  transform: scale(1.05);
}
.btn-confirmar:active{
  transform: scale(1);
}

.resumen-carrito {
  position: fixed;
  bottom: 20px;
  right: 20px;
  font-family: Arial, Helvetica, sans-serif;
  background-image: linear-gradient(180deg, #a5b0ce00 0%, #ffffff 50%);
  color: rgb(0, 0, 0);
  backdrop-filter: blur(3px);
  padding: 20px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.329);
  border-radius: 10px;
}
.resumen-carrito h3 {
  margin: 0 0 10px 0;
}
.resumen-carrito ul {
  list-style: none;
  padding: 0;
  margin: 0 0 15px 0;
}
.carrito-acciones {
  display: flex;
  justify-content: space-between;
}
.resumen-carrito .btn-vaciar {
  background-color: #dc3545;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.resumen-carrito .btn-vaciar:hover {
  background-color: #c82333;
  transform: scale(1.05);
}
.resumen-carrito .btn-pagar {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
  transition: all 0.1s ease-in-out;
}
.resumen-carrito .btn-pagar:hover {
  background-color: #218838;
  transform: scale(1.05);
}


</style>