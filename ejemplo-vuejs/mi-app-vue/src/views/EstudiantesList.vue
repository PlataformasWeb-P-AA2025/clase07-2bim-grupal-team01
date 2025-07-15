<template>
  <div class="estudiantes-list-container">
    <h2>Listado de Estudiantes</h2>
    <p v-if="loading">Cargando estudiantes...</p>
    <p v-if="error" class="error-message">{{ error }}</p>
    <ul v-else-if="estudiantes.length">
      <li
        v-for="estudiante in estudiantes"
        :key="estudiante.url"
        class="estudiante-item"
      >
        <router-link
          :to="{
            name: 'EstudianteDetail',
            params: { estudianteUrl: estudiante.url },
          }"
        >
          {{ estudiante.nombre }} {{ estudiante.apellido }} (Cédula:
          {{ estudiante.cedula }})
        </router-link>

        <button
          class="delete-button"
          @click="eliminarEstudiante(estudiante.url)"
        >
          Eliminar
        </button>
      </li>
    </ul>
    <p v-else>No hay estudiantes registrados.</p>
    <router-link to="/estudiantes/nuevo" class="add-button">
      Agregar Nuevo Estudiante
    </router-link>
  </div>
</template>

<script>
import api from "@/api/axios";

export default {
  name: "EstudiantesList",
  data() {
    return {
      estudiantes: [],
      loading: true,
      error: null,
    };
  },
  async created() {
    await this.fetchEstudiantes();
  },
  methods: {
    async fetchEstudiantes() {
      try {
        this.loading = true;
        this.error = null;
        const response = await api.get("estudiantes/");
        this.estudiantes = response.data.results || response.data;
      } catch (err) {
        console.error("Error al cargar estudiantes:", err.response || err);
        this.error =
          "No se pudieron cargar los estudiantes. Asegúrate de estar logueado.";
      } finally {
        this.loading = false;
      }
    },
    async eliminarEstudiante(estudianteUrl) {
      const confirmar = confirm(
        "¿Estás seguro de que deseas eliminar este estudiante?"
      );
      if (!confirmar) return;

      try {
        await api.delete(estudianteUrl);
        this.fetchEstudiantes();
      } catch (err) {
        console.error("Error al eliminar estudiante:", err.response || err);
        this.error = "No se pudo eliminar el estudiante. Intenta nuevamente.";
      }
    },
  },
};
</script>

<style scoped>
.estudiantes-list-container {
  max-width: 800px;
  margin: 50px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

h2 {
  text-align: center;
  color: #333;
  margin-bottom: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

.estudiante-item {
  padding: 10px 0;
  border-bottom: 1px solid #eee;
}

.estudiante-item:last-child {
  border-bottom: none;
}

.estudiante-item a {
  text-decoration: none;
  color: #007bff;
  font-weight: bold;
}

.estudiante-item a:hover {
  text-decoration: underline;
}

.add-button {
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

.add-button:hover {
  background-color: #218838;
}

.error-message {
  color: red;
  text-align: center;
  margin-top: 10px;
}

.delete-button {
  margin-left: 15px;
  padding: 5px 10px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 0.9em;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.delete-button:hover {
  background-color: #c82333;
}
</style>
