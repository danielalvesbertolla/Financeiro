# Financeiro
metas-financeiras

<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX</title>
<style>
body { font-family: Arial; background:#0f172a; color:white; padding:20px; }
.card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; }
button { padding:8px 12px; border:none; border-radius:8px; cursor:pointer; margin:5px; }
input { padding:8px; margin:5px; border-radius:8px; border:none; }
.green { background:#22c55e; }
.red { background:#ef4444; }
.blue { background:#3b82f6; }
.alert { margin-top:10px; font-weight:bold; }
</style>
</head>
<body>

<h1>📊 Metas Financeiras PRO MAX</h1>

<div id="home" class="card">
  <h3>Suas Metas</h3>
  <div id="listaMetas"></div>

  <h4>Criar Nova Meta</h4>
  Nome: <input id="nomeMeta">
  Valor: <input id="valorMeta" type="number">
  Dias: <input id="diasMeta" type="number">
  <button class="green" onclick="criarMeta()">Criar</button>
</div>

<div id="metaPage" style="display:none;">
  <div class="card">
    <button onclick="voltar()">⬅ Voltar</button>
    <button class="red" onclick="resetarMeta()">Resetar</button>
    <button class="red" onclick="excluirMeta()">Excluir</button>

    <h2 id="tituloMeta"></h2>
    <p id="restante"></p>
    <p id="diaria"></p>
    <p id="percentual"></p>

    <input id="ganho" type="number">
    <button class="green" onclick="registrar()">Registrar ganho</button>

    <p class="alert" id="alerta"></p>
  </div>

  <div class="card">
    <h3>Distribuição para Contas</h3>
    Nome: <input id="nomeDespesa">
    Valor: <input id="valorDespesa" type="number">
    <button class="blue" onclick="addDespesa()">Adicionar</button>
    <div id="listaDespesas"></div>
  </div>

  <div class="card">
    <h3>Histórico</h3>
    <div id="historico"></div>
  </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;

function salvar(){
  localStorage.setItem('metas', JSON.stringify(metas));
}

function renderMetas(){
  let html='';
  metas.forEach((m,i)=>{
    html += `<div>
      ${m.nome} - R$ ${m.meta}
      <button class='blue' onclick='abrirMeta(${i})'>Abrir</button>
    </div>`;
  });
  document.getElementById('listaMetas').innerHTML = html;
}

function criarMeta(){
  let nome = document.getElementById('nomeMeta').value;
  let valor = Number(document.getElementById('valorMeta').value);
  let dias = Number(document.getElementById('diasMeta').value);

  metas.push({
    nome,
    meta:valor,
    dias,
    restante:valor,
    historico:[],
    despesas:[
      {nome:'Aluguel', valor:1400},
      {nome:'Internet', valor:100},
      {nome:'Cartão', valor:2500}
    ]
  });

  salvar();
  renderMetas();
}

function abrirMeta(i){
  metaAtual = i;
  document.getElementById('home').style.display='none';
  document.getElementById('metaPage').style.display='block';
  atualizar();
}

function voltar(){
  document.getElementById('home').style.display='block';
  document.getElementById('metaPage').style.display='none';
  renderMetas();
}

function atualizar(){
  let m = metas[metaAtual];

  let diaria = m.restante / m.dias;
  let progresso = ((m.meta - m.restante)/m.meta)*100;

  document.getElementById('tituloMeta').innerText = m.nome;
  document.getElementById('restante').innerText = `Falta: R$ ${m.restante.toFixed(2)}`;
  document.getElementById('diaria').innerText = `Meta diária: R$ ${diaria.toFixed(2)}`;
  document.getElementById('percentual').innerText = `Progresso: ${progresso.toFixed(1)}%`;

  if(diaria > 400){
    document.getElementById('alerta').innerText = `⚠️ Você está atrasado!`;
  } else {
    document.getElementById('alerta').innerText = `🔥 Bom ritmo!`;
  }

  let hist='';
  m.historico.forEach((v,i)=>{
    hist += `<div>Dia ${i+1}: R$ ${v.toFixed(2)}</div>`;
  });
  document.getElementById('historico').innerHTML = hist;

  renderDespesas();
}

function registrar(){
  let ganho = Number(document.getElementById('ganho').value);
  let m = metas[metaAtual];

  let dizimo = ganho * 0.10;
  ganho -= dizimo;

  m.restante -= ganho;
  m.dias--;
  m.historico.push(ganho);

  salvar();
  atualizar();
}

function resetarMeta(){
  let m = metas[metaAtual];
  m.restante = m.meta;
  m.historico = [];
  salvar();
  atualizar();
}

function excluirMeta(){
  metas.splice(metaAtual,1);
  salvar();
  voltar();
}

function addDespesa(){
  let nome = document.getElementById('nomeDespesa').value;
  let valor = Number(document.getElementById('valorDespesa').value);

  metas[metaAtual].despesas.push({nome, valor});
  salvar();
  renderDespesas();
}

function removerDespesa(index){
  metas[metaAtual].despesas.splice(index,1);
  salvar();
  renderDespesas();
}

function renderDespesas(){
  let m = metas[metaAtual];
  let total = m.despesas.reduce((s,d)=>s+d.valor,0);
  let ganhoTotal = m.meta - m.restante;

  let prioridade = null;
  let maiorFalta = 0;

  let html='';

  m.despesas.forEach((d,i)=>{
    let perc = (d.valor/total)*100;
    let alocado = ganhoTotal * (perc/100);
    let falta = d.valor - alocado;

    if(falta > maiorFalta){
      maiorFalta = falta;
      prioridade = d.nome;
    }

    html += `<div>
      ${d.nome}: R$ ${d.valor} (${perc.toFixed(1)}%)<br>
      👉 Alocado: R$ ${alocado.toFixed(2)}<br>
      ❗ Falta: R$ ${falta.toFixed(2)}
      <button class='red' onclick='removerDespesa(${i})'>X</button>
    </div><br>`;
  });

  html += `<p><strong>💡 Prioridade agora: ${prioridade || '-'} </strong></p>`;

  document.getElementById('listaDespesas').innerHTML = html;
}

renderMetas();
</script>

</body>
</html>
