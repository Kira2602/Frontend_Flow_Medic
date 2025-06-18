<template>
  <div v-if="isVisible" class="register-popup-overlay" @click.self="closePopup">
    <div class="register-popup">
      <button class="close-btn" @click="closePopup">×</button>
      <div class="register-content">
        <!-- Imagen lateral -->
        <div class="register-image">
          <!-- Ajusta el src a la ruta de tu imagen -->
          <img
            src="@/assets/login_image.png"
            alt="Imagen registro"
            class="side-image"
          />
        </div>

        <!-- Formulario de registro -->
        <div class="register-form">
          <h2>Registrarse</h2>
          <form @submit.prevent="onRegister">
            <!-- Nombre -->
            <label for="nombre">Nombre</label>
            <input
              type="text"
              id="nombre"
              v-model.trim="nombre"
              placeholder="Nombre"
              @blur="validateField('nombre')"
              :class="{ 'error-border': errors.nombre }"
            />
            <span v-if="errors.nombre" class="error-message">{{ errors.nombre }}</span>

            <!-- Email -->
            <label for="email">Email</label>
            <input
              type="email"
              id="email"
              v-model.trim="email"
              placeholder="Email"
              @blur="validateField('email')"
              :class="{ 'error-border': errors.email }"
            />
            <span v-if="errors.email" class="error-message">{{ errors.email }}</span>

            <!-- Teléfono -->
            <label for="telefono">Teléfono</label>
            <input
              type="tel"
              id="telefono"
              v-model.trim="telefono"
              placeholder="+591 7XX-XXXXXX"
              @blur="validateField('telefono')"
              :class="{ 'error-border': errors.telefono }"
            />
            <span v-if="errors.telefono" class="error-message">{{ errors.telefono }}</span>

            <!-- Contraseña -->
            <label for="password">Contraseña</label>
            <div class="password-wrapper">
              <input
                :type="showPassword ? 'text' : 'password'"
                id="password"
                v-model="contrasenia"
                placeholder="Contraseña"
                @blur="validateField('contrasenia')"
                :class="{ 'error-border': errors.contrasenia }"
              />
              <span class="toggle-password" @click="togglePasswordVisibility">
                <font-awesome-icon
                  :icon="showPassword ? ['fas','eye'] : ['fas','eye-slash']"
                />
              </span>
            </div>
            <span v-if="errors.contrasenia" class="error-message">{{ errors.contrasenia }}</span>

            <!-- Botón Registrarse -->
            <button type="submit" class="btn-register">Registrarse</button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// Sin axios; solo emite evento al padre
import Swal from 'sweetalert2'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { library } from '@fortawesome/fontawesome-svg-core'
import { faEye, faEyeSlash } from '@fortawesome/free-solid-svg-icons'
library.add(faEye, faEyeSlash)

export default {
  name: 'RegisterPopup',
  components: { FontAwesomeIcon },
  props: {
    isVisible: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      nombre: '',
      email: '',
      telefono: '',
      contrasenia: '',
      showPassword: false,
      errors: {
        nombre: '',
        email: '',
        telefono: '',
        contrasenia: ''
      }
    }
  },
  methods: {
    closePopup() {
      this.$emit('close')
    },
    togglePasswordVisibility() {
      this.showPassword = !this.showPassword
    },
    validateField(field) {
      this.errors[field] = ''
      if (field === 'nombre') {
        if (!this.nombre) {
          this.errors.nombre = 'El nombre no puede estar vacío'
        }
      } else if (field === 'email') {
        const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
        if (!this.email) {
          this.errors.email = 'El email no puede estar vacío'
        } else if (!re.test(this.email)) {
          this.errors.email = 'Introduce un correo electrónico válido'
        }
      } else if (field === 'telefono') {
        const norm = this.telefono.replace(/[\s-]/g, '')
        const re = /^\+?\d{7,15}$/
        if (!this.telefono) {
          this.errors.telefono = 'El teléfono no puede estar vacío'
        } else if (!re.test(norm)) {
          this.errors.telefono = 'Introduce un número de teléfono válido'
        }
      } else if (field === 'contrasenia') {
        if (!this.contrasenia) {
          this.errors.contrasenia = 'La contraseña no puede estar vacía'
        } else if (this.contrasenia.length < 6) {
          this.errors.contrasenia = 'La contraseña debe tener al menos 6 caracteres'
        }
      }
    },
    validateForm() {
      this.validateField('nombre')
      this.validateField('email')
      this.validateField('telefono')
      this.validateField('contrasenia')
      return (
        !this.errors.nombre &&
        !this.errors.email &&
        !this.errors.telefono &&
        !this.errors.contrasenia
      )
    },
    onRegister() {
      if (!this.validateForm()) return

      const payload = {
        nombre: this.nombre,
        email: this.email,
        telefono: this.telefono,
        contrasenia: this.contrasenia
      }
      // Emitir al padre para backend o manejo externo
      this.$emit('register', payload)

      // Feedback simulado
      Swal.fire({
        icon: 'success',
        title: 'Registro simulado',
        text: 'Datos preparados para envío al backend',
        confirmButtonText: 'OK'
      })

      // Limpiar campos y cerrar popup
      this.nombre = ''
      this.email = ''
      this.telefono = ''
      this.contrasenia = ''
      this.showPassword = false
      this.closePopup()
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
}

/* Overlay */
.register-popup-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0, 0, 0, 0.2);
  display: flex; justify-content: center; align-items: center;
  overflow-x: auto;
  z-index: 9999;
}

