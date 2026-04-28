<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meta Financeira PRO MAX - Full Edition</title>
    <style>
        body { font-family: Arial, sans-serif; background:#0f172a; color:white; padding:20px; line-height: 1.5; }
        .card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; border: 1px solid #334155; }
        button { padding:8px 12px; border:none; border-radius:8px; cursor:pointer; margin:3px; font-weight: bold; transition: 0.2s; }
        button:hover { opacity: 0.8; }
        input { padding:8px; margin:3px; border-radius:8px; border:none; background: #334155; color: white; width: 100px; }
        .green { background:#22c55e; color: white; }
        .red { background:#ef4444; color: white; }
        .blue { background:#3b82f6; color: white; }
        .orange { background:#f97316; color: white; }
        .alert { margin-top:10px; font-weight:bold; padding: 10px; border-radius: 8px; }
        .despesa-item { border-bottom: 1px solid #334155; padding: 10px 0; transition: all 0.3s ease; position: relative; }
        .manual-badge { background: #3b82f6; font-size: 10px; padding: 2px 5px; border-radius: 4px; margin-left: 5px; }
        hr { margin:20px 0; border: 0; border-top: 1px solid #334155; }
    </style>
</head>
<body>

<h1>📊 Metas Financeiras PRO MAX</h1>

<div id="home" class="card">
  <h3>Suas Metas</h3>
  <div id="listaMetas"></div>
  <hr>
  <h4>Criar Nova Meta</h4>
  Nome: <input id="nomeMeta" style="width:150px" placeholder="Ex: Carro Novo">
  Valor: <input id="valorMeta" type="number" placeholder="R$">
  Dias: <input id="diasMeta" type="number" placeholder="Ex: 30">
  <button class="green" onclick="criarMeta()">Criar</button>
</div>

<div id="metaPage" style="display:none;">
  <div class="card">
    <button onclick="voltar()">⬅ Voltar</button>
    <button class="blue" onclick="desfazer()">↩ Desfazer Último</button>
    <button class="red" onclick="resetarMeta()">Resetar Histórico</button>
    <button class="red" onclick="excluirMeta()">Excluir Meta</button>

    <h2 id="tituloMeta"></h2>

    <div id="infoEstatisticas">
        <p id="diaAtual"></p>
        <p id="percentual"></p>
        <p id="valorAtual" style="font-size: 1.2em; font-weight: bold;"></p>
        <p id="restante"></p>
        <hr>
        <p id="diaria" style="color: #22c55e; font-weight: bold;"></p>
        <p id="proximoDia"></p>
        <p id="dizimoInfo" style="color: #fbbf24; font-weight: bold;"></p>
    </div>

    <div class="alert" id="alerta"></div>

    <div style="margin-top:20px; background: #334155; padding: 15px; border-radius: 8px;">
        <strong>Registrar Ganho Bruto (R$):</strong>
        <input id="ganhoInput" type="number" placeholder="0.00">
        <button class="green" onclick="registrar()">Registrar</button>
    </div>
  </div>

  <div class="card">
    <h3>📋 Contas e Distribuição (Ordenado por Prioridade)</h3>
    
    <div style="margin-bottom:20px; background: #0f172a; padding: 10px; border-radius: 8px;">
        <button class="orange" onclick="priorizarQuitacao()">🚀 Priorizar Quitação (Menores Valores)</button>
        <p style="margin: 5px 0 0 0;"><small>Foca o esforço nas contas mais próximas de acabar.</small></p>
    </div>

    <div style="margin-bottom:20px;">
      Nome: <input id="nomeDespesa" style="width:120px" placeholder="Ex: Aluguel">
      Valor: <input id="valorDespesa" type="number" placeholder="R$">
      <button class="blue" onclick="addDespesa()">Adicionar Conta</button>
    </div>
    <div id="listaDespesas"></div>
  </div>

  <div class="card">
    <h3>📜 Histórico (Líquido)</h3>
    <div id="historico"></div>
  </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;
let historicoBackup = [];

const salvar = () => localStorage.setItem('metas', JSON.stringify(metas));

function renderMetas(){
  const lista = document.getElementById('listaMetas');
  let html='';
  metas.forEach((m,i)=>{
    html+=`<div style="margin-bottom:10px; display: flex; align-items: center; justify-content: space-between; background: #334155; padding: 10px; border-radius: 8px;">
    <span><b>${m.nome}</b> - R$ ${m.meta.toLocaleString('pt-BR')}</span>
    <button class="blue" onclick='abrirMeta(${i})'>Abrir</button></div>`;
  });
  lista.innerHTML = html || '<p>Nenhuma meta criada.</p>';
}

function criarMeta(){
  const nome = document.getElementById('nomeMeta');
  const valor = document.getElementById('valorMeta');
  const dias = document.getElementById('diasMeta');

  if(!nome.value || !valor.value) return alert("Preencha nome e valor!");
  
  metas.push({
    nome: nome.value,
    meta: Number(valor.value),
    diasTotal: Number(dias.value) || 1,
    historico: [],
    despesas: []
  });
  
  salvar(); 
  renderMetas();
  nome.value = ''; valor.value = ''; dias.value = '';
}

function abrirMeta(i){
  metaAtual=i;
  document.getElementById('home').style.display='none';
  document.getElementById('metaPage').style.display='block';
  atualizar();
}

function voltar(){ 
  document.getElementById('home').style.display='block'; 
  document.getElementById('metaPage').style.display='none'; 
  renderMetas(); 
}

function totalGanho(m){ return m.historico.reduce((s,v)=>s+v,0); }

function atualizar(){
  let m = metas[metaAtual];
  let ganho = totalGanho(m);
  let dia = m.historico.length;
  let restanteVal = m.meta - ganho;
  let diasRest = m.diasTotal - dia;
  
  // Evitar divisão por zero ou números negativos
  let diariaLiquida = diasRest > 0 ? restanteVal / diasRest : restanteVal;
  if (diariaLiquida < 0) diariaLiquida = 0;

  let diariaBruta = diariaLiquida / 0.9;
  let progresso = (ganho / m.meta) * 100;
  let deveria = (m.meta / m.diasTotal) * dia;

  document.getElementById('tituloMeta').innerText = m.nome;
  document.getElementById('diaAtual').innerText = `📅 Dia ${dia} de ${m.diasTotal}`;
  document.getElementById('percentual').innerText = `📈 Progresso: ${progresso.toFixed(1)}%`;
  document.getElementById('valorAtual').innerText = `💰 Você tem acumulado: R$ ${ganho.toFixed(2)}`;
  document.getElementById('restante').innerText = `❗ Falta: R$ ${Math.max(0, restanteVal).toFixed(2)}`;
  
  document.getElementById('diaria').innerText = `Meta diária (LÍQUIDA): R$ ${diariaLiquida.toFixed(2)}`;
  document.getElementById('proximoDia').innerText = `👉 Próximo Bruto sugerido: R$ ${diariaBruta.toFixed(2)}`;
  document.getElementById('dizimoInfo').innerText = `🙏 Dízimo sugerido (10%): R$ ${(diariaBruta * 0.1).toFixed(2)}`;

  const alertBox = document.getElementById('alerta');
  alertBox.innerText = ganho < deveria ? '⚠️ Abaixo do esperado para o dia' : '🔥 No ritmo/Meta batida!';
  alertBox.className = 'alert ' + (ganho < deveria ? 'red' : 'green');

  renderHistorico();
  renderDespesas(diariaLiquida, ganho);
}

function registrar(){
  let m = metas[metaAtual];
  const input = document.getElementById('ganhoInput');
  if(!input.value) return;
  
  historicoBackup = [...m.historico];
  m.historico.push(Number(input.value) * 0.9);
  input.value = '';
  salvar(); 
  atualizar();
}

function renderHistorico(){
  let m = metas[metaAtual];
  let html='';
  [...m.historico].reverse().forEach((v, idx)=>{
    let originalIdx = m.historico.length - 1 - idx;
    html+=`<div style="display:flex; justify-content:space-between; margin-bottom:5px;">
        <span>Dia ${originalIdx+1}: R$ ${v.toFixed(2)}</span>
        <button class='red' style="padding:2px 8px" onclick='apagarDia(${originalIdx})'>🗑️</button>
    </div>`;
  });
  document.getElementById('historico').innerHTML = html || 'Sem registros.';
}

function apagarDia(i){
  metas[metaAtual].historico.splice(i,1);
  salvar(); atualizar();
}

function addDespesa(){
  const nome = document.getElementById('nomeDespesa');
  const valor = document.getElementById('valorDespesa');
  if(!nome.value || !valor.value) return;

  metas[metaAtual].despesas.push({
    nome: nome.value,
    valor: Number(valor.value),
    porcentagem: 0,
    manual: false,
    id: Date.now()
  });
  nome.value = ''; valor.value = '';
  recalcularEquitativo();
}

function recalcularEquitativo(){
  let m = metas[metaAtual];
  if(m.despesas.length === 0) return;
  let fatia = 1 / m.despesas.length;
  m.despesas.forEach(d => { d.porcentagem = fatia; d.manual = false; });
  salvar(); atualizar();
}

function atualizarPorcentagemManual(id, val){
  let m = metas[metaAtual];
  let index = m.despesas.findIndex(d => d.id === id);
  let novaPerc = Number(val) / 100;
  
  m.despesas[index].porcentagem = novaPerc;
  m.despesas[index].manual = true;
  
  let ocupadoManual = m.despesas.reduce((s, d) => s + (d.manual ? d.porcentagem : 0), 0);
  let disponivel = Math.max(0, 1 - ocupadoManual);
  let automaticas = m.despesas.filter(d => !d.manual);
  
  if(automaticas.length > 0) {
      let fatia = disponivel / automaticas.length;
      automaticas.forEach(d => d.porcentagem = fatia);
  }
  salvar(); atualizar();
}

function priorizarQuitacao(){
    let m = metas[metaAtual];
    let ganhoTotal = totalGanho(m);
    if(m.despesas.length === 0) return;

    // Ordenar por quem falta menos para quitar
    let ordenadas = [...m.despesas].sort((a, b) => {
        let faltaA = a.valor - (ganhoTotal * a.porcentagem);
        let faltaB = b.valor - (ganhoTotal * b.porcentagem);
        return faltaA - faltaB;
    });

    ordenadas.forEach((d, index) => {
        let realD = m.despesas.find(x => x.id === d.id);
        realD.manual = false;
        if(index === 0) realD.porcentagem = 0.60;
        else if(index === 1) realD.porcentagem = 0.25;
        else if(index === 2) realD.porcentagem = 0.10;
        else realD.porcentagem = 0.05 / (ordenadas.length - 3 || 1);
    });

    salvar(); atualizar();
}

function removerDespesa(id){ 
  let m = metas[metaAtual];
  m.despesas = m.despesas.filter(d => d.id !== id);
  recalcularEquitativo(); 
}

function renderDespesas(diaria, ganhoTotal){
  let m = metas[metaAtual];
  const container = document.getElementById('listaDespesas');
  let html='';
  let exibicao = [...m.despesas].sort((a, b) => b.porcentagem - a.porcentagem);

  exibicao.forEach((d)=>{
    let alocado = ganhoTotal * d.porcentagem;
    let hoje = diaria * d.porcentagem;
    let faltaParaQuitar = d.valor - alocado;

    html+=`
    <div class="despesa-item" style="${faltaParaQuitar <= 0 ? 'opacity: 0.6; background: #064e3b; padding:10px; border-radius:8px;' : ''}">
      <button class='red' style="float:right" onclick='removerDespesa(${d.id})'>X</button>
      <b>${d.nome}</b> ${d.manual ? '<span class="manual-badge">MANUAL</span>' : ''}
      <br>
      Meta Total: R$ ${d.valor.toFixed(2)} | <span style="color: #fbbf24; font-weight:bold;">Prioridade: ${(d.porcentagem*100).toFixed(1)}%</span>
      <br>
      Ajustar %: <input type='number' style="width:60px" value='${(d.porcentagem*100).toFixed(1)}' onchange='atualizarPorcentagemManual(${d.id},this.value)'>
      <br>
      <small>💰 Já Acumulado: R$ ${alocado.toFixed(2)}</small><br>
      <small style="color: ${faltaParaQuitar <= 0 ? '#22c55e' : '#ef4444'}">
        ${faltaParaQuitar <= 0 ? '✅ QUITADA!' : '❗ Falta: R$ ' + faltaParaQuitar.toFixed(2)}
      </small>
      <br>
      <small>🎯 Separar hoje: <b style="font-size:1.1em;">R$ ${hoje.toFixed(2)}</b></small>
    </div>`;
  });
  container.innerHTML = html || '<p>Adicione contas para ver a distribuição.</p>';
}

function desfazer(){ 
    if(metaAtual !== null && historicoBackup.length >= 0){ 
        metas[metaAtual].historico = [...historicoBackup]; 
        salvar(); 
        atualizar(); 
        alert("Última ação desfeita!");
    } 
}

function resetarMeta(){ if(confirm("Zerar histórico?")) { metas[metaAtual].historico=[]; salvar(); atualizar(); } }
function excluirMeta(){ if(confirm("Excluir meta permanentemente?")) { metas.splice(metaAtual,1); salvar(); voltar(); } }

// Inicialização
renderMetas();
</script>

</body>
</html>
