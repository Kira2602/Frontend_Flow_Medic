<template>
  <div>
    <NavbarComponent />

    <!-- Popup de registro de usuario -->
    <RegisterPopup
      :isVisible="showRegisterPopup"
      @close="showRegisterPopup = false"
      @register="handleUserRegister" 
    />

    <!-- Popup de registro de hospital -->
    <RegisterHospitalPopup
      :isVisible="showHospitalPopup"
      @close="showHospitalPopup = false"
      @register-hospital="handleHospitalRegister"
    />

    <!-- Contenedor principal -->
    <div class="grid">
      <!-- Sección izquierda: formulario -->
      <div class="order__left centered">
        <div class="form">
          <!-- Logo -->
          <div class="logo">
            <h1 style="color: #ac3030; font-size: 28px; font-weight: bold;">
              <font-awesome-icon :icon="['fas', 'circle-user']" /> FlowMedic
            </h1>
          </div>
          <h4>
            Bienvenido a FlowMedic. Plataforma oficial para monitoreo de urgencias hospitalarias.
          </h4>

          <!-- Formulario de login -->
          <form @submit.prevent="login">
            <input
              type="email"
              placeholder="Correo"
              v-model.trim="email"
              autocomplete="off"
            />
            <input
              type="password"
              placeholder="Contraseña"
              v-model.trim="password"
              autocomplete="off"
            />
            <button type="submit" class="login__button">
              <font-awesome-icon :icon="['fas', 'sign-in-alt']" /> Ingresar
            </button>
          </form>

          <!-- Sección registro de usuario -->
          <div class="register-prompt">
            <label>¿Aún no te has registrado?</label>
            <button class="register__button" @click="openRegisterPopup">
              <font-awesome-icon :icon="['fas', 'user-plus']" class="icon-btn" />
              Registrarse
            </button>
          </div>
          <!-- Sección registro de hospitales -->
          <div class="hospital-register-prompt">
            <label>¿Eres administrador de un hospital?</label>
            <button class="hospital-register__button" @click="openHospitalPopup">
              <font-awesome-icon :icon="['fas', 'hospital']" class="icon-btn" />
              Registrar Hospital
            </button>
          </div>
        </div>
      </div>

      <!-- Sección derecha: imagen decorativa -->
      <div class="order__right centered no__overflow">
        <img
          class="img"
          src="../assets/fondo_login.png"
          alt="Decoración médica"
        />
      </div>
    </div>
  </div>
</template>

<script>
import NavbarComponent from '@/components/NavbarGeneral.vue'
import RegisterPopup from '@/components/RegisterPopup.vue'
import RegisterHospitalPopup from '@/components/RegisterHospitalPopup.vue'
import Swal from 'sweetalert2'
import axios from 'axios' // si usas login con backend; si no, elimina axios y lógica de login

// FontAwesome
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { library } from '@fortawesome/fontawesome-svg-core'
import {
  faCircleUser,
  faSignInAlt,
  faUserPlus,
  faHospital
} from '@fortawesome/free-solid-svg-icons'

library.add(faCircleUser, faSignInAlt, faUserPlus, faHospital)

