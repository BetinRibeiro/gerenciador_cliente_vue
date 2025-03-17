<template>
  <div>
    <canvas id="graficoMensalidade"></canvas>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';

export default {
  name: 'GraficoMensalidade',  // Atualize o nome aqui
  mounted() {
    Chart.register(...registerables);
    this.atualizaGrafico();
  },
  methods: {
    atualizaGrafico() {
      const ctx = document.getElementById('graficoMensalidade').getContext('2d');
      if (window.grafico) {
        window.grafico.destroy();
      }
      const mensalidades = [
        { nome: 'Janeiro', valor: 150 },
        { nome: 'Fevereiro', valor: 180 },
        { nome: 'Março', valor: 200 }
      ];
      const coresPrimarias = ["#5cb85c", "#66c266", "#4cae4c"];
      let backgroundColors = mensalidades.map((_, index) => coresPrimarias[index % coresPrimarias.length]);
      const data = {
        labels: mensalidades.map(m => m.nome),
        datasets: [{
          label: 'Valor da Mensalidade (R$)',
          data: mensalidades.map(m => m.valor),
          backgroundColor: backgroundColors,
          borderColor: backgroundColors,
          borderWidth: 1
        }]
      };
      window.grafico = new Chart(ctx, {
        type: 'bar',
        data: data,
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          },
          responsive: true,
          plugins: {
            legend: {
              position: 'top',
            }
          }
        }
      });
    }
  }
}
</script>

<style scoped>
canvas {
  max-width: 600px;
  margin: 20px auto;
}
</style>
