<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Controle Financeiro Pessoal</title>
<style>
  body {
    font-family: 'Poppins', sans-serif;
    background: #f5f6fa;
    margin: 0;
    padding: 0;
  }

  header {
    background: #4CAF50;
    color: white;
    padding: 15px;
    text-align: center;
    font-size: 1.5em;
  }

  .container {
    max-width: 400px;
    margin: 20px auto;
    background: white;
    border-radius: 15px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    padding: 20px;
  }

  .saldo {
    text-align: center;
    margin-bottom: 20px;
  }

  .saldo h2 {
    margin: 0;
    font-size: 1.2em;
  }

  .saldo span {
    font-size: 2em;
    font-weight: bold;
    color: #333;
  }

  form {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  input, select, button {
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 10px;
    font-size: 1em;
  }

  button {
    background: #4CAF50;
    color: white;
    border: none;
    font-weight: bold;
    cursor: pointer;
    transition: 0.3s;
  }

  button:hover {
    background: #43a047;
  }

  .lista-transacoes {
    margin-top: 20px;
  }

  .transacao {
    display: flex;
    justify-content: space-between;
    background: #f1f1f1;
    margin-bottom: 8px;
    padding: 10px;
    border-radius: 10px;
  }

  .transacao.receita {
    border-left: 5px solid #4CAF50;
  }

  .transacao.despesa {
    border-left: 5px solid #f44336;
  }

  .transacao button {
    background: none;
    border: none;
    color: #777;
    cursor: pointer;
    font-size: 1em;
  }

  .grafico {
    margin-top: 25px;
    text-align: center;
  }

  canvas {
    max-width: 100%;
  }
</style>
</head>
<body>

<header>💸 Controle Financeiro</header>

<div class="container">
  <div class="saldo">
    <h2>Saldo Atual</h2>
    <span id="saldo">R$ 0,00</span>
  </div>

  <form id="form">
    <input type="text" id="descricao" placeholder="Descrição" required />
    <input type="number" id="valor" placeholder="Valor" required />
    <select id="tipo">
      <option value="receita">Receita</option>
      <option value="despesa">Despesa</option>
    </select>
    <button type="submit">Adicionar</button>
  </form>

  <div class="lista-transacoes" id="transacoes"></div>

  <div class="grafico">
    <canvas id="graficoFinanceiro"></canvas>
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
  const form = document.getElementById('form');
  const transacoesEl = document.getElementById('transacoes');
  const saldoEl = document.getElementById('saldo');
  const ctx = document.getElementById('graficoFinanceiro');

  let transacoes = JSON.parse(localStorage.getItem('transacoes')) || [];

  function atualizarInterface() {
    transacoesEl.innerHTML = '';
    let saldo = 0, receitas = 0, despesas = 0;

    transacoes.forEach((t, i) => {
      const div = document.createElement('div');
      div.classList.add('transacao', t.tipo);
      div.innerHTML = `
        <span>${t.descricao}</span>
        <span>${t.tipo === 'despesa' ? '-' : '+'} R$ ${t.valor.toFixed(2)}</span>
        <button onclick="remover(${i})">✖</button>
      `;
      transacoesEl.appendChild(div);

      if (t.tipo === 'receita') receitas += t.valor;
      else despesas += t.valor;
    });

    saldo = receitas - despesas;
    saldoEl.textContent = `R$ ${saldo.toFixed(2)}`;
    localStorage.setItem('transacoes', JSON.stringify(transacoes));

    atualizarGrafico(receitas, despesas);
  }

  function remover(index) {
    transacoes.splice(index, 1);
    atualizarInterface();
  }

  form.addEventListener('submit', e => {
    e.preventDefault();
    const descricao = document.getElementById('descricao').value;
    const valor = parseFloat(document.getElementById('valor').value);
    const tipo = document.getElementById('tipo').value;

    if (descricao && valor) {
      transacoes.push({ descricao, valor, tipo });
      form.reset();
      atualizarInterface();
    }
  });

  let grafico;
  function atualizarGrafico(receitas, despesas) {
    if (grafico) grafico.destroy();
    grafico = new Chart(ctx, {
      type: 'doughnut',
      data: {
        labels: ['Receitas', 'Despesas'],
        datasets: [{
          data: [receitas, despesas],
          backgroundColor: ['#4CAF50', '#f44336'],
          borderWidth: 0
        }]
      },
      options: {
        plugins: {
          legend: { position: 'bottom' }
        },
        animation: { animateScale: true }
      }
    });
  }

  atualizarInterface();
</script>

</body>
</html>
