<!DOCTYPE html>
<html lang="pt-BR">
<head>
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
.gray { background:#64748b; }
.alert { margin-top:10px; font-weight:bold; padding: 10px; border-radius: 8px; }
.despesa-item { border-bottom: 1px solid #334155; padding: 12px 0; }
.quitada-item { opacity: 0.7; background: #064e3b; padding: 10px; border-radius: 8px; margin-bottom: 5px; border-left: 5px solid #22c55e; }
.finance-row { display: flex; justify-content: space-between; padding: 5px 0; border-bottom: 1px dashed #475569; }
</style>
</head>
<body>

<h1>📊 Metas Financeiras PRO MAX</h1>

<div id="home" class="card">
  <h3>Meus Projetos</h3>
  <div id="listaMetas"></div>
  <hr style="margin:20px 0; border: 0; border-top: 1px solid #334155;">
  <h4>Novo Projeto</h4>
  Nome: <input id="nomeMeta" style="width:150px">
  Dias: <input id="diasMeta" type="number">
  <button class="green" onclick="criarMeta()">Criar</button>
</div>

<div id="metaPage" style="display:none;">
  <div class="card">
    <button onclick="voltar()">⬅ Voltar</button>
    <button class="blue" onclick="desfazerAcoes()" title="Desfazer priorização ou exclusão de contas">↩ Desfazer Alterações</button>
    <h2 id="tituloMeta"></h2>

    <div id="infoEstatisticas">
        <p id="diaAtual"></p>
        <p id="valorAtual" style="font-size: 1.1em; font-weight: bold;"></p>
        <p id="restante" style="color: #ef4444;"></p>
        
        <div style="background: #0f172a; padding: 15px; border-radius: 8px; margin-top: 10px;">
            <div class="finance-row"><span>🎯 Meta Diária (Contas Ativas):</span> <span id="diaria" style="color: #22c55e; font-weight: bold;"></span></div>
            <div class="finance-row"><span>⛽ Combustível (Fixo):</span> <span style="color: #ef4444;">R$ 75,00</span></div>
            <div class="finance-row"><span>🙏 Dízimo (10%):</span> <span id="dizimoInfo" style="color: #fbbf24;"></span></div>
            <div class="finance-row" style="border:none; margin-top: 10px; font-size: 1.2em;">
                <span>🚀 <b>BRUTO SUGERIDO:</b></span> <span id="proximoDia" style="color: #3b82f6; font-weight: bold;"></span>
            </div>
        </div>
    </div>

    <div style="margin-top:20px; background: #334155; padding: 15px; border-radius: 8px;">
        <strong>Registrar Ganho Bruto REAL (R$):</strong>
        <input id="ganhoInput" type="number" placeholder="0.00">
        <button class="green" onclick="registrar()">Registrar</button>
    </div>
  </div>

  <div class="card">
    <h3>📋 Contas em Aberto</h3>
    <div style="margin-bottom:20px;">
        <button class="orange" onclick="priorizarQuitacao()">🚀 Priorizar Quitação Inteligente</button>
    </div>
    <div style="margin-bottom:20px;">
      Nome: <input id="nomeDespesa" style="width:120px">
      Valor: <input id="valorDespesa" type="number">
      <button class="blue" onclick="addDespesa()">Adicionar</button>
    </div>
    <div id="listaDespesas"></div>
  </div>

  <div class="card" style="border-top: 4px solid #22c55e;">
    <h3>✅ Contas Quitadas</h3>
    <div id="listaQuitadas"></div>
  </div>

  <div class="card">
    <h3>📜 Histórico Líquido</h3>
    <div id="historico"></div>
  </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;
let despesasBackup = null; // Backup para o botão Desfazer Alterações
const CUSTO_COMBUSTIVEL = 75;

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }

function renderMetas(){
  let html='';
  metas.forEach((m,i)=>{
    let total = m.despesas.reduce((s, d) => s + d.valor, 0);
    html+=`<div style="margin-bottom:10px;"><b>${m.nome}</b> (R$ ${total.toFixed(2)}) 
    <button class="blue" onclick='abrirMeta(${i})'>Abrir</button>
    <button class="red" onclick='excluirProjeto(${i})'>🗑</button></div>`;
  });
  listaMetas.innerHTML=html;
}

function criarMeta(){
  if(!nomeMeta.value || !diasMeta.value) return;
  metas.push({ nome: nomeMeta.value, meta: 0, diasTotal: Number(diasMeta.value), historico: [], despesas: [] });
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
  let ganhoTotalAcumulado = totalGanho(m);
  
  // 1. Identificar o que está quitado e o que está ativo
  let ativas = m.despesas.filter(d => (ganhoTotalAcumulado * d.porcentagem) < d.valor);
  let quitadas = m.despesas.filter(d => (ganhoTotalAcumulado * d.porcentagem) >= d.valor);

  // 2. Recalcular Meta Total (apenas das ativas para o cálculo diário)
  let metaRestanteAtivas = ativas.reduce((s, d) => s + (d.valor - (ganhoTotalAcumulado * d.porcentagem)), 0);
  
  let dia = m.historico.length;
  let diasRest = Math.max(1, m.diasTotal - dia);
  
  let diariaLiquida = metaRestanteAtivas / diasRest;
  let diariaBruta = (diariaLiquida + CUSTO_COMBUSTIVEL) / 0.9;

  tituloMeta.innerText = m.nome;
  diaAtual.innerText = `📅 Dia ${dia} de ${m.diasTotal}`;
  valorAtual.innerText = `💰 Saldo p/ Contas: R$ ${ganhoTotalAcumulado.toFixed(2)}`;
  restante.innerText = `❗ Falta quitar: R$ ${metaRestanteAtivas.toFixed(2)}`;
  
  diaria.innerText = `R$ ${diariaLiquida.toFixed(2)}`;
  proximoDia.innerText = `R$ ${diariaBruta.toFixed(2)}`;
  dizimoInfo.innerText = `R$ ${(diariaBruta * 0.1).toFixed(2)}`;

  renderHistorico();
  renderListas(ativas, quitadas, diariaLiquida, ganhoTotalAcumulado);
}

function registrar(){
  let m = metas[metaAtual];
  let bruto = Number(ganhoInput.value);
  if(!bruto) return;

  let vDizimo = bruto * 0.10;
  let vContas = (bruto - vDizimo) - CUSTO_COMBUSTIVEL;

  if(confirm(`Confirmar Registro:\n🙏 Dízimo: R$ ${vDizimo.toFixed(2)}\n⛽ Combustível: R$ ${CUSTO_COMBUSTIVEL.toFixed(2)}\n💰 Para Contas: R$ ${vContas.toFixed(2)}`)) {
      m.historico.push(vContas);
      ganhoInput.value = '';
      salvar(); atualizar();
  }
}

function renderListas(ativas, quitadas, diaria, totalGeral){
  let htmlAtivas = '';
  ativas.forEach(d => {
    let alocado = totalGeral * d.porcentagem;
    let hoje = diaria * d.porcentagem;
    htmlAtivas += `<div class="despesa-item">
      <b>${d.nome}</b> <span style="color:#fbbf24">(${(d.porcentagem*100).toFixed(1)}%)</span><br>
      Falta: R$ ${(d.valor - alocado).toFixed(2)} | <small>Hoje: R$ ${hoje.toFixed(2)}</small>
      <button class="red" style="float:right" onclick="removerDespesa(${d.id})">X</button>
    </div>`;
  });
  listaDespesas.innerHTML = htmlAtivas || '<p>Todas as contas foram quitadas! 🎉</p>';

  let htmlQuitadas = '';
  quitadas.forEach(d => {
    htmlQuitadas += `<div class="quitada-item">
      <b>✅ ${d.nome}</b> - Total R$ ${d.valor.toFixed(2)}
      <button class="gray" style="float:right; font-size:10px" onclick="reabrirDespesa(${d.id})">REABRIR</button>
    </div>`;
  });
  listaQuitadas.innerHTML = htmlQuitadas || '<p>Nenhuma conta quitada ainda.</p>';
}

function addDespesa(){
  backupAcao();
  let m = metas[metaAtual];
  m.despesas.push({ nome: nomeDespesa.value, valor: Number(valorDespesa.value), porcentagem: 0, id: Date.now() });
  nomeDespesa.value = ''; valorDespesa.value = '';
  redistribuirSobra();
}

function redistribuirSobra(){
  let m = metas[metaAtual];
  let ganhoTotal = totalGanho(m);
  // Só distribui entre as que NÃO estão quitadas
  let ativas = m.despesas.filter(d => (ganhoTotal * d.porcentagem) < d.valor);
  if(ativas.length > 0){
    let fatia = 1 / ativas.length;
    m.despesas.forEach(d => {
        if(ativas.includes(d)) d.porcentagem = fatia;
        else if(!m.despesas.find(x => x.id === d.id && (ganhoTotal * x.porcentagem) >= x.valor)) d.porcentagem = 0;
    });
  }
  salvar(); atualizar();
}

function priorizarQuitacao(){
    backupAcao();
    let m = metas[metaAtual];
    let ganhoTotal = totalGanho(m);
    let ativas = m.despesas.filter(d => (ganhoTotal * d.porcentagem) < d.valor);
    
    if(ativas.length === 0) return;

    ativas.sort((a, b) => (a.valor - (ganhoTotal * a.porcentagem)) - (b.valor - (ganhoTotal * b.porcentagem)));

    let pesos = [0.70, 0.20, 0.10];
    m.despesas.forEach(d => {
        let rank = ativas.indexOf(d);
        if(rank !== -1) d.porcentagem = pesos[rank] || (0.05 / (ativas.length - 2 || 1));
        else d.porcentagem = d.porcentagem; // mantém se já estiver quitada
    });
    salvar(); atualizar();
}

function reabrirDespesa(id){
    backupAcao();
    let m = metas[metaAtual];
    let d = m.despesas.find(x => x.id === id);
    d.porcentagem = 0; // Zera para redistribuir
    redistribuirSobra();
}

function removerDespesa(id){
    backupAcao();
    metas[metaAtual].despesas = metas[metaAtual].despesas.filter(d => d.id !== id);
    redistribuirSobra();
}

function backupAcao(){
    despesasBackup = JSON.parse(JSON.stringify(metas[metaAtual].despesas));
}

function desfazerAcoes(){
    if(despesasBackup){
        metas[metaAtual].despesas = JSON.parse(JSON.stringify(despesasBackup));
        despesasBackup = null;
        salvar(); atualizar();
    } else {
        alert("Não há alterações recentes para desfazer.");
    }
}

function renderHistorico(){
  let m = metas[metaAtual];
  let html = m.historico.slice().reverse().map((v, i) => `<div>Dia ${m.historico.length - i}: R$ ${v.toFixed(2)}</div>`).join('');
  historico.innerHTML = html;
}

function excluirProjeto(i){ if(confirm("Excluir projeto?")) { metas.splice(i,1); salvar(); renderMetas(); } }

renderMetas();
</script>

</body>
</html>
