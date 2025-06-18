<template>
  <div class="results-charts">
    <h3>Histogramas de LOS por prioridad</h3>
    <canvas ref="losChart"></canvas>

    <h3>Tiempos medios de espera</h3>
    <canvas ref="waitChart"></canvas>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import {
  Chart,
  BarController,
  BarElement,
  CategoryScale,
  LinearScale,
  Title,
  Tooltip,
  Legend
} from 'chart.js'

// Registrar los componentes de Chart.js
Chart.register(
  BarController,
  BarElement,
  CategoryScale,
  LinearScale,
  Title,
  Tooltip,
  Legend
)

export default {
  name: 'ResultsCharts',

  // Definición explícita de props
  props: {
    records: {
      type: Array,
      required: true
    },
    metrics: {
      type: Object,
      required: true
    }
  },

  setup(props) {
    const losChart = ref(null)
    const waitChart = ref(null)

    onMounted(() => {
      // Parámetros del histograma
      const binSize = 50
      const priorities = [
        { label: 'Critical',    color: 'rgba(255, 99, 132, 0.6)' },
        { label: 'Urgent',      color: 'rgba(54, 162, 235, 0.6)' },
        { label: 'Non-urgent',  color: 'rgba(75, 192, 192, 0.6)' }
      ]

      // Construir datasets para cada prioridad
      const datasetsLOS = priorities.map((p, idx) => {
        const values = props.records
          .filter(r => r.priority === idx)
          .map(r => r.los)

        const maxVal = values.length ? Math.max(...values) : 0
        const binCount = Math.ceil((maxVal + 1) / binSize)
        const counts = Array(binCount).fill(0)

        values.forEach(v => {
          const binIndex = Math.min(binCount - 1, Math.floor(v / binSize))
          counts[binIndex]++
        })

        return {
          label: p.label,
          data: counts,
          backgroundColor: p.color,
          borderWidth: 1
        }
      })

      const labelsLOS = datasetsLOS[0].data.map((_, i) => {
        const min = i * binSize
        const max = (i + 1) * binSize
        return `${min}-${max}`
      })

      // Gráfico de LOS
      new Chart(losChart.value, {
        type: 'bar',
        data: {
          labels: labelsLOS,
          datasets: datasetsLOS
        },
        options: {
          responsive: true,
          plugins: {
            legend: { position: 'top' },
            tooltip: { mode: 'index', intersect: false }
          },
          scales: {
            x: {
              title: {
                display: true,
                text: 'Intervalo de LOS (min)'
              }
            },
            y: {
              title: {
                display: true,
                text: 'Frecuencia'
              },
              beginAtZero: true
            }
          }
        }
      })

      // Gráfico de tiempos medios
      const waitLabels = ['Triage', 'Consulta', 'Diagnóstico', 'Tratamiento']
      const waitData = [
        props.metrics.avg_wait_triage,
        props.metrics.avg_wait_consult,
        props.metrics.avg_wait_diag,
        props.metrics.avg_wait_treat
      ]

      new Chart(waitChart.value, {
        type: 'bar',
        data: {
          labels: waitLabels,
          datasets: [{
            label: 'Tiempo medio (min)',
            data: waitData,
            backgroundColor: [
              'rgba(255, 159, 64, 0.6)',
              'rgba(153, 102, 255, 0.6)',
              'rgba(255, 205, 86, 0.6)',
              'rgba(75, 192, 192, 0.6)'
            ],
            borderWidth: 1
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { display: false }
          },
          scales: {
            y: {
              beginAtZero: true,
              title: {
                display: true,
                text: 'Minutos'
              }
            }
          }
        }
      })
    })

    return {
      losChart,
      waitChart
    }
  }
}
</script>

<style scoped>
.results-charts {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  margin-top: 1.5rem;
}
canvas {
  width: 100%;
  max-height: 300px;
}
</style>
