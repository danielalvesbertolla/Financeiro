# Financeiro
Metas Financeiras

<Vai Dar Bom><html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX++++</title>
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
<body><h1>📊 Metas Financeiras PRO MAX++++</h1><div id="home" class="card">
  <h3>Suas Metas</h3>
  <div id="listaMetas"></div>  <h4>Criar Nova Meta</h4>
  Nome: <input id="nomeMeta">
  Valor: <input id="valorMeta" type="number">
  Dias: <input id="diasMeta" type="number">
  <button class="green" onclick="criarMeta()">Criar</button>
</div><div id="metaPage" style="display:none;">
  <div class="card">
    <button onclick="voltar()">⬅ Voltar</button>
    <button class="red" onclick="resetarMeta()">Resetar</button>
    <button class="red" onclick="excluirMeta()">Excluir</button>
    <button class="blue" onclick="desfazer()">↩ Desfazer</button><h2 id="tituloMeta"></h2>

<h3>✏️ Editar Meta</h3>
Valor: <input id="editarValor" type="number">
Dias: <input id="editarDias" type="number">
<button class="blue" onclick="salvarEdicaoMeta()">Salvar Alterações</button>

<p id="diaAtual"></p>
<p id="diasRestantes"></p>
<p id="valorDevia"></p>
<p id="valorAtual"></p>
<p id="restante"></p>
<p id="diaria"></p>
<p id="percentual"></p>

<input id="ganhoInput" type="number">
<button class="green" onclick="registrar()">Registrar ganho</button>

<p class="alert" id="alerta"></p>

  </div>  <div class="card">
    <h3>Distribuição do dia</h3>
    <div id="distribuicaoDia"></div>
  </div>  <div class="card">
    <h3>Contas</h3>
    Nome: <input id="nomeDespesa">
    Valor: <input id="valorDespesa" type="number">
    <button class="blue" onclick="addDespesa()">Adicionar</button>
    <div id="listaDespesas"></div>
  </div>  <div class="card">
    <h3>Histórico</h3>
    <div id="historico"></div>
  </div>
</div><script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;
let historicoBackup = [];

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }

function renderMetas(){
  let html='';
  metas.forEach((m,i)=>{
    html += `<div>${m.nome} - R$ ${m.meta}
    <button class='blue' onclick='abrirMeta(${i})'>Abrir</button></div>`;
  });
  listaMetas.innerHTML = html;
}

function criarMeta(){
  metas.push({
    nome:nomeMeta.value,
    meta:Number(valorMeta.value),
    diasTotal:Number(diasMeta.value),
    historico:[],
    despesas:[{nome:'Aluguel',valor:1400},{nome:'Internet',valor:100},{nome:'Cartão',valor:2500}]
  });
  salvar(); renderMetas();
}

function abrirMeta(i){
  metaAtual=i;
  home.style.display='none';
  metaPage.style.display='block';
  carregarEdicao();
  atualizar();
}

function carregarEdicao(){
  let m = metas[metaAtual];
  editarValor.value = m.meta;
  editarDias.value = m.diasTotal;
}

function salvarEdicaoMeta(){
  let m = metas[metaAtual];
  m.meta = Number(editarValor.value);
  m.diasTotal = Number(editarDias.value);
  salvar(); atualizar();
}

function voltar(){ home.style.display='block'; metaPage.style.display='none'; renderMetas(); }

function totalGanho(m){ return m.historico.reduce((s,v)=>s+v,0); }

function atualizar(){
  let m = metas[metaAtual];
  let ganho = totalGanho(m);
  let dia = m.historico.length;
  let restante = m.meta - ganho;
  let diasRest = m.diasTotal - dia;
  let diaria = restante / (diasRest || 1);
  let progresso = (ganho/m.meta)*100;
  let deveria = (m.meta/m.diasTotal)*dia;

  tituloMeta.innerText = m.nome;
  diaAtual.innerText = `📅 Dia ${dia} de ${m.diasTotal}`;
  diasRestantes.innerText = `⏳ Dias restantes: ${diasRest}`;
  valorDevia.innerText = `📌 Deveria ter: R$ ${deveria.toFixed(2)}`;
  valorAtual.innerText = `💰 Você tem: R$ ${ganho.toFixed(2)}`;
  restante.innerText = `❗ Falta: R$ ${restante.toFixed(2)}`;
  diaria.innerText = `Meta diária: R$ ${diaria.toFixed(2)}`;
  percentual.innerText = `Progresso: ${progresso.toFixed(1)}%`;

  alerta.innerText = ganho < deveria ? '⚠️ Abaixo do esperado' : '🔥 No ritmo';

  renderHistorico();
  renderDespesas();
  renderDistribuicaoDia(diaria);
}

function registrar(){
  let m = metas[metaAtual];
  historicoBackup = [...m.historico];
  let liquido = Number(ganhoInput.value)*0.9;
  m.historico.push(liquido);
  salvar(); atualizar();
}

function editarDia(i,val){
  let m = metas[metaAtual];
  historicoBackup = [...m.historico];
  m.historico[i] = val*0.9;
  salvar(); atualizar();
}

function apagarDia(i){
  let m = metas[metaAtual];
  historicoBackup = [...m.historico];
  m.historico.splice(i,1);
  salvar(); atualizar();
}

function desfazer(){ metas[metaAtual].historico=[...historicoBackup]; salvar(); atualizar(); }

function renderHistorico(){
  let m = metas[metaAtual];
  let html='';
  m.historico.forEach((v,i)=>{
    html+=`Dia ${i+1}: 
    <input type='number' value='${v.toFixed(2)}' onchange='editarDia(${i},this.value)'>
    <button class='red' onclick='apagarDia(${i})'>🗑</button><br>`;
  });
  historico.innerHTML=html;
}

function addDespesa(){
  metas[metaAtual].despesas.push({nome:nomeDespesa.value, valor:Number(valorDespesa.value)});
  salvar(); renderDespesas();
}

function removerDespesa(i){ metas[metaAtual].despesas.splice(i,1); salvar(); renderDespesas(); }

function renderDespesas(){
  let m = metas[metaAtual];
  let total = m.despesas.reduce((s,d)=>s+d.valor,0);
  let ganho = totalGanho(m);
  let html='';
  m.despesas.forEach((d,i)=>{
    let perc = d.valor/total;
    let alocado = ganho*perc;
    let falta = d.valor - alocado;
    html+=`${d.nome}: R$${d.valor} (${(perc*100).toFixed(1)}%)<br>
    👉 Alocado: R$${alocado.toFixed(2)}<br>
    ❗ Falta: R$${falta.toFixed(2)}
    <button onclick='removerDespesa(${i})'>X</button><br><br>`;
  });
  listaDespesas.innerHTML=html;
}

function renderDistribuicaoDia(diaria){
  let m = metas[metaAtual];
  let total = m.despesas.reduce((s,d)=>s+d.valor,0);
  let html='';
  m.despesas.forEach(d=>{
    let perc = d.valor/total;
    let hoje = diaria * perc;
    html+=`${d.nome}: 👉 Hoje separar R$ ${hoje.toFixed(2)}<br>`;
  });
  distribuicaoDia.innerHTML = html;
}

function resetarMeta(){ metas[metaAtual].historico=[]; salvar(); atualizar(); }
function excluirMeta(){ metas.splice(metaAtual,1); salvar(); voltar(); }

renderMetas();
</script></body>
</html>
