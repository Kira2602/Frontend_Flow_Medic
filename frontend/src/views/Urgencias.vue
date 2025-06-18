<template>
  <div>
    <NavbarComponent />

    <div class="form-container">
      <div class="form-card">
        <h2 class="form-title">Simulación de Urgencias</h2>

        <form @submit.prevent="runSimulation" class="form-body">
          <div
            class="form-group"
            v-for="(key) in Object.keys(form).filter(k => k !== 'hospital_id')"
            :key="key"
            
          >
            <label class="form-label">
              {{ labels[key] || key }}
            </label>
            <div class="form-input-wrapper">
              <i class="fas fa-user input-icon"></i>
              <input
                v-if="typeof value === 'number'"
                type="number"
                step="any"
                v-model.number="form[key]"
                class="form-input"
              />
              <input
                v-else
                type="text"
                v-model="form[key]"
                class="form-input"
              />
            </div>
          </div>

          <button type="submit" class="submit-button">
            Ejecutar Simulación
          </button>
        </form>
      </div>

      <!-- Resultados -->
      <div v-if="result" ref="resultSection" class="result-box">
        <h3 class="result-title">Resultados</h3>
        <p><strong>Archivo:</strong> {{ result.file_name }}</p>
        <p><strong>ID de simulacion:</strong> {{ result.queue_id }}</p>
        <h4>Métricas</h4>
        <pre class="result-metrics">{{ result.metrics }}</pre>
        <div class="metrics-interpretation">
          <h4>Interpretación de Métricas</h4>
          <ul class="list-disc ml-6 mt-2 text-sm text-gray-700">
            <li>
              <strong>Tasa de admisión (admission_rate):</strong>
              {{ (result.metrics.admission_rate * 100).toFixed(2) }}% de los pacientes fueron hospitalizados tras su atención inicial. Una tasa superior al 10% puede indicar una alta severidad de los casos.
            </li>
            <li>
              <strong>Estancia promedio en el sistema (avg_los):</strong>
              {{ result.metrics.avg_los.toFixed(2) }} minutos en promedio desde que el paciente entra al sistema hasta que se va. Valores altos pueden reflejar congestión o procesos lentos.
            </li>
            <li>
              <strong>Tiempo promedio de espera en triaje (avg_wait_triage):</strong>
              {{ result.metrics.avg_wait_triage.toFixed(2) }} minutos de espera antes de la clasificación inicial del paciente. Debe mantenerse bajo para garantizar atención oportuna.
            </li>
            <li>
              <strong>Tiempo promedio de espera en consulta (avg_wait_consult):</strong>
              {{ result.metrics.avg_wait_consult.toFixed(2) }} minutos de espera antes de ver a un médico. Tiempos altos indican una carga excesiva en el personal médico.
            </li>
            <li>
              <strong>Tiempo promedio de espera en diagnóstico (avg_wait_diag):</strong>
              {{ result.metrics.avg_wait_diag.toFixed(2) }} minutos esperando resultados de laboratorio o imágenes. Puede reflejar saturación en rayos X, laboratorio o ultrasonido.
            </li>
            <li>
              <strong>Tiempo promedio de espera en tratamiento (avg_wait_treat):</strong>
              {{ result.metrics.avg_wait_treat.toFixed(2) }} minutos de espera para recibir tratamiento. Un valor elevado puede deberse a limitaciones de camas o personal de enfermería.
            </li>
            <li>
              <strong>Pacientes atendidos por hora (throughput):</strong>
              {{ result.metrics.throughput.toFixed(2) }} pacientes completan todo el proceso por hora. Un valor alto indica buena capacidad de resolución del sistema hospitalario.
            </li>
          </ul>
        </div>

         <div class="flex justify-center mb-6">
        <button
          @click="goToDatos"
          class="submit-button"        
          >
          Ver datos de simulación
        </button>
      </div>
      </div>
      <!-- Nueva sección de gráficas -->
    <ResultsCharts
      v-if="records.length"
      :records="records"
      :metrics="metrics"
    />

      <div v-if="error" class="error-message">
        {{ error }}
      </div>
      <!-- Botón para ir a generar simulación -->
      
    </div>
     <FooterComponent />
  </div>
</template>

<script>
import { defineComponent, reactive, ref ,nextTick } from 'vue'
import axios from 'axios'
import NavbarComponent from '@/components/NavbarGeneral.vue'
import FooterComponent from '@/components/Footer_Component.vue'
import ResultsCharts from '@/components/ResultsCharts.vue'

