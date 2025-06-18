<template>
  <div>
    <NavbarGeneral :mode="'user'" />

    <div class="main-container">
      <div class="content-box">
        <!-- Sección izquierda -->
        <div class="left-section">
          <h1 class="title">Bienvenido a FlowMedic</h1>
          <p class="subtitle">Gestiona y visualiza tus simulaciones hospitalarias con un solo clic.</p>
          <button @click="goToSimulacion" class="sim-button">
            <font-awesome-icon :icon="['fas', 'play']" class="mr-2" />
            ­­­ㅤGenerar Simulación
          </button>

          <h2 class="section-title mt-8">Archivos Generados</h2>
          <div v-if="files.length > 0" class="table-container">
            <table class="styled-table">
              <thead>
                <tr>
                  <th>Archivo JSON</th>
                  <th>Ver JSON</th>
                  <th>Descargar CSV</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="file in files" :key="file">
                  <td>{{ file }}</td>
                  <td>
                    <a
                      :href="`http://127.0.0.1:5000/app/output/simulations/${file}`"
                      target="_blank"
                      class="text-blue-600 hover:underline"
                    >
                      Ver JSON
                    </a>
                  </td>
                  <td class="space-x-2">
                    <button @click="descargarCSVDesdeJSON(file)" class="btn-blue">Descargar CSV</button>
                    <button @click="previsualizarCSV(file)" class="btn-gray">Previsualizar</button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <div v-else class="text-gray-500 mt-2">
            No se encontraron archivos generados para tu hospital.
          </div>
        </div>

        <!-- Sección derecha: animación -->
        <div class="right-section">
          <iframe
            src="https://lottie.host/embed/5647ea39-8d31-4dc4-9d4b-680e208173a7/gbocCk5ABf.lottie"
            style="width: 100%; height: 100%; border: none"
            allowfullscreen
          ></iframe>
        </div>
      </div>

      <!-- Modal de previsualización -->
      <div v-if="mostrarPreview" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
        <div class="bg-white rounded-lg shadow-lg p-6 w-11/12 max-w-5xl overflow-auto max-h-[80vh]">
          <div class="flex justify-between items-center mb-4">
            <h3 class="text-xl font-bold">Vista previa del CSV</h3>
            <button @click="cerrarPreview" class="text-red-500 font-bold text-lg">✖</button>
          </div>

          <table class="preview-table">
            <thead>
              <tr>
                <th
                  v-for="col in columnasPreview"
                  :key="col"
                >
                  {{ columnLabels[col] || col }}
                </th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(fila, index) in filasPreview" :key="index">
                <td v-for="col in columnasPreview" :key="col">
                  {{ fila[col] }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>

    <FooterComponent />
  </div>
</template>

<script>
import axios from 'axios'
import NavbarGeneral from './NavbarGeneral.vue'
import FooterComponent from './Footer_Component.vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { faPlay } from '@fortawesome/free-solid-svg-icons'
import { library } from '@fortawesome/fontawesome-svg-core'
library.add(faPlay)

export default {
  name: 'MenuUsuario',
  components: { NavbarGeneral, FontAwesomeIcon, FooterComponent },
  data() {
    return {
      files: [],
      mostrarPreview: false,
      columnasPreview: [],
      filasPreview: [],
      // Mapeo de columnas técnicas a etiquetas en español
      columnLabels: {
        patient_id:     'ID Paciente',
        priority:       'Prioridad',
        arrival_time:   'Hora de llegada',
        triage_wait:    'Espera triaje',
        triage_end:     'Fin triaje',
        consult_wait:   'Espera consulta',
        consult_end:    'Fin consulta',
        diag_wait:      'Espera diagnóstico',
        diag_end:       'Fin diagnóstico',
        treat_wait:     'Espera tratamiento',
        treat_end:      'Fin tratamiento'
      }
    }
  },
  methods: {
    async fetchFiles() {
      const hospitalId = localStorage.getItem('hospital_id')
      if (!hospitalId) return
      try {
        const response = await axios.get(`http://127.0.0.1:5000/queue/files/${hospitalId}`)
        this.files = response.data.files || []
      } catch (e) {
        console.error('Error:', e)
      }
    },
    goToSimulacion() {
      this.$router.push('/urgencias')
    },
    async descargarCSVDesdeJSON(nombreArchivoJson) {
      try {
        const response = await axios.get(`http://127.0.0.1:5000/app/output/simulations/${nombreArchivoJson}`)
        const registros = response.data.records
        const keys = Object.keys(registros[0])
        const csvRows = [keys.join(','), ...registros.map(obj => keys.map(k => JSON.stringify(obj[k])).join(','))]
        const blob = new Blob([csvRows.join('\n')], { type: 'text/csv' })
        const url = URL.createObjectURL(blob)
        const link = document.createElement('a')
        link.href = url
        link.setAttribute('download', nombreArchivoJson.replace('.json', '.csv'))
        document.body.appendChild(link)
        link.click()
        document.body.removeChild(link)
        URL.revokeObjectURL(url)
      } catch {
        alert('Error al convertir el JSON.')
      }
    },
    async previsualizarCSV(nombreArchivoJson) {
      try {
        const response = await axios.get(`http://127.0.0.1:5000/app/output/simulations/${nombreArchivoJson}`)
        const registros = response.data.records
        if (!registros || registros.length === 0) {
          alert('No hay datos para mostrar en la vista previa.')
          return
        }
        this.columnasPreview = Object.keys(registros[0])
        this.filasPreview    = registros
        this.mostrarPreview  = true
      } catch {
        alert('Error al mostrar la vista previa.')
      }
    },
    cerrarPreview() {
      this.mostrarPreview   = false
      this.columnasPreview  = []
      this.filasPreview     = []
    }
  },
  mounted() {
    this.fetchFiles()
  }
}
</script>

<style scoped>
.main-container {
  background-color: #f9fafb;
  padding: 40px 20px;
  min-height: 100vh;
  font-family: 'Segoe UI', sans-serif;
}

.content-box {
  display: flex;
  flex-direction: row;
  background-color: #fff;
  border-radius: 20px;
  box-shadow: 0px 8px 30px rgba(0, 0, 0, 0.05);
  padding: 40px;
  gap: 20px;
  max-width: 960px;
  margin: 0 auto;
}

.left-section {
  flex: 1;
  max-width: 55%;
}

.right-section {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  max-width: 45%;
}

.title {
  font-size: 32px;
  font-weight: bold;
  color: #1f2937;
}

.subtitle {
  margin: 10px 0 20px;
  color: #4b5563;
}

.sim-button {
  background-color: #ac3030;
  color: white;
  font-size: 16px;
  padding: 12px 20px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  transition: background-color 0.3s ease;
}
.sim-button:hover {
  background-color: #efdbbf;
  color: #121211;
}

.section-title {
  font-size: 20px;
  font-weight: 600;
  color: #1f2937;
  margin-top: 30px;
  margin-bottom: 10px;
}

.table-container {
  overflow-x: auto;
}

.styled-table {
  width: 100%;
  border-collapse: collapse;
}
.styled-table th,
.styled-table td {
  padding: 10px;
  border: 1px solid #e5e7eb;
}
.styled-table th {
  background-color: #f3f4f6;
  text-align: left;
}

.btn-blue {
  background-color: #ac3030;
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 14px;
}
.btn-blue:hover {
  background-color: #68181f;
}

.btn-gray {
  background-color: #6b7280;
  color: white;
  padding: 5px 10px;
  border-radius: 5px;
  font-size: 14px;
}
.btn-gray:hover {
  background-color: #4b5563;
}

/* Excel-like preview table */
.preview-table {
  width: 100%;
  border-collapse: collapse;
  display: block;
  overflow-x: auto;
  background-color: #ffecec;
}
.preview-table th,
.preview-table td {
  border: 1px solid #999;
  padding: 8px;
  text-align: left;
  background-color: #ffecec;
}
.preview-table th {
  background-color: #ffdede;
  font-weight: bold;
}
.preview-table tr:hover td {
  background-color: #ffdcdc;
}
</style>
