<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX - Confirmação Real</title>
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
.despesa-item { border-bottom: 1px solid #334155; padding: 10px 0; transition: all 0.3s ease; }
.manual-badge { background: #3b82f6; font-size: 10px; padding: 2px 5px; border-radius: 4px; margin-left: 5px; }
.finance-row { display: flex; justify-content: space-between; padding: 5px 0; border-bottom: 1px dashed #475569; }
.meta-badge { background: #3b82f6; padding: 2px 8px; border-radius: 20px; font-size: 0.8em; vertical-align: middle; }
</style>
</head>
<body>

<h1>📊 Metas Financeiras PRO MAX</h1>

<div id="home" class="card">
  <h3>Suas Metas</h3>
  <div id="listaMetas"></div>
  <hr style="margin:20px 0; border: 0; border-top: 1px solid #334155;">
  <h4>Criar Novo Projeto de Quitação</h4>
  Nome: <input id="nomeMeta" style="width:150px" placeholder="Ex: Abril">
  Dias: <input id="diasMeta" type="number">
  <button class="green" onclick="criarMeta()">Começar Projeto</button>
</div>

<div id="metaPage" style="display:none;">
  <div class="card">
    <button onclick="voltar()">⬅ Voltar</button>
    <button class="blue" onclick="desfazer()">↩ Desfazer Último</button>

    <h2 id="tituloMeta"></h2>

    <div id="infoEstatisticas">
        <p id="diaAtual"></p>
        <p id="percentual"></p>
        <p id="valorAtual" style="font-size: 1.1em; font-weight: bold;"></p>
        <p id="restante" style="color: #ef4444;"></p>
        
        <div style="background: #0f172a; padding: 15px; border-radius: 8px; margin-top: 10px;">
            <div class="finance-row"><span>🎯 Meta Diária p/ Contas:</span> <span id="diaria" style="color: #22c55e; font-weight: bold;"></span></div>
            <div class="finance-row"><span>⛽ Combustível (Fixo):</span> <span style="color: #ef4444;">R$ 75,00</span></div>
            <div class="finance-row"><span>🙏 Dízimo (10%):</span> <span id="dizimoInfo" style="color: #fbbf24;"></span></div>
            <div class="finance-row" style="border:none; margin-top: 10px; font-size: 1.2em;">
                <span>🚀 <b>BRUTO SUGERIDO:</b></span> <span id="proximoDia" style="color: #3b82f6; font-weight: bold;"></span>
            </div>
        </div>
    </div>

    <div class="alert" id="alerta"></div>

    <div style="margin-top:20px; background: #334155; padding: 15px; border-radius: 8px;">
        <strong>Registrar Ganho Bruto REAL (R$):</strong>
        <input id="ganhoInput" type="number" placeholder="0.00">
        <button class="green" onclick="registrar()">Registrar</button>
    </div>
  </div>

  <div class="card">
    <h3>📋 Divisão das Contas</h3>
    <div style="margin-bottom:20px; background: #0f172a; padding: 15px; border-radius: 8px; border: 1px solid #f97316;">
        <button class="orange" onclick="priorizarQuitacao()">🚀 Priorizar Quitação</button>
    </div>

    <div style="margin-bottom:20px;">
      Nome: <input id="nomeDespesa" style="width:120px">
      Valor: <input id="valorDespesa" type="number">
      <button class="blue" onclick="addDespesa()">Adicionar</button>
    </div>
    <div id="listaDespesas"></div>
  </div>

  <div class="card">
    <h3>📜 Histórico Líquido</h3>
    <div id="historico"></div>
  </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;
let historicoBackup = [];
const CUSTO_COMBUSTIVEL = 75;

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }

function renderMetas(){
  let html='';
  metas.forEach((m,i)=>{
    let metaTotalCalculada = m.despesas.reduce((s, d) => s + d.valor, 0);
    html+=`<div style="margin-bottom:10px;"><b>${m.nome}</b> (R$ ${metaTotalCalculada.toFixed(2)}) 
    <button class="blue" onclick='abrirMeta(${i})'>Abrir</button>
    <button class="red" onclick='excluirMetaTotal(${i})'>🗑</button></div>`;
  });
  listaMetas.innerHTML=html;
}

function criarMeta(){
  if(!nomeMeta.value || !diasMeta.value) return;
  metas.push({
    nome: nomeMeta.value,
    meta: 0,
    diasTotal: Number(diasMeta.value),
    historico: [],
    despesas: []
  });
  salvar(); renderMetas();
  nomeMeta.value = ''; diasMeta.value = '';
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
  let metaTotalDinamica = m.despesas.reduce((s, d) => s + d.valor, 0);
  m.meta = metaTotalDinamica;

  let ganho = totalGanho(m);
  let dia = m.historico.length;
  let restanteVal = m.meta - ganho;
  let diasRest = m.diasTotal - dia;
  
  let diariaLiquida = restanteVal / (diasRest > 0 ? diasRest : 1);
  let diariaBruta = (diariaLiquida + CUSTO_COMBUSTIVEL) / 0.9;
  
  let progresso = m.meta > 0 ? (ganho / m.meta) * 100 : 0;
  let deveria = (m.meta / m.diasTotal) * dia;

  tituloMeta.innerText = m.nome;
  diaAtual.innerText = `📅 Dia ${dia} de ${m.diasTotal}`;
  percentual.innerText = `📈 Progresso: ${progresso.toFixed(1)}%`;
  valorAtual.innerText = `💰 Acumulado p/ Contas: R$ ${ganho.toFixed(2)}`;
  restante.innerText = `❗ Falta Quitar: R$ ${restanteVal.toFixed(2)}`;
  
  diaria.innerText = `R$ ${diariaLiquida.toFixed(2)}`;
  proximoDia.innerText = `R$ ${diariaBruta.toFixed(2)}`;
  dizimoInfo.innerText = `R$ ${(diariaBruta * 0.1).toFixed(2)}`;

  alerta.innerText = ganho < deveria ? '⚠️ Atrasado' : '🔥 No Ritmo';
  alerta.className = 'alert ' + (ganho < deveria ? 'red' : 'green');

  renderHistorico();
  renderDespesas(diariaLiquida, ganho);
}

function registrar(){
  let m = metas[metaAtual];
  let brutoInserido = Number(ganhoInput.value);
  if(!brutoInserido || brutoInserido <= 0) return;

  // CÁLCULOS REAIS BASEADOS NO QUE FOI DIGITADO
  let valorDizimo = brutoInserido * 0.10;
  let liquidoAposDizimo = brutoInserido - valorDizimo;
  let valorParaContas = liquidoAposDizimo - CUSTO_COMBUSTIVEL;

  // CAIXA DE DIÁLOGO DE CONFIRMAÇÃO
  let mensagem = `DISTRIBUIÇÃO DO VALOR REGISTRADO (R$ ${brutoInserido.toFixed(2)}):\n\n` +
                 `🙏 Dízimo (10%): R$ ${valorDizimo.toFixed(2)}\n` +
                 `⛽ Combustível: R$ ${CUSTO_COMBUSTIVEL.toFixed(2)}\n` +
                 `-----------------------------------\n` +
                 `💰 SOBRA PARA AS CONTAS: R$ ${valorParaContas.toFixed(2)}\n\n` +
                 `Deseja confirmar este registro?`;

  if(confirm(mensagem)) {
      historicoBackup = [...m.historico];
      m.historico.push(valorParaContas);
      ganhoInput.value = '';
      salvar(); atualizar();
  }
}

function renderHistorico(){
  let m = metas[metaAtual];
  let html='';
  m.historico.slice().reverse().forEach((v, idx)=>{
    let originalIdx = m.historico.length - 1 - idx;
    html+=`<div>Dia ${originalIdx+1}: + R$ ${v.toFixed(2)} <button class='red' style="padding:2px 5px" onclick='apagarDia(${originalIdx})'>🗑</button></div>`;
  });
  historico.innerHTML=html;
}

function apagarDia(i){
  metas[metaAtual].historico.splice(i,1);
  salvar(); atualizar();
}

function addDespesa(){
  if(!nomeDespesa.value || !valorDespesa.value) return;
  metas[metaAtual].despesas.push({
    nome: nomeDespesa.value,
    valor: Number(valorDespesa.value),
    porcentagem: 0,
    manual: false,
    id: Date.now()
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

function atualizarPorcentagemManual(id, val){
  let m = metas[metaAtual];
  let index = m.despesas.findIndex(d => d.id === id);
  let novaPerc = Number(val) / 100;
  m.despesas[index].porcentagem = novaPerc;
  m.despesas[index].manual = true;
  
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
    let despesasAtivas = m.despesas.filter(d => (d.valor - (ganhoTotal * d.porcentagem)) > 0);
    if(despesasAtivas.length === 0) return;

    despesasAtivas.forEach(d => {
        d.faltaReal = d.valor - (ganhoTotal * d.porcentagem);
        d.manual = false;
    });
    despesasAtivas.sort((a, b) => a.faltaReal - b.faltaReal);

    let pesos = [0.60, 0.25, 0.10, 0.05];
    m.despesas.forEach(d => {
        let rank = despesasAtivas.indexOf(d);
        if(rank !== -1) {
            d.porcentagem = pesos[rank] || (0.05 / (despesasAtivas.length - 3 || 1));
        } else {
            d.porcentagem = 0;
        }
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
  let html='';
  let exibicao = [...m.despesas].sort((a, b) => b.porcentagem - a.porcentagem);

  exibicao.forEach((d)=>{
    let alocado = ganhoTotal * d.porcentagem;
    let hoje = diaria * d.porcentagem;
    let faltaParaQuitar = d.valor - alocado;

    html+=`
    <div class="despesa-item" style="${faltaParaQuitar <= 0 ? 'opacity: 0.5; background: #064e3b;' : ''}">
      <b>${d.nome}</b> ${d.manual ? '<span class="manual-badge">M</span>' : ''}
      <br>
      Total: R$ ${d.valor.toFixed(2)} | Prioridade: ${(d.porcentagem*100).toFixed(1)}%
      <br>
      Ajustar %: <input type='number' value='${(d.porcentagem*100).toFixed(1)}' onchange='atualizarPorcentagemManual(${d.id},this.value)'>
      <br>
      <small>💰 Acumulado: R$ ${alocado.toFixed(2)}</small><br>
      <small style="color: ${faltaParaQuitar <= 0 ? '#22c55e' : '#ef4444'}; font-weight: bold;">
        ${faltaParaQuitar <= 0 ? '✅ QUITADA!' : '❗ Falta: R$ ' + faltaParaQuitar.toFixed(2)}
      </small>
      <br>
      <small>🎯 Separar de Hoje: <b>R$ ${hoje.toFixed(2)}</b></small>
      <button class='red' style="float:right" onclick='removerDespesa(${d.id})'>X</button>
    </div>`;
  });
  listaDespesas.innerHTML = html || '<p>Adicione contas.</p>';
}

function desfazer(){ if(historicoBackup.length > 0){ metas[metaAtual].historico = [...historicoBackup]; salvar(); atualizar(); } }
function excluirMetaTotal(i){ if(confirm("Excluir projeto?")) { metas.splice(i,1); salvar(); renderMetas(); } }

renderMetas();
</script>

</body>
</html>
