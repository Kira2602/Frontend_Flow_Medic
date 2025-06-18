<template>
  <div class="min-h-screen font-sans">
    <!-- Navbar -->
    <NavbarGeneral :mode="mode" :userName="userName" />

    <!-- Hero Section rediseñada -->
    <section class="hero">
      <!-- IZQUIERDA: Título + CTA -->
      <div class="hero-content">
        <h1>✨ Bienvenido a FlowMedic 🚑</h1>
        <p class="text-lg">
          Monitorea y predice tiempos de espera en urgencias hospitalarias.
        </p>
      </div>
      <!-- DERECHA: Imagen recortada -->
      <div class="hero-image">
        <div class="circular-image">
          <img src="../assets/img_header.png" alt="Interior hospitalario" />
        </div>
      </div>
    </section>

    <!-- Sección de Hospitales -->
    <section class="mt-10">
      <HospitalCards />
    </section>

    <!-- Footer -->
    <FooterComponent />
  </div>
</template>

<script>
import NavbarGeneral   from '@/components/NavbarGeneral.vue'
import HospitalCards   from '@/components/HospitalCards.vue'
import FooterComponent from '@/components/Footer_Component.vue'

export default {
  name: 'HomePage',
  components: {
    NavbarGeneral,
    HospitalCards,
    FooterComponent
  },
  computed: {
    // ¿Hay token guardado?
    isLoggedIn() {
      return !!localStorage.getItem('authToken')
    },
    // Modo para el Navbar
    mode() {
      return this.isLoggedIn ? 'user' : 'public'
    },
    // Nombre de usuario
    userName() {
      return localStorage.getItem('nombre') || ''
    }
  },
  methods: {
    logout() {
      // Elimina las claves correctas
      localStorage.removeItem('authToken')
      localStorage.removeItem('nombre')
      // Opcionalmente limpia otros
      localStorage.removeItem('Usuario_id_usuario')
      localStorage.removeItem('hospital_id')
      this.$router.push({ name: 'Login' })
    }
  }
}
</script>

<style scoped>
.hero {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 3rem 1rem;
  background: #fff;
  flex-wrap: wrap;
}

/* Texto + CTA */
.hero-content {
  flex: 1 1 350px;
}
.hero-content h1 {
  font-size: 2.8rem;
  line-height: 1.2;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
}
.thumbnail {
  display: inline-block;
  margin-left: 0.75rem;
}
.thumbnail img {
  width: 80px;
  height: 48px;
  object-fit: cover;
  border-radius: 30px 20px;
}


/* Imagen recortada con borde punteado */
.hero-image {
  flex: 1 1 400px;
  position: relative;
  min-height: 300px;
}
.circular-image {
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  margin: auto;
  width: 100%;
  height: 100%;
  border-radius: 190px 0 0 140px;
  border-top: 3px dotted #979696;
  border-left: 3px dotted #979696;
  border-bottom: 3px dotted #979696;
  overflow: hidden;
  padding: 0.5rem;
}
.circular-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 180px 0 0 135px;
}

/* Responsive */
@media (max-width: 768px) {
  .hero {
    flex-direction: column;
    gap: 2rem;
  }
  .hero-image {
    height: 250px;
  }
}
</style>
