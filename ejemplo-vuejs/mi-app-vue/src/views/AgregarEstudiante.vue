<template>
  <div class="form-container">
    <h2>Agregar Nuevo Estudiante</h2>
    <form @submit.prevent="agregarEstudiante">
      <div class="form-group">
        <label for="nombre">Nombre:</label>
        <input v-model="form.nombre" id="nombre" required />
      </div>

      <div class="form-group">
        <label for="apellido">Apellido:</label>
        <input v-model="form.apellido" id="apellido" required />
      </div>

      <div class="form-group">
        <label for="cedula">Cédula:</label>
        <input v-model="form.cedula" id="cedula" required />
      </div>

      <div class="form-group">
        <label for="correo">Correo:</label>
        <input v-model="form.correo" id="correo" type="email" required />
      </div>

      <button type="submit">Guardar</button>
      <p v-if="error" class="error-message">{{ error }}</p>
    </form>

    <router-link to="/estudiantes" class="back-button">Cancelar</router-link>
  </div>
</template>

<script>
import api from "@/api/axios";
import router from "@/router";

export default {
  name: "AgregarEstudiante",
  data() {
    return {
      form: {
        nombre: "",
        apellido: "",
        cedula: "",
        correo: "",
      },
      error: null,
    };
  },
  methods: {
    async agregarEstudiante() {
      try {
        this.error = null;
        const response = await api.post("estudiantes/", this.form);
        console.log("Estudiante agregado:", response.data);
        router.push("/estudiantes"); // redirige a la lista
      } catch (err) {
        console.error("Error al agregar estudiante:", err.response || err);
        this.error = "No se pudo agregar el estudiante. Revisa los campos.";
      }
    },
  },
};
</script>

<style scoped>
.form-container {
  max-width: 600px;
  margin: 40px auto;
  padding: 20px;
  background-color: #fff;
  border: 1px solid #ccc;
  border-radius: 8px;
}

h2 {
  text-align: center;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

label {
  display: block;
  font-weight: bold;
  margin-bottom: 5px;
}

input {
  width: 100%;
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

button {
  width: 100%;
  padding: 10px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1em;
  cursor: pointer;
}

button:hover {
  background-color: #218838;
}

.error-message {
  color: red;
  margin-top: 10px;
  text-align: center;
}

.back-button {
  display: block;
  text-align: center;
  margin-top: 20px;
  color: #007bff;
  text-decoration: underline;
}
</style>
