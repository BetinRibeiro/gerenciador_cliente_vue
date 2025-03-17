<template>
  <!-- Contêiner fluido com espaçamento interno para melhor apresentação -->
  <div class="container-fluid p-3">
    <!-- Caixa escura arredondada para o conteúdo -->
    <div class="bg-dark rounded p-4">
      <div class="row">
        
        <!-- Coluna esquerda: Formulário e Gráfico -->
        <div class="col-md-6 col-12">
          <h5>Cadastro de Mensalidade</h5>
          <!-- Formulário de cadastro de mensalidade com validação -->
          <form @submit.prevent="adicionarMensalidade">
            <div class="mb-3">
              <label for="nomeCliente" class="form-label">Nome do Cliente</label>
              <input 
                type="text" 
                v-model="novoCliente.nome" 
                class="form-control" 
                id="nomeCliente" 
                required 
                pattern="^[a-zA-ZÀ-ÿ'\s]+$" 
                title="O nome deve conter apenas letras e espaços">
            </div>
            <div class="mb-3">
              <label for="valorMensalidade" class="form-label">Valor da Mensalidade</label>
              <input 
                type="number" 
                v-model="novoCliente.valor" 
                class="form-control" 
                id="valorMensalidade" 
                required 
                min="1" 
                step="0.01" 
                title="O valor deve ser maior que zero">
            </div>
            <button type="submit" class="btn btn-primary">Adicionar</button>
          </form>
          <hr>
          <!-- Área para exibir o gráfico de mensalidades -->
          <canvas id="graficoMensalidade"></canvas>
        </div>

        <!-- Coluna direita: Tabela de mensalidades -->
        <div class="col-md-6 col-12">
          <h5>Clientes e Mensalidades</h5>
          <div class="table-responsive text-center">
            <table class="table text-start align-middle table-sm table-bordered table-hover mb-0">
              <thead>
                <tr>
                  <th>Ação</th> <!-- Coluna de ação para deletar -->
                  <th>Nome</th>
                  <th>Valor da Mensalidade (R$)</th>
                  <th>Percentual (%)</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(mensalidade, index) in mensalidades" :key="index">
                  <td>
                    <button @click="removerMensalidade(index)" class="btn btn-danger btn-sm">
                      <i class="fa fa-trash" aria-hidden="true"></i>
                    </button>
                  </td>
                  <!-- Nome e valor editáveis -->
                  <td contenteditable="true" @blur="salvaAlteracoes($event, index, 'nome')">{{ mensalidade.nome }}</td>
                  <td contenteditable="true" @blur="salvaAlteracoes($event, index, 'valor')">{{ mensalidade.valor.toFixed(2) }}</td>
                  <td>{{ calculaPercentual(mensalidade.valor).toFixed(1) }}%</td>
                </tr>
              </tbody>
              <tfoot>
                <tr>
                  <th>#</th>
                  <th>Total</th>
                  <td>{{ totalMensalidades.toFixed(2) }}</td>
                  <th>100 (%)</th>
                </tr>
              </tfoot>
            </table>
          </div>
        </div>
      </div>
    </div>
    <!-- Botão para imprimir a página -->
    <button @click="imprimirPagina">Imprimir Página</button>
  </div>
</template>

<script>

import { Chart, registerables } from 'chart.js';

// Registrar todos os componentes necessários do Chart.js para garantir que os gráficos funcionem corretamente
Chart.register(...registerables);