/* Popup ancho fijo */
.register-popup {
  background-color: white;
  border-radius: 20px;
  width: 800px;
  padding: 20px;
  position: relative;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

/* Botón cerrar */
.close-btn {
  position: absolute;
  top: 12px; right: 12px;
  background: none; border: none;
  font-size: 24px; cursor: pointer;
}

/* Contenido */
.register-content {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  flex-wrap: nowrap;
}

/* Contenedor de la imagen lateral */
.register-image {
  flex: 0 0 360px;
  margin-right: 20px;
  display: flex; justify-content: center; align-items: center;
}
/* Ajustes de la imagen */
.register-image .side-image {
  width: 100%;
  max-width: 360px;
  height: auto;
  border-radius: 10px;
  object-fit: cover;
}

/* Formulario */
.register-form {
  flex: 0 0 300px;
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.register-form h2 {
  color: #121211;
  margin-bottom: 16px;
  font-size: 28px;
  align-self: center;
}
.register-form label {
  font-size: 14px;
  color: #333;
  margin-top: 12px;
  margin-bottom: 4px;
  font-weight: 500;
  width: 100%;
  text-align: left;
}
.register-form input {
  width: 100%;
  padding: 10px;
  margin-bottom: 4px;
  border: 1px solid #ddd;
  border-radius: 6px;
  font-size: 14px;
}
.register-form input:focus {
  outline: none;
  border-color: #ac3030;
  box-shadow: 0 0 0 2px rgba(172,48,48,0.2);
}
/* Errores */
.error-border {
  border-color: red !important;
}
.error-message {
  color: red;
  font-size: 12px;
  margin-top: 4px;
  margin-bottom: 8px;
  width: 100%;
  text-align: left;
}
/* Password con ícono */
.password-wrapper {
  position: relative;
  width: 100%;
  margin-bottom: 4px;
}
.password-wrapper input {
  width: 100%;
  padding-right: 36px;
}
.toggle-password {
  position: absolute;
  top: 50%;
  right: 10px;
  transform: translateY(-50%);
  cursor: pointer;
  color: #666;
  font-size: 1rem;
}
/* Botón Registrarse */
.btn-register {
  margin-top: 16px;
  width: 100%;
  background-color: #ac3030;
  color: white;
  border: none;
  border-radius: 20px;
  padding: 12px 0;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 2px 6px rgba(0,0,0,0.15);
}
.btn-register:hover {
  background-color: #efdbbf;
  color: #121211;
}

/* Media Queries */
@media (max-width: 992px) {
  .register-popup {
    width: 90%;
  }
  .register-content {
    flex-wrap: wrap;
    align-items: flex-start;
  }
  .register-image {
    flex: 0 0 100%;
    max-width: 100%;
    margin-bottom: 16px;
  }
  .register-form {
    flex: 0 0 100%;
    padding-left: 0;
    align-items: center;
    width: 90%;
  }
  .register-form h2 {
    font-size: 24px;
  }
  .register-form label,
  .register-form input,
  .password-wrapper,
  .btn-register {
    width: 100%;
    max-width: none;
  }
}
@media (max-width: 576px) {
  .register-popup {
    width: 95%;
    padding: 12px;
  }
  .register-form h2 {
    font-size: 20px;
    margin-bottom: 12px;
  }
  .register-form label,
  .register-form input,
  .password-wrapper,
  .btn-register {
    width: 100%;
    padding: 8px;
    font-size: 14px;
  }
  .toggle-password {
    right: 8px;
  }
}
</style>
