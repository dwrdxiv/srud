<script setup>
import { ref, onMounted } from 'vue';
import { collection, addDoc, getDocs, deleteDoc, doc, updateDoc } from 'firebase/firestore';
import { db } from '../firebase';

const zapatos = ref([]);
const nuevoZapato = ref({
  nombre: '',
  marca: '',
  costo: null,
  precio_venta: null,
  tallas: {},
  imagen_url: ''
});
const tallaInput = ref('');
const cantidadInput = ref(null);
const editandoID = ref(null);
const editando = ref(false);

const prepararEdicion = (zapato) => {
  editandoID.value = zapato.id;
  editando.value = true;
  
  nuevoZapato.value = { 
    nombre: zapato.nombre,
    marca: zapato.marca,
    precio_venta: zapato.precio_venta,
    costo: zapato.costo,
    tallas: { ...zapato.tallas },
    imagen_url: zapato.imagen_url || ''
  };
  
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

const cancelarEdicion = () => {
  editandoID.value = null;
  editando.value = false;
  nuevoZapato.value = { nombre: '', marca: '', precio_venta: null, costo: null, tallas: {}, imagen_url: '' };
};

const agregarTallaAlMapa = () => {
  if(tallaInput.value && cantidadInput.value > 0) {
    nuevoZapato.value.tallas[tallaInput.value] = cantidadInput.value;
    tallaInput.value = '';
    cantidadInput.value = null;
  }
};

const quitarTallaDelMapa = (talla) => {
  delete nuevoZapato.value.tallas[talla];
};

const agregarZapato = async () => {
  if (!nuevoZapato.value.nombre || !nuevoZapato.value.marca) {
    alert("Please complete the name and brand fields.");
    return;
  }

  if (nuevoZapato.value.precio_venta <= 0 || nuevoZapato.value.costo < 0) {
    alert("Please enter valid prices. (e.g., 1.50$)");
    return;
  }

  if (Object.keys(nuevoZapato.value.tallas).length === 0) {
    alert("Please add at least one size with stock.");
    return;
  }

  try {
    if (editandoID.value) {
      const docRef = doc(db, "Zapatos", editandoID.value);
      await updateDoc(docRef, nuevoZapato.value);
      editando.value = false;
      alert("Shoe updated successfully.");
    } else {
      const docRef = await addDoc(collection(db, "Zapatos"), nuevoZapato.value);
      alert("Shoe added successfully. ID: " + docRef.id);
    }

    cancelarEdicion();
    obtenerZapatos(); 
    
    nuevoZapato.value = { nombre: '', marca: '', precio_venta: null, costo: null, tallas: {}, imagen_url: '' };
  } catch (e) {
    console.error("Error adding shoe: ", e);
    alert("Error adding shoe.");
  }
};

const obtenerZapatos = async () => {
  const querySnapshot = await getDocs(collection(db, "Zapatos"));
  zapatos.value = querySnapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
};
const borrarZapato = async (id) => {
  if(confirm("Are you sure you want to delete it?")) {
    await deleteDoc(doc(db, "Zapatos", id));
    obtenerZapatos();
  }
};

onMounted(obtenerZapatos);
</script>

<template>
  <div class="container" style="font-family: Arial, Helvetica, sans-serif;">
    <h2 class="admin-titulo" style="text-align: center;">Inventory Management</h2>

    <form @submit.prevent="agregarZapato" class="register-form">
      <label for="nombreInput">Name</label>
      <input v-model="nuevoZapato.nombre" placeholder="Jordan Retro 4" class="nombreInput" />
      <label for="marcaInput">Brand</label>
      <input v-model="nuevoZapato.marca" placeholder="Nike" class="marcaInput" />
      <label for="pcostoInput">Cost</label>
      <input v-model.number="nuevoZapato.costo" type="number" placeholder="$4.50" class="pcostoInput"  />
      <label for="pventaInput">Price</label>
      <input v-model.number="nuevoZapato.precio_venta" type="number" placeholder="$10.00" class="pventaInput" />

        <p class="pTallas" style="margin: auto;">Sizes</p>

        <div class="tallasDiv">
          <input v-model="tallaInput" type="number" placeholder="Size (e.g. 7)" class="tallaInput" />
          <input v-model.number="cantidadInput" type="number" placeholder="Quantity (e.g. 16)" class="ctallaInput" />

        </div>

        <div class="listaTallas" style="grid-column: span 2;">
          <div v-for="(stock, talla) in nuevoZapato.tallas" :key="talla">
            <span><strong>Size: {{ talla }}</strong> ({{ stock }})</span>
            <button @click="quitarTallaDelMapa(talla)" class="deleteTallaButton">x</button>
          </div>
        </div>
        
        <button @click.prevent="agregarTallaAlMapa" class="addTallaButton" style="grid-column: span 4;"> + Add Size</button>

        <label style="grid-column: span 2;">Product Image (URL):</label>
        <input v-model="nuevoZapato.imagen_url" type="text" placeholder="https://ejemplo.com/zapato.jpg" class="input-estandar" style="grid-column: span 2;" />
      <button type="submit" :class="{ 'addZapatoButton': !editando, 'updateButton': editando}" style="grid-column: span 4; font-size: 1rem;">{{ editando ? 'Update Product' : 'Add Inventory' }}</button>
      <button v-if="editandoID" type="button" @click="cancelarEdicion" class="btn-cancel"> Cancel</button>  
    </form>
    

    <table class="inventarioTable">
      <thead>
        <tr>
          <th style="width: 120px; min-width: 100px;">ID</th>
          <th> </th>
          <th>Product</th>
          <th>Brand</th>
          <th style="max-width: 1rem;">Cost</th>
          <th style="max-width: 1rem;">Price</th>
          <th style="max-width: 1rem;">Stock</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="z in zapatos" :key="z.id">
          
          <td style=" word-break: break-all; overflow-wrap: break-word; overflow: hidden; font-size: 0.8rem; text-align: center;">{{ z.id }}</td>
          <td style="max-width: 40px;">
            <div class="contenedor-foto">
              <img :src="z.imagen_url || 'data:image/png;base64,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'"  
              :alt="z.nombre" style="max-width: 50px; max-height: 50px;">
            </div>
          </td>
          
          <td>{{ z.nombre }}</td>
          <td>{{ z.marca }}</td>
          <td>${{ z.costo }}</td>
          <td>${{ z.precio_venta }}</td>
          <td>
            <div class="scroll-tallas" style="max-height: 2.5rem; overflow-y: auto; max">
              <div v-for="(stock, talla) in z.tallas" :key="talla" class="fila-talla" >
                Size {{ talla }} <strong>({{ stock }})</strong>
              </div>
            </div>
          </td>
          <td>
            <div class="actionButtonDivs">
              <button @click="prepararEdicion(z)" class="editButton">✎</button>
              <button @click="borrarZapato(z.id)" class="deleteButton">✖</button>
              
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>

</template>

<style scoped>
  .register-form {
    max-width: 35rem;
    margin: 20px auto;
    padding: 20px;
    background-image: linear-gradient(180deg, #f8f6f6 0%, #c6cedf 100%);
    border-radius: 8px;
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 15px;
  }
  .register-form input {
    padding: 8px;
    border: 1px solid #bebebe;
    border-radius:  8px;
    width: auto;
  }
  .register-form label {
    margin:auto;
  }
  .register-form .addZapatoButton {
    padding: 10px 15px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s ease-in-out, transform 0.1s ease-in-out;
  }
  .register-form .addZapatoButton:hover {
    background-color: #0040ff;
    transform: scale(1.03);
  }
  .register-form .addZapatoButton:active {
    transform: scale(1);
  }
  .register-form .updateButton {
    padding: 10px 15px;
    background-color: #ec981a;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.3s ease-in-out, transform 0.1s ease-in-out;
  }
  .register-form .updateButton:hover {
    background-color: #df6c00;
    transform: scale(1.03);
  }
  .register-form .updateButton:active {
    transform: scale(1);
  }
  
  .addTallaButton {
    width: 100%;
    padding: 8px 12px;
    background-color: #42a846;
    font-size: 1rem;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-top: 12px;
    transition: background-color 0.2s ease-in-out, transform 0.1s ease-in-out;
  }
  .addTallaButton:hover {
    background-color: #20b657;
    transform: scale(1.05);
  }
  .addTallaButton:active {
    transform: scale(1);
  }

  .listaTallas {
    display: grid;
    overflow-y: auto;
    max-height: 60px;
    grid-template-columns: repeat(auto-fit, minmax(120px, 100px));
    gap: 10px;
    align-items: center;
    padding: 8px;
  }
  .listaTallas .deleteTallaButton {
    background-color: red;
    border: none;
    border-radius: 50%;
    color: #ffffff;
    font-size: 16px;
    cursor: pointer;
    transition: transform 0.1s ease-in-out;
  }
  .listaTallas .deleteTallaButton:hover {
    transform: scale(1.1);
  }

  .inventarioTable {
    width: 70%;
    margin: 20px auto;
    margin-top: 4rem;
    border-collapse: collapse;
    box-shadow: 0 0px 20px rgba(59, 4, 4, 0.212);
    background-color: rgb(255, 255, 255);
    border-radius: 8px;
    overflow: hidden;
  }
  .inventarioTable th {
    background-color: #797979;
    color: white;
  }
  .inventarioTable td {
     overflow-y: auto;
     padding: 5px;
  }
  .inventarioTable tbody tr:hover {
    background-color: #ebebeb;
  }
  .actionButtonDivs {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
  }
  .actionButtonDivs .editButton {
    padding: 5px 10px;
    height: 2.5rem;
    width: 2.5rem;
    background-color: #ff9036;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out, transform 0.1s ease-in-out;
  }
  .actionButtonDivs .editButton:hover {
    background-color: #c75a1e;
    transform: scale(1.1);
  }
  .actionButtonDivs .editButton:active {
    transform: scale(0.95);
  }
  .actionButtonDivs .deleteButton {
    padding: 5px 10px;
    height: 2.5rem;
    width: 2.5rem;
    background-color: #dc3545;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out, transform 0.1s ease-in-out;
  }
  .actionButtonDivs .deleteButton:hover {
    background-color: #6e140d;
    transform: scale(1.1);
  }
  .actionButtonDivs .deleteButton:active {
    transform: scale(0.95);
  }
  .register-form .btn-cancel {
    padding: 5px 10px;
    height: 2rem;
    background-color: #dc3545;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out, transform 0.1s ease-in-out;
  }
  .register-form .btn-cancel:hover {
    background-color: #6e140d;
    transform: scale(1.1);
  }
  .register-form .btn-cancel:active {
    transform: scale(0.95);
  }
</style>