export default {
  name: 'LoginPage',
  components: {
    FontAwesomeIcon,
    NavbarComponent,
    RegisterPopup,
    RegisterHospitalPopup
  },
  data() {
    return {
      email: '',
      password: '',
      showRegisterPopup: false,
      showHospitalPopup: false
    }
  },
  methods: {
    async login() {
      // Si no tienes backend aún, podrías simular o simplemente validar campos vacíos
      if (!this.email || !this.password) {
        Swal.fire({
          icon: 'warning',
          title: 'Campos incompletos',
          text: 'Por favor ingresa tu correo y contraseña.',
        })
        return
      }
      // Si tienes backend para login, conserva lógica axios aquí:
      try {
        const response = await axios.post('http://127.0.0.1:5000/usuario/login', {
          email: this.email,
          contrasenia: this.password
        })
        // ... manejo de respuesta ...
        if (response.data.id_usuario) {
          localStorage.setItem('Usuario_id_usuario', response.data.id_usuario)
        }
        if (response.data.hospital_id) {
          localStorage.setItem('hospital_id', response.data.hospital_id)
        }
        localStorage.setItem('authToken', response.data.token)
        localStorage.setItem('nombre', response.data.nombre)

        Swal.fire({
          icon: 'success',
          title: 'Login exitoso',
          text: 'Has iniciado sesión correctamente',
        })
        this.$router.push('/menu-usuario')
      } catch (error) {
        Swal.fire({
          icon: 'error',
          title: 'Login fallido',
          text: 'Verifica tus credenciales.',
        })
      }
    },
    openRegisterPopup() {
      this.showRegisterPopup = true
    },
    handleUserRegister(payload) {
      // Ejemplo de recepción de registro de usuario simulado
      // payload = { nombre, email, telefono, contrasenia }
      console.log('Datos registro usuario:', payload)
      Swal.fire({
        icon: 'success',
        title: 'Registro Usuario (simulado)',
        text: `Usuario "${payload.nombre}" listo para registrar.`,
      })
      // Cerrar popup (ya lo cierra el propio componente al emitir close)
      this.showRegisterPopup = false
    },

    openHospitalPopup() {
      this.showHospitalPopup = true
    },
    handleHospitalRegister(payload) {
      // payload = { nombre, ubicacion }
      console.log('Datos hospital:', payload)
      Swal.fire({
        icon: 'success',
        title: 'Registro Hospital (simulado)',
        text: `Hospital "${payload.nombre}" en "${payload.ubicacion}" listo para registrar.`,
      })
      // cerrar popup (se cierra automáticamente en RegistroHospitalPopup tras emitir close)
      this.showHospitalPopup = false
    }
  }
}
</script>

<style scoped>
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  height: 100vh;
}
.order__left {
  order: 1;
  padding: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh; 
}
.order__right {
  order: 2;
}
.centered {
  display: flex;
  align-items: center;
  justify-content: center;
}
.no__overflow {
  display: flex;
  align-items: center;
  overflow: hidden;
}
.form {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}
form {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100%;
}

.logo {
  margin-bottom: 12px;
  text-align: center;
}
h4 {
  color: #121211;
  text-align: center;
  margin-bottom: 30px;
  letter-spacing: 0.2px;
  line-height: 20px;
  font-size: 15px;
  max-width: 280px;
}
input[type=email],
input[type=password] {
  width: 260px;
  padding: 12px 16px;
  margin: 10px 0;
  border: 1px solid #c8c3cf;
  border-radius: 4px;
  background-color: #F6F5F7;
  font-size: 15px;
  color: #121211;
}
input:focus {
  border-color: #ac3030;
  background: #FDFCFE;
  box-shadow: 0 0 0 0.25rem #efdbbf;
}
.login__button {
  width: 260px;
  margin-top: 16px;
  border: none;
  background-color: #ac3030;
  padding: 12px;
  color: white;
  border-radius: 4px;
  font-size: 15px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}
.login__button :deep(svg) {
  margin-right: 8px;
}
.login__button:hover {
  background-color: #68181f;
}

/* Estilos para la sección de registro de usuario */
.register-prompt {
  margin-top: 20px;
  text-align: center;
}
.register-prompt label {
  display: block;
  margin-bottom: 8px;
  color: #333;
  font-size: 14px;
}
.register__button {
  width: 260px;
  border: 2px solid #121211;
  background-color: #efdbbf;
  padding: 10px;
  color: #121211;
  border-radius: 4px;
  font-size: 15px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.register__button :deep(svg) {
  margin-right: 8px;
}
.register__button:hover {
  background-color: #e3b87c;
}

/* Estilos para la sección de registro de hospitales */
.hospital-register-prompt {
  margin-top: 15px;
  text-align: center;
}
.hospital-register-prompt label {
  display: block;
  margin-bottom: 8px;
  color: #333;
  font-size: 14px;
}
.hospital-register__button {
  width: 260px;
  border: none;
  background-color: #ac3030;
  padding: 10px;
  color: white;
  border-radius: 4px;
  font-size: 15px;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
.hospital-register__button :deep(svg) {
  margin-right: 8px;
}
.hospital-register__button:hover {
  background-color: #68181f;
}

.img {
  height: 100%;
  object-fit: cover;
}
@media only screen and (max-width: 800px) {
  .grid {
    grid-template-columns: 1fr;
  }
  .order__left {
    order: 2;
    padding: 20px;
  }
  .order__right {
    order: 1;
  }
  .img {
    width: 100vw;
  }
  .login__button,
  .register__button,
  .hospital-register__button {
    width: 90%;
  }
}
/* Pequeña clase para separar iconos en botones */
.icon-btn {
  margin-right: 8px;
}
</style>
