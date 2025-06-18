<template>
  <nav class="navbar">
    <!-- Logo -->
    <div class="logo" @click="goBack">
      <img src="../assets/logo_fm.png" alt="Logo" class="logo-img" />
    </div>

    <!-- Home: si no está logeado → Iniciar Sesión; si está logeado → Usuario + Cerrar Sesión + Volver -->
    <div v-if="isHome" class="right-block">
      <div v-if="isLoggedIn" class="user-info">
        <i class="fas fa-user"></i>
        <span class="user-name">{{ userName }}</span>
      </div>
      <button
        v-if="!isLoggedIn"
        class="btn btn-outline"
        @click="navigate('Login')"
      >
        Iniciar Sesión
      </button>
      <button
        v-else
        class="btn btn-outline"
        @click="logout"
      >
        Cerrar Sesión
      </button>
      <button
        class="btn btn-outline"
        @click="goBack"
      >
        Volver
      </button>
    </div>

    <!-- menú-usuario: Usuario + Inicio + Cerrar Sesión -->
    <div v-else-if="isMenuUsuario" class="right-block">
      <div v-if="isLoggedIn" class="user-info">
        <i class="fas fa-user"></i>
        <span class="user-name">{{ userName }}</span>
      </div>
      <button class="btn btn-outline" @click="goInicio">Inicio</button>
      <button class="btn btn-outline" @click="logout">Cerrar Sesión</button>
    </div>

    <!-- urgencias: Usuario + Inicio + Volver + Cerrar Sesión -->
    <div v-else-if="isUrgencias" class="right-block">
      <div v-if="isLoggedIn" class="user-info">
        <i class="fas fa-user"></i>
        <span class="user-name">{{ userName }}</span>
      </div>
      <button class="btn btn-outline" @click="goInicio">Inicio</button>
      <button class="btn btn-outline" @click="goBack">Volver</button>
      <button class="btn btn-outline" @click="logout">Cerrar Sesión</button>
    </div>

    <!-- Login: Volver + Inicio -->
    <div v-else-if="isLogin" class="right-block">
      <button class="btn btn-outline" @click="goInicio">Inicio</button>
      <button class="btn btn-outline" @click="goBack">Volver</button>
    </div>

    <!-- Resto de rutas: menú completo + Usuario -->
    <div v-else class="menu-desktop">
      <ul class="menu">
        <li><a @click.prevent="navigate('Home')" class="menu-link">Inicio</a></li>
        <li><a @click.prevent="navigate('Hospitales')" class="menu-link">Hospitales</a></li>
      </ul>
      <div class="right-block">
        <div v-if="isLoggedIn" class="user-info">
          <i class="fas fa-user"></i>
          <span class="user-name">{{ userName }}</span>
        </div>
        <button
          v-if="mode === 'user'"
          class="btn btn-outline"
          @click="goInicio"
        >
          Inicio
        </button>
        <button
          v-if="mode === 'user'"
          class="btn btn-outline"
          @click="goBack"
        >
          Volver
        </button>
        <button
          v-else-if="mode === 'public'"
          class="btn btn-outline"
          @click="navigate('Login')"
        >
          Iniciar Sesión
        </button>
        <button
          v-if="onSpecialRoute"
          class="btn btn-outline"
          @click="goBack"
        >
          Volver
        </button>
      </div>
    </div>

    <!-- Mobile menu -->
    <div class="menuToggle">
      <input type="checkbox" id="menu-btn" v-model="menuOpen" />
      <span></span><span></span><span></span>
      <ul class="menuItem">
        <li><a @click.prevent="navigate('Home')" class="menu-link">Inicio</a></li>
        <li><a @click.prevent="navigate('Hospitales')" class="menu-link">Hospitales</a></li>
        <li v-if="mode === 'public'">
          <button class="btn btn-outline" @click="navigate('Login')">Iniciar Sesión</button>
        </li>
        <li v-else-if="isLoggedIn" class="user-info-mobile">
          <i class="fas fa-user"></i>
          <span class="user-name">{{ userName }}</span>
          <button class="btn btn-outline" @click="logout">Cerrar Sesión</button>
        </li>
      </ul>
    </div>
  </nav>
</template>

<script>
export default {
  name: 'NavbarGeneral',
  props: {
    mode: {
      type: String,
      default: 'public'
    },
    userName: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      menuOpen: false
    };
  },
  computed: {
    isLoggedIn() {
      return this.mode === 'user' && this.userName !== '';
    },
    isHome() {
      return this.$route.name === 'Home' || this.$route.path === '/';
    },
    isLogin() {
      return this.$route.name === 'Login' || this.$route.path.includes('login');
    },
    isMenuUsuario() {
      return (
        this.$route.name.toLowerCase().includes('menu-usuario') ||
        this.$route.path.includes('/menu-usuario')
      );
    },
    isUrgencias() {
      return (
        this.$route.name.toLowerCase().includes('urgencias') ||
        this.$route.path.includes('/urgencias')
      );
    },
    onSpecialRoute() {
      const specials = [
        'mis-mascotas',
        'perfil-mascota',
        'historial-actividades',
        'salud',
        'calendario',
        'notificaciones'
      ];
      return specials.includes(this.$route.name);
    }
  },
  methods: {
    navigate(name) {
      this.$router.push({ name });
      this.menuOpen = false;
    },
    goBack() {
      this.$router.go(-1);
    },
    goInicio() {
      this.$router.push({ name: 'Home' });
    },
    logout() {
      // Limpia tu store/token aquí
      this.$router.push({ name: 'Login' });
    }
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');
@import '@fortawesome/fontawesome-free/css/all.css';

.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px;
  background: #68181f;
  border-radius: 50px;
  box-shadow: 0 6px 20px rgba(0,0,0,0.2);
  margin: 20px;
}

.logo-img {
  width: 70px;
  height: 70px;
  cursor: pointer;
}

.menu-desktop {
  flex: 1;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 2rem;
}

.menu {
  display: flex;
  gap: 1.5rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

.menu-link {
  color: #fff;
  font-size: 16px;
  font-weight: 500;
  text-decoration: none;
  transition: color 0.3s;
}
.menu-link:hover {
  color: #f4cfc6;
}

.right-block {
  display: flex;
  align-items: center;
  gap: 1rem;
}

/* Bloque de info de usuario */
.user-info,
.user-info-mobile {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: #efdbbf;
  font-weight: 600;
}
.user-info i,
.user-info-mobile i {
  font-size: 1.2rem;
}

/* Botones */
.btn {
  padding: 8px 18px;
  border-radius: 30px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s;
}
.btn-outline {
  background: transparent;
  color: #efdbbf;
  border: 2px solid #efdbbf;
}
.btn-outline:hover {
  background: #efdbbf;
  color: #121211;
}

/* Mobile */
.menuToggle {
  display: none;
  flex-direction: column;
  align-items: flex-end;
}
.menuToggle input {
  display: none;
}
.menuToggle span {
  width: 25px;
  height: 3px;
  background: #fff;
  margin: 3px 0;
  transition: all 0.3s;
}
.menuItem {
  position: absolute;
  top: 60px;
  right: 20px;
  background: #68181f;
  border-radius: 10px;
  display: none;
  flex-direction: column;
  padding: 10px;
}
.menuToggle input:checked ~ .menuItem {
  display: flex;
}
.menuItem li {
  list-style: none;
  margin: 8px 0;
}

@media (max-width: 768px) {
  .menu-desktop {
    display: none;
  }
  .menuToggle {
    display: flex;
  }
}
</style>