export default {
  name: 'App', // Define o nome do componente

  data() {
    return {
      // Objeto que armazena os dados do novo cliente a ser cadastrado
      novoCliente: {
        nome: '',  // Nome do cliente (inicialmente vazio)
        valor: 0    // Valor da mensalidade (inicialmente zero)
      },
      // Lista de mensalidades, recuperada do localStorage se existir, ou inicializada como um array vazio
      mensalidades: JSON.parse(localStorage.getItem('mensalidades')) || [],
      grafico: null // Variável para armazenar a instância do gráfico
    };
  },

  computed: {
    // Calcula o total de todas as mensalidades cadastradas
    totalMensalidades() {
      return this.mensalidades.reduce((acc, mensalidade) => acc + mensalidade.valor, 0);
    }
  },

  methods: {
    // Adiciona uma nova mensalidade à lista
    adicionarMensalidade() {
      this.mensalidades.push({ ...this.novoCliente }); // Adiciona uma cópia do objeto novoCliente à lista
      this.salvarLocalStorage(); // Salva os dados no localStorage para persistência
      this.novoCliente = { nome: '', valor: 0 }; // Reseta os campos do formulário após a adição
      this.atualizarGrafico(); // Atualiza o gráfico para refletir os novos dados
    },

    // Remove uma mensalidade da lista com base no índice fornecido
    removerMensalidade(index) {
      this.mensalidades.splice(index, 1); // Remove o item da lista pelo índice
      this.salvarLocalStorage(); // Atualiza os dados no localStorage
      this.atualizarGrafico(); // Atualiza o gráfico para refletir a remoção
    },

    // Salva os dados da lista de mensalidades no localStorage para persistência
    salvarLocalStorage() {
      localStorage.setItem('mensalidades', JSON.stringify(this.mensalidades));
    },

    // Atualiza os valores diretamente na tabela quando o usuário edita um campo
    salvaAlteracoes(event, index, key) {
      const valorAtualizado = event.target.textContent; // Obtém o valor editado pelo usuário

      if (key === 'valor') {
        this.mensalidades[index][key] = parseFloat(valorAtualizado) || 0; // Converte para número e evita NaN
      } else {
        this.mensalidades[index][key] = valorAtualizado; // Atualiza o nome normalmente
      }

      this.salvarLocalStorage(); // Salva a alteração no localStorage
      this.atualizarGrafico(); // Atualiza o gráfico para refletir as alterações
    },

    // Calcula o percentual que uma mensalidade representa em relação ao total
    calculaPercentual(valor) {
      return (valor / this.totalMensalidades) * 100 || 0; // Evita divisões por zero retornando 0 quando necessário
    },

    // Atualiza o gráfico de mensalidades com os dados atuais
    atualizarGrafico() {
      const ctx = document.getElementById('graficoMensalidade').getContext('2d'); // Obtém o contexto do canvas para o gráfico

      if (this.grafico) this.grafico.destroy(); // Destroi o gráfico existente antes de recriar um novo

      // Define cores diferentes para cada barra do gráfico
      const coresPrimarias = [
        "#5cb85c", "#66c266", "#4cae4c", "#52be52", "#449d44", "#73c673", "#3f9f3f"
      ];
      const backgroundColors = this.mensalidades.map((_, index) => coresPrimarias[index % coresPrimarias.length]);

      // Estrutura dos dados para o gráfico
      const data = {
        labels: this.mensalidades.map(m => m.nome), // Nomes dos clientes como rótulos
        datasets: [{
          label: 'Valor da Mensalidade (R$)', // Nome do conjunto de dados
          data: this.mensalidades.map(m => m.valor), // Valores das mensalidades
          backgroundColor: backgroundColors, // Cores de fundo para cada barra
          borderColor: backgroundColors, // Cor da borda das barras
          borderWidth: 1 // Define a espessura da borda das barras
        }]
      };

      // Criação de um novo gráfico do tipo barra
      this.grafico = new Chart(ctx, {
        type: 'bar',
        data: data,
        options: {
          scales: {
            y: {
              beginAtZero: true // Define o eixo Y para começar no zero
            }
          },
          responsive: true, // Torna o gráfico responsivo para diferentes tamanhos de tela
          plugins: {
            legend: {
              position: 'top', // Define a posição da legenda no topo do gráfico
            }
          }
        }
      });
    },

    // Aciona a impressão da página pelo navegador
    imprimirPagina() {
      window.print();
    }
  },

  // Hook do ciclo de vida: Executa ao montar o componente
  mounted() {
    this.atualizarGrafico(); // Garante que o gráfico seja gerado ao carregar a página
  }
};
</script>


<style scoped>
canvas {
  max-width: 600px;
  margin: 20px auto;
}
#graficoMensalidade {
  width: 100vw;  /* 100% da largura da tela */
  height: 100vh; /* 100% da altura da tela */
}
</style>
