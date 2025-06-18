<template>
  <div
    v-if="isVisible"
    class="register-hospital-popup-overlay"
    @click.self="closePopup"
  >
    <div class="register-hospital-popup">
      <!-- Botón cerrar -->
      <button class="close-btn" @click="closePopup">×</button>

      <div class="popup-content">
        <!-- Título -->
        <h2>Registrar Hospital</h2>

        <!-- Contenedor del GIF centrado -->
        <div class="icon-container">
          <!-- GIF desde assets -->
          <img src="@/assets/medicos.gif" alt="Médicos" class="gif-animation" />
        </div>

        <!-- Formulario -->
        <form @submit.prevent="onRegister">
          <!-- Nombre del hospital -->
          <div class="form-group">
            <label for="hospitalName">Nombre</label>
            <input
              type="text"
              id="hospitalName"
              v-model.trim="nombre"
              placeholder="Nombre del hospital"
              @blur="validateField('nombre')"
              :class="{ 'error-border': errors.nombre }"
            />
            <span v-if="errors.nombre" class="error-message">{{ errors.nombre }}</span>
          </div>

          <!-- Ubicación -->
          <div class="form-group">
            <label for="ubicacion">Ubicación</label>
            <input
              type="text"
              id="ubicacion"
              v-model.trim="ubicacion"
              placeholder="Ubicación del hospital"
              @blur="validateField('ubicacion')"
              :class="{ 'error-border': errors.ubicacion }"
            />
            <span v-if="errors.ubicacion" class="error-message">{{ errors.ubicacion }}</span>
          </div>

          <!-- Botón Registrar -->
          <button type="submit" class="btn-register">Registrar</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import Swal from 'sweetalert2'

export default {
  name: 'RegistroHospitalPopup',
  props: {
    isVisible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      nombre: '',
      ubicacion: '',
      errors: {
        nombre: '',
        ubicacion: ''
      }
    }
  },
  watch: {
    isVisible(val) {
      if (!val) this.clearForm()
    }
  },
  methods: {
    closePopup() {
      this.$emit('close')
      this.clearForm()
    },
    validateField(field) {
      this.errors[field] = ''
      if (field === 'nombre') {
        if (!this.nombre) this.errors.nombre = 'El nombre no puede estar vacío'
        else if (this.nombre.length < 3) this.errors.nombre = 'Debe tener al menos 3 caracteres'
      }
      if (field === 'ubicacion') {
        if (!this.ubicacion) this.errors.ubicacion = 'La ubicación no puede estar vacía'
        else if (this.ubicacion.length < 3) this.errors.ubicacion = 'Debe tener al menos 3 caracteres'
      }
    },
    validateForm() {
      this.validateField('nombre')
      this.validateField('ubicacion')
      return !this.errors.nombre && !this.errors.ubicacion
    },
    clearForm() {
      this.nombre = ''
      this.ubicacion = ''
      this.errors.nombre = ''
      this.errors.ubicacion = ''
    },
    async onRegister() {
      if (!this.validateForm()) return

      try {
        // El blueprint está montado en "/hospitals/" para POST
        const { data } = await axios.post(
          'http://127.0.0.1:5000/hospitals/',
          {
            nombre: this.nombre,
            ubicacion: this.ubicacion
          }
        )

        Swal.fire({
          icon: 'success',
          title: 'Hospital creado',
          text: `Hospital: ${data.nombre}`
        })

        this.clearForm()
        this.closePopup()

      } catch (err) {
        // Si el OPTIONS preflight falla, revisa CORS en tu servidor
        if (err.response && err.response.data.error) {
          Swal.fire('Error', err.response.data.error, 'error')
        } else if (err.response) {
          Swal.fire('Error', 'Revisa tus datos', 'error')
        } else {
          Swal.fire('Error de red', 'No se pudo conectar al servidor', 'error')
        }
      }
    }
  }
}
</script>


<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

/* Box-sizing global */
*,
*::before,
*::after {
  box-sizing: border-box;
}
* {
  font-family: 'Poppins', sans-serif;
  margin: 0;
  padding: 0;
}

/* Overlay */
.register-hospital-popup-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.2);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  overflow-y: auto;
  padding: 20px; /* espacio en móviles */
}

/* Popup */
.register-hospital-popup {
  background-color: white;
  border-radius: 12px;
  width: 400px;
  max-width: 100%;
  padding: 20px 24px;
  position: relative;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* Botón cerrar */
.close-btn {
  position: absolute;
  top: 12px;
  right: 12px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: #333;
}

/* Contenido */
.popup-content {
  margin-top: 8px;
}
.popup-content h2 {
  color: #af8a8a;
  font-size: 24px;
  text-align: center;
  margin-bottom: 12px;
}

/* Contenedor del GIF */
.icon-container {
  width: 100%;
  display: flex;
  justify-content: center;
  margin-bottom: 16px;
}
.gif-animation {
  width: 120px;
  height: auto;
  border-radius: 8px;
  /* Ajusta width según prefieras */
}

/* Formulario */
.form-group {
  margin-bottom: 16px;
  text-align: left;
}
.form-group label {
  display: block;
  font-size: 14px;
  color: #333;
  margin-bottom: 6px;
  font-weight: 500;
}
.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.2s;
}
.form-group input:focus {
  outline: none;
  border-color: #ac3030;
  box-shadow: 0 0 0 2px rgba(172, 48, 48, 0.2);
}

/* Error */
.error-border {
  border-color: red !important;
}
.error-message {
  color: red;
  font-size: 12px;
  margin-top: 4px;
}

/* Botón Registrar */
.btn-register {
  width: 100%;
  background-color: #ac3030;
  color: white;
  border: none;
  padding: 12px;
  border-radius: 20px;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 8px;
}
.btn-register:hover {
  background-color: #68181f;
  color: white;
}

/* Responsive */
@media (max-width: 480px) {
  .register-hospital-popup {
    width: 95%;
    padding: 16px;
  }
  .popup-content h2 {
    font-size: 20px;
  }
  .gif-animation {
    width: 100px;
  }
}
</style>
