<template>
  <section class="chart-section">
    <h2>Hourly Temperature and Rainfall</h2>

    <canvas ref="weatherChart"></canvas>
  </section>
</template>

<script>
import Chart from 'chart.js/auto'

export default {
  props: {
    hourlyData: {
      type: Array,
      required: true
    }
  },
  mounted() {
    this.createChart()
  },
  watch: {
    hourlyData() {
      this.createChart()
    }
  },
  methods: {
    createChart() {
      if (!this.hourlyData.length) return

      if (this.chart) {
        this.chart.destroy()
      }

      const labels = this.hourlyData.map(hour => hour.time.split(' ')[1])
      const temperatures = this.hourlyData.map(hour => hour.temp_f)
      const rainfall = this.hourlyData.map(hour => hour.precip_in)

      this.chart = new Chart(this.$refs.weatherChart, {
        type: 'line',
        data: {
          labels,
          datasets: [
            {
              label: 'Temperature °F',
              data: temperatures
            },
            {
              label: 'Rainfall inches',
              data: rainfall
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false
        }
      })
    }
  }
}
</script>

<style scoped>
.chart-section {
  margin-top: 30px;
  height: 400px;
  max-width: 900px;
}
</style>