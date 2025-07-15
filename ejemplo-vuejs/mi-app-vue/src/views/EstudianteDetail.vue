<template>
  <div class="estudiante-detail-container">
    <h2>Detalle del Estudiante</h2>
    <p v-if="loading">Cargando detalles...</p>
    <p v-if="error" class="error-message">{{ error }}</p>

    <div v-else-if="estudiante">
      <div v-if="editandoEstudiante">
        <label>Nombre:</label>
        <input v-model="estudiante.nombre" />
        <label>Apellido:</label>
        <input v-model="estudiante.apellido" />
        <label>Cédula:</label>
        <input v-model="estudiante.cedula" />
        <label>Correo:</label>
        <input v-model="estudiante.correo" />
        <button @click="guardarEstudiante">Guardar</button>
        <button @click="cancelarEdicionEstudiante">Cancelar</button>
      </div>

      <div v-else>
        <h3>{{ estudiante.nombre }} {{ estudiante.apellido }}</h3>
        <p><strong>Cédula:</strong> {{ estudiante.cedula }}</p>
        <p><strong>Correo:</strong> {{ estudiante.correo }}</p>
        <button @click="editandoEstudiante = true">Editar Estudiante</button>
      </div>

      <h4>Números Telefónicos:</h4>
      <ul v-if="numerosTelefonicos.length">
        <li v-for="(numero, index) in numerosTelefonicos" :key="numero.url">
          <div v-if="numero.editing">
            <input v-model="numero.telefono" placeholder="Teléfono" />
            <input v-model="numero.tipo" placeholder="Tipo (fijo/móvil)" />
            <button @click="guardarNumero(numero)">Guardar</button>
            <button @click="cancelarEdicionNumero(numero)">Cancelar</button>
          </div>
          <div v-else>
            {{ numero.telefono }} ({{ numero.tipo }})
            <button @click="numero.editing = true">Editar</button>
            <button @click="eliminarNumero(numero, index)">Eliminar</button>
          </div>
        </li>
      </ul>
      <p v-else>No tiene números telefónicos registrados.</p>

      <h4>Agregar Nuevo Número:</h4>
      <form @submit.prevent="crearNumero">
        <input v-model="nuevoNumero.telefono" placeholder="Teléfono" required />
        <input
          v-model="nuevoNumero.tipo"
          placeholder="Tipo (fijo/móvil)"
          required
        />
        <button type="submit">Agregar</button>
      </form>

      <router-link :to="{ name: 'EstudiantesList' }" class="back-button">
        Volver al Listado
      </router-link>
    </div>

    <p v-else>Estudiante no encontrado.</p>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudianteDetail",
  props: ["estudianteUrl"],
  data() {
    return {
      estudiante: null,
      numerosTelefonicos: [],
      nuevoNumero: {
        telefono: "",
        tipo: "",
      },
      editandoEstudiante: false,
      loading: true,
      error: null,
    };
  },
  async created() {
    const decodedUrl = decodeURIComponent(this.estudianteUrl);
    await this.fetchEstudianteDetail(decodedUrl);
    await this.fetchNumerosTelefonicos(decodedUrl);
  },
  methods: {
    async fetchEstudianteDetail(url) {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get(url);
        this.estudiante = response.data;
      } catch (err) {
        console.error(
          "Error al cargar detalle del estudiante:",
          err.response || err
        );
        this.error = "No se pudo cargar el detalle del estudiante.";
      } finally {
        this.loading = false;
      }
    },
    async fetchNumerosTelefonicos(estudianteApiUrl) {
      try {
        const response = await api.get("numerosts/");
        this.numerosTelefonicos = response.data.results
          .filter((numero) => numero.estudiante === estudianteApiUrl)
          .map((numero) => ({ ...numero, editing: false }));
      } catch (err) {
        console.error(
          "Error al cargar números telefónicos:",
          err.response || err
        );
      }
    },
    async guardarEstudiante() {
      try {
        await api.put(this.estudiante.url, this.estudiante);
        this.editandoEstudiante = false;
      } catch (err) {
        console.error("Error al guardar estudiante:", err.response || err);
        alert("No se pudo guardar el estudiante.");
      }
    },
    cancelarEdicionEstudiante() {
      this.editandoEstudiante = false;
      this.fetchEstudianteDetail(decodeURIComponent(this.estudianteUrl)); // recarga original
    },
    async crearNumero() {
      try {
        const response = await api.post("numerosts/", {
          telefono: this.nuevoNumero.telefono,
          tipo: this.nuevoNumero.tipo,
          estudiante: decodeURIComponent(this.estudianteUrl),
        });
        this.numerosTelefonicos.push({ ...response.data, editing: false });
        this.nuevoNumero.telefono = "";
        this.nuevoNumero.tipo = "";
      } catch (err) {
        console.error("Error al crear número:", err.response || err);
        alert("No se pudo crear el número.");
      }
    },
    cancelarEdicionNumero(numero) {
      numero.editing = false;
      this.fetchNumerosTelefonicos(decodeURIComponent(this.estudianteUrl)); // refresca
    },
    async guardarNumero(numero) {
      try {
        await api.put(numero.url, {
          telefono: numero.telefono,
          tipo: numero.tipo,
          estudiante: decodeURIComponent(this.estudianteUrl),
        });
        numero.editing = false;
      } catch (err) {
        console.error("Error al guardar número:", err.response || err);
        alert("No se pudo guardar el número.");
      }
    },
    async eliminarNumero(numero, index) {
      if (!confirm("¿Seguro que quieres eliminar este número?")) return;
      try {
        await api.delete(numero.url);
        this.numerosTelefonicos.splice(index, 1);
      } catch (err) {
        console.error("Error al eliminar número:", err.response || err);
        alert("No se pudo eliminar el número.");
      }
    },
  },
};
</script>

<style scoped>
.estudiante-detail-container {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
  text-align: left;
}

h2,
h3,
h4 {
  text-align: center;
  color: #333;
  margin-bottom: 15px;
}

label {
  display: block;
  margin-top: 10px;
  font-weight: bold;
}

input {
  padding: 6px;
  width: 100%;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 5px 10px;
  margin-right: 5px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

ul {
  list-style: none;
  padding-left: 0;
}

li {
  margin-bottom: 10px;
  display: flex;
  gap: 10px;
  align-items: center;
}

form {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 20px;
}

.back-button {
  display: block;
  width: fit-content;
  margin: 20px auto 0;
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border-radius: 5px;
  text-decoration: none;
  text-align: center;
  transition: background-color 0.3s ease;
}

.back-button:hover {
  background-color: #218838;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}
</style>
