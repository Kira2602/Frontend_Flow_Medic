<template>
  <div class="cards-wrapper">
    <div class="cards-container">
      <div
        class="card"
        v-for="h in hospitals"
        :key="h.id"
      >
        <div class="card-head">
          <div class="card-product-img">
            <!-- pon aquí una imagen por hospital, o un placeholder -->
            <img src="@/assets/hospital_image.png" alt="hospital" />
          </div>
        </div>

        <div class="card-body">
          <h3 class="card-title">{{ h.nombre }}</h3>
          <p class="card-text">{{ h.ubicacion }}</p>
          <button
            class="status-btn"
            :style="{ backgroundColor: h.available ? '#c7ea46' : '#e00000' }"
          >
            {{ h.available ? 'Disponible' : 'Lleno' }}
          </button>
        </div>

        <button
          class="view-btn"
          aria-label="Ver"
          @click="openPopup(h)"
        >
          <span class="dots">•••</span>
        </button>
      </div>
    </div>

    <EstadoPopup
      v-if="popupVisible"
      :hospital="selectedHospital"
      @close="popupVisible = false"
    />
  </div>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue'
import EstadoPopup from '@/components/Estado_Popup.vue'
import axios from 'axios'

export default defineComponent({
  name: 'HospitalCards',
  components: { EstadoPopup },
  setup() {
    const hospitals    = ref([])
    const popupVisible = ref(false)
    const selectedHospital = ref(null)

    async function loadHospitals() {
      try {
        const { data } = await axios.get('http://localhost:5000/hospitals/')
        hospitals.value = data
      } catch (e) {
        console.error('Error cargando hospitales', e)
      }
    }

    function openPopup(hospital) {
      selectedHospital.value = hospital
      popupVisible.value     = true
    }

    onMounted(loadHospitals)

    return { hospitals, popupVisible, selectedHospital, openPopup }
  }
})
</script>

<style scoped>
/* Quicksand */
@import url('https://fonts.googleapis.com/css2?family=Quicksand:wght@300;400;500;600;700;800;900&display=swap');

:root {
  --oxford-blue: hsl(217, 54%, 11%);
  --light-oxford-blue: hsl(216, 50%, 16%);
  --indigo-dye: hsl(215, 32%, 27%);
  --blue-yonder: hsl(216, 30%, 55%);
  --white: hsl(0, 0%, 100%);
  --border-color: #68181f;
}

/* Aplica Quicksand */
.cards-wrapper,
.cards-wrapper * {
  font-family: 'Quicksand', sans-serif;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

a {
  text-decoration: none;
}

/* Layout general */
.cards-wrapper {
  background: var(--oxford-blue);
  min-height: 100vh;
  padding: 25px;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

/* Contenedor de cards */
.cards-container {
  display: flex;
  flex-wrap: wrap;
  gap: 40px;
  justify-content: center;
  max-width: 1280px;
}

/* Card con borde y sombra reducida */
.card {
  position: relative;
  display: flex;
  flex-direction: column;
  width: 300px;
  padding: 20px;
  border: 2px solid var(--border-color);
  border-radius: 15px;
  box-shadow:
    4px 10px 15px rgba(0,0,0,0.05),
    0px 20px 20px rgba(0,0,0,0.06);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.card:hover {
  transform: translateY(-4px);
  box-shadow:
    4px 15px 20px rgba(0,0,0,0.08),
    0px 25px 30px rgba(0,0,0,0.06);
}

/* Imagen fija en 200px de alto */
.card-product-img {
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  height: 200px;
}
.card-product-img img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Cuerpo de la card */
.card-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 16px 0;
  border-bottom: 1px solid var(--indigo-dye);
  margin-bottom: 12px;
}

.card-title {
  color: #121211;
  font-size: 20px;
  font-weight: 500;
  margin-bottom: 10px;
  text-align: center;
  transition: color 0.3s ease;
}
.card-title:hover {
  color: #68181f;
}

.card-text {
  color: #121211;
  font-size: 15px;
  line-height: 1.4;
  margin-bottom: 16px;
  text-align: justify;
}

/* Botón “Disponible / Lleno” */
.status-btn {
  align-self: center;
  margin-top: auto;
  padding: 12px 36px;
  color: #ffffff;
  font-weight: 600;
  font-size: 14px;
  border: none;
  border-radius: 40px;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  cursor: pointer;
  transition: transform 0.3s;
}
.status-btn:hover {
  transform: translateY(-2px);
}

/* Botón pequeño inferior derecho */
.view-btn {
  position: absolute;
  right: 12px;
  bottom: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  background-color: #ac3030;
  border: none;
  border-radius: 50%;
  box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  cursor: pointer;
  transition: background-color 0.3s, transform 0.3s;
}
.view-btn:hover {
  background-color: #efdbbf;
  transform: translateY(-2px);
}

.dots {
  color: #121211;
  font-size: 18px;
  line-height: 1;
}
</style>