export default defineComponent({
  name: 'UrgenciasPage',
  components: { NavbarComponent, FooterComponent,ResultsCharts  },
  
  setup() {
    const form = reactive({
      peak_lambda: 15,
      night_lambda: 12,
      day_lambda: 10,
      triage_mean: 5,
      triage_std: 1,
      consult_mean: 10,
      consult_std: 2,
      diag_xray: 20,
      diag_lab: 30,
      treat_mean: 10,
      treat_std: 2,
      doctors_day: 5,
      doctors_night: 4,
      nurses_day: 9,
      nurses_night: 7,
      beds: 15,
      xray: 2,
      ultrasound: 1,
      priority_critical: 0.2,
      priority_urgent: 0.3,
      priority_non_urgent: 0.5,
      diag_prob: 0.5,
      admit_prob: 0.1,
      num_runs: 500,
      hospital_id: Number(localStorage.getItem('hospital_id')) || 1
    })
      
    const labels = {
      peak_lambda: 'Llegadas pico',
      night_lambda: 'Llegadas noche',
      day_lambda: 'Llegadas día',
      triage_mean: 'Promedio de triaje',
      triage_std: 'Desviación triaje',
      consult_mean: 'Promedio de consulta',
      consult_std: 'Desviación consulta',
      diag_xray: 'Tiempo rayos X',
      diag_lab: 'Tiempo laboratorio',
      treat_mean: 'Promedio tratamiento',
      treat_std: 'Desviación tratamiento',
      doctors_day: 'Doctores día',
      doctors_night: 'Doctores noche',
      nurses_day: 'Enfermeras día',
      nurses_night: 'Enfermeras noche',
      beds: 'Camas',
      xray: 'Máquinas rayos X',
      ultrasound: 'Ultrasonidos',
      priority_critical: 'Prioridad crítica (%)',
      priority_urgent: 'Prioridad urgente (%)',
      priority_non_urgent: 'Prioridad no urgente (%)',
      diag_prob: 'Probabilidad de diagnóstico',
      admit_prob: 'Probabilidad de admisión',
      num_runs: 'N.º de simulaciones',
      hospital_id: 'ID del hospital'
    }

    const result = ref(null)
    const error = ref(null)
    const resultSection = ref(null)

    const records = ref([])  
    const metrics = ref({})

    async function runSimulation() {
      error.value = null

      try {
        const resp = await axios.post('http://localhost:5000/simulate/', form)
        result.value = resp.data
        const data = resp.data
        result.value
        records.value = data.records
        metrics.value = data.metrics
        await nextTick()
        resultSection.value?.scrollIntoView({ behavior: 'smooth' })
      } catch (e) {
        error.value = e.response?.data?.error || e.message
      }
    }

    return { form, result, error, runSimulation, labels, resultSection, records, metrics }
  },
  methods: {
    goToDatos() {
      this.$router.push('/menu-usuario')
    }
  }
  
})
</script>

<style scoped>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css');

.metrics-interpretation {
  background-color: #f9fafb;
  border-left: 4px solid #10b981;
  padding: 1rem;
  margin-top: 1rem;
  border-radius: 0.375rem;
}

.form-container {
  padding: 40px 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  font-family: Arial, sans-serif;
}

.form-card {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  padding: 30px;
  width: 100%;
  max-width: 480px;
}

.form-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
  text-align: center;
}

.form-body {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.form-label {
  margin-bottom: 6px;
  font-weight: 500;
  color: #333;
  text-align: center;
  text-transform: none;
}

.form-input-wrapper {
  position: relative;
  display: flex;
  justify-content: center;
}

.input-icon {
  position: absolute;
  top: 50%;
  left: 12px;
  transform: translateY(-50%);
  color: #aaa;
  font-size: 14px;
}

.form-input {
  width: 250px;
  padding: 10px 12px 10px 36px;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-size: 14px;
  transition: border 0.3s;
}

.form-input:focus {
  border-color: #3498db;
  outline: none;
}

.submit-button {
  width: 250px;
  margin: 0 auto;
  background-color: #efdbb7;
  color: #121211;
  font-weight: bold;
  border: none;
  border-radius: 8px;
  padding: 12px;
  border-width: 2px;
  border-style: solid;
  cursor: pointer;
  font-size: 15px;
  transition: background-color 0.3s;
}

.submit-button:hover {
  background-color: #ac3030;
  color: white;
}

/* Resultado y errores */
.result-box {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  margin-top: 30px;
  padding: 20px;
  border-radius: 10px;
  max-width: 480px;
  width: 100%;
}

.result-title {
  font-weight: bold;
  margin-bottom: 10px;
}

.result-metrics {
  background: white;
  padding: 10px;
  border-radius: 6px;
  overflow-x: auto;
}

.error-message {
  margin-top: 20px;
  color: #68181f;
  font-weight: bold;
}
</style>
