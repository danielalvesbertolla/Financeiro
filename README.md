<!DOCTYPE html>
<html lang="pt-BR">
<head>

# Financeiro
Metas Financeiras
  
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX - Quitação Inteligente</title>
<style>
body { font-family: Arial, sans-serif; background:#0f172a; color:white; padding:20px; line-height: 1.5; }
.card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; border: 1px solid #334155; }
button { padding:8px 12px; border:none; border-radius:8px; cursor:pointer; margin:3px; font-weight: bold; transition: 0.2s; }
button:hover { opacity: 0.8; }
input { padding:8px; margin:3px; border-radius:8px; border:none; background: #334155; color: white; width: 100px; }
.green { background:#22c55e; }
.red { background:#ef4444; }
.blue { background:#3b82f6; }
.orange { background:#f97316; }
.alert { margin-top:10px; font-weight:bold; padding: 10px; border-radius: 8px; }
.despesa-item { border-bottom: 1px solid #334155; padding: 10px 0; }
.manual-badge { background: #3b82f6; font-size: 10px; padding: 2px 5px; border-radius: 4px; margin-left: 5px; }
</style>
</head>
<body>

<h1>📊 Metas Financeiras PRO MAX</h1>

<div id="home" class="card">
  <h3>Suas Metas</h3>
  <div id="listaMetas"></div>
  <hr style="margin:20px 0; border: 0; border-top: 1px solid #334155;">
  <h4>Criar Nova Meta</h4>
  Nome: <input id="nomeMeta" style="width:150px">
  Valor: <input id="valorMeta" type="number">
  Dias: <input id="diasMeta" type="number">
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
    </div>

    <div class="alert" id="alerta"></div>

    <div style="margin-top:20px; background: #334155; padding: 15px; border-radius: 8px;">
        <strong>Registrar Ganho Bruto (R$):</strong>
        <input id="ganhoInput" type="number">
        <button class="green" onclick="registrar()">Registrar</button>
    </div>
  </div>

  <div class="card">
    <h3>📋 Contas e Distribuição</h3>
    
    <div style="margin-bottom:20px; background: #0f172a; padding: 10px; border-radius: 8px;">
        <button class="orange" onclick="priorizarQuitacao()">🚀 Priorizar Quitação (Menores Valores)</button>
        <p style="margin: 5px 0 0 0;"><small>Isso focará 70% do seu ganho nas contas mais fáceis de quitar hoje.</small></p>
    </div>

    <div style="margin-bottom:20px;">
      Nome: <input id="nomeDespesa" style="width:120px">
      Valor: <input id="valorDespesa" type="number">
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

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }

function renderMetas(){
  let html='';
  metas.forEach((m,i)=>{
    html+=`<div style="margin-bottom:10px;"><b>${m.nome}</b> - R$ ${m.meta} 
    <button class="blue" onclick='abrirMeta(${i})'>Abrir</button></div>`;
  });
  listaMetas.innerHTML=html;
}

function criarMeta(){
  if(!nomeMeta.value || !valorMeta.value) return;
  metas.push({
    nome: nomeMeta.value,
    meta: Number(valorMeta.value),
    diasTotal: Number(diasMeta.value),
    historico: [],
    despesas: []
  });
  salvar(); renderMetas();
  nomeMeta.value = ''; valorMeta.value = ''; diasMeta.value = '';
}

function abrirMeta(i){
  metaAtual=i;
  home.style.display='none';
  metaPage.style.display='block';
  atualizar();
}

function voltar(){ home.style.display='block'; metaPage.style.display='none'; renderMetas(); }

function totalGanho(m){ return m.historico.reduce((s,v)=>s+v,0); }

function atualizar(){
  let m = metas[metaAtual];
  let ganho = totalGanho(m);
  let dia = m.historico.length;
  let restanteVal = m.meta - ganho;
  let diasRest = m.diasTotal - dia;
  
  let diariaLiquida = restanteVal / (diasRest > 0 ? diasRest : 1);
  let diariaBruta = diariaLiquida / 0.9;
  let progresso = (ganho / m.meta) * 100;
  let deveria = (m.meta / m.diasTotal) * dia;

  tituloMeta.innerText = m.nome;
  diaAtual.innerText = `📅 Dia ${dia} de ${m.diasTotal}`;
  percentual.innerText = `📈 Progresso: ${progresso.toFixed(1)}%`;
  valorAtual.innerText = `💰 Você tem: R$ ${ganho.toFixed(2)}`;
  restante.innerText = `❗ Falta: R$ ${restanteVal.toFixed(2)}`;
  
  diaria.innerText = `Meta diária (LÍQUIDA): R$ ${diariaLiquida.toFixed(2)}`;
  proximoDia.innerText = `👉 Próximo Bruto sugerido: R$ ${diariaBruta.toFixed(2)}`;

  alerta.innerText = ganho < deveria ? '⚠️ Abaixo do esperado' : '🔥 No ritmo!';
  alerta.className = 'alert ' + (ganho < deveria ? 'red' : 'green');

  renderHistorico();
  renderDespesas(diariaLiquida, ganho);
}

function registrar(){
  let m = metas[metaAtual];
  if(!ganhoInput.value) return;
  historicoBackup = [...m.historico];
  m.historico.push(Number(ganhoInput.value) * 0.9);
  ganhoInput.value = '';
  salvar(); atualizar();
}

function renderHistorico(){
  let m = metas[metaAtual];
  let html='';
  m.historico.slice().reverse().forEach((v, idx)=>{
    let originalIdx = m.historico.length - 1 - idx;
    html+=`<div>Dia ${originalIdx+1}: R$ ${v.toFixed(2)} <button class='red' onclick='apagarDia(${originalIdx})'>🗑</button></div>`;
  });
  historico.innerHTML=html;
}

function apagarDia(i){
  metas[metaAtual].historico.splice(i,1);
  salvar(); atualizar();
}

function addDespesa(){
  if(!nomeDespesa.value) return;
  metas[metaAtual].despesas.push({
    nome: nomeDespesa.value,
    valor: Number(valorDespesa.value),
    porcentagem: 0,
    manual: false
  });
  nomeDespesa.value = ''; valorDespesa.value = '';
  recalcularEquitativo();
}

function recalcularEquitativo(){
  let m = metas[metaAtual];
  if(m.despesas.length === 0) return;
  let fatia = 1 / m.despesas.length;
  m.despesas.forEach(d => { d.porcentagem = fatia; d.manual = false; });
  salvar(); atualizar();
}

function atualizarPorcentagemManual(i, val){
  let m = metas[metaAtual];
  let novaPerc = Number(val) / 100;
  m.despesas[i].porcentagem = novaPerc;
  m.despesas[i].manual = true;
  
  let ocupadoManual = m.despesas.reduce((s, d) => s + (d.manual ? d.porcentagem : 0), 0);
  let restante = Math.max(0, 1 - ocupadoManual);
  let automaticas = m.despesas.filter(d => !d.manual);
  
  if(automaticas.length > 0) {
      let fatia = restante / automaticas.length;
      automaticas.forEach(d => d.porcentagem = fatia);
  }
  salvar(); atualizar();
}

function priorizarQuitacao(){
    let m = metas[metaAtual];
    let ganhoTotal = totalGanho(m);
    
    if(m.despesas.length === 0) return;

    // 1. Calcular quanto falta para cada uma
    m.despesas.forEach(d => {
        d.faltaReal = d.valor - (ganhoTotal * d.porcentagem);
        d.manual = false; // Resetamos o manual para priorizar o algoritmo
    });

    // 2. Ordenar por quem falta menos (mais fácil de pagar)
    let ordenadas = [...m.despesas].sort((a, b) => a.faltaReal - b.faltaReal);

    // 3. Distribuição agressiva (Snowball Method)
    // A primeira (mais fácil) ganha 60%, a segunda 25%, a terceira 10%, o resto divide 5%
    ordenadas.forEach((d, index) => {
        let originalIdx = m.despesas.indexOf(d);
        if(index === 0) m.despesas[originalIdx].porcentagem = 0.60;
        else if(index === 1) m.despesas[originalIdx].porcentagem = 0.25;
        else if(index === 2) m.despesas[originalIdx].porcentagem = 0.10;
        else m.despesas[originalIdx].porcentagem = 0.05 / (ordenadas.length - 3);
    });

    salvar();
    atualizar();
}

function removerDespesa(i){ 
  metas[metaAtual].despesas.splice(i,1); 
  recalcularEquitativo(); 
}

function renderDespesas(diaria, ganhoTotal){
  let m = metas[metaAtual];
  let html='';

  m.despesas.forEach((d, i)=>{
    let alocado = ganhoTotal * d.porcentagem;
    let hoje = diaria * d.porcentagem;
    let faltaParaQuitar = d.valor - alocado;

    html+=`
    <div class="despesa-item" style="${faltaParaQuitar <= 0 ? 'opacity: 0.5; background: #064e3b;' : ''}">
      <b>${d.nome}</b> ${d.manual ? '<span class="manual-badge">MANUAL</span>' : ''}
      <br>
      Meta Total: R$ ${d.valor.toFixed(2)} | <span style="color: #fbbf24">% Atual: ${(d.porcentagem*100).toFixed(1)}%</span>
      <br>
      Ajustar %: <input type='number' value='${(d.porcentagem*100).toFixed(1)}' onchange='atualizarPorcentagemManual(${i},this.value)'>
      <br>
      <small>💰 Já Acumulado: R$ ${alocado.toFixed(2)}</small><br>
      <small style="color: ${faltaParaQuitar <= 0 ? '#22c55e' : '#ef4444'}">
        ${faltaParaQuitar <= 0 ? '✅ QUITADA!' : '❗ Falta: R$ ' + faltaParaQuitar.toFixed(2)}
      </small>
      <br>
      <small>🎯 Separar do ganho de hoje: <b>R$ ${hoje.toFixed(2)}</b></small>
      <button class='red' style="float:right" onclick='removerDespesa(${i})'>X</button>
    </div>`;
  });
  listaDespesas.innerHTML = html || '<p>Adicione contas para ver a distribuição.</p>';
}

function desfazer(){ if(historicoBackup.length > 0){ metas[metaAtual].historico = [...historicoBackup]; salvar(); atualizar(); } }
function resetarMeta(){ if(confirm("Zerar histórico?")) { metas[metaAtual].historico=[]; salvar(); atualizar(); } }
function excluirMeta(){ if(confirm("Excluir meta permanentemente?")) { metas.splice(metaAtual,1); salvar(); voltar(); } }

renderMetas();
</script>

</body>
</html>
