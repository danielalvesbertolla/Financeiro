<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX - Inteligência de Porcentagem</title>
<style>
body { font-family: Arial, sans-serif; background:#0f172a; color:white; padding:20px; line-height: 1.5; }
.card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; border: 1px solid #334155; }
button { padding:8px 12px; border:none; border-radius:8px; cursor:pointer; margin:3px; font-weight: bold; transition: 0.2s; }
button:hover { opacity: 0.8; }
input { padding:8px; margin:3px; border-radius:8px; border:none; background: #334155; color: white; width: 100px; }
.green { background:#22c55e; }
.red { background:#ef4444; }
.blue { background:#3b82f6; }
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

    <div style="background: #0f172a; padding: 15px; border-radius: 8px;">
        <h3>✏️ Editar Meta</h3>
        Valor: <input id="editarValor" type="number">
        Dias: <input id="editarDias" type="number">
        <button class="blue" onclick="salvarEdicaoMeta()">Salvar</button>
    </div>

    <div id="infoEstatisticas" style="margin-top:20px;">
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
    <h3>📋 Contas e Distribuição Automática</h3>
    <p><small>Ajuste uma % e o sistema redistribuirá o restante sozinho.</small></p>
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
    porcentagem: null,
    manual: false
  });
  nomeDespesa.value = ''; valorDespesa.value = '';
  recalcularPorcentagens();
}

function atualizarPorcentagemManual(i, val){
  let m = metas[metaAtual];
  let novaPerc = Number(val) / 100;
  
  if(novaPerc > 1) novaPerc = 1;
  if(novaPerc < 0) novaPerc = 0;

  m.despesas[i].porcentagem = novaPerc;
  m.despesas[i].manual = true; // Marca que o usuário definiu esta
  
  recalcularPorcentagens();
}

function recalcularPorcentagens(){
  let m = metas[metaAtual];
  let despesas = m.despesas;
  
  // Soma quanto já foi ocupado pelas definições manuais
  let ocupadoManual = despesas.reduce((s, d) => s + (d.manual ? d.porcentagem : 0), 0);
  
  // Se passou de 100%, reduz proporcionalmente as outras (ou trava)
  if(ocupadoManual > 1) {
      alert("Atenção: A soma das porcentagens excedeu 100%!");
  }

  let restante = Math.max(0, 1 - ocupadoManual);
  let automaticas = despesas.filter(d => !d.manual);
  
  // Divide o que sobrou igualmente entre as que não são manuais
  if(automaticas.length > 0) {
      let fatia = restante / automaticas.length;
      automaticas.forEach(d => d.porcentagem = fatia);
  }

  salvar(); atualizar();
}

function removerDespesa(i){ 
  metas[metaAtual].despesas.splice(i,1); 
  recalcularPorcentagens(); 
}

function renderDespesas(diaria, ganhoTotal){
  let m = metas[metaAtual];
  let html='';

  m.despesas.forEach((d, i)=>{
    let alocado = ganhoTotal * d.porcentagem;
    let hoje = diaria * d.porcentagem;

    html+=`
    <div class="despesa-item">
      <b>${d.nome}</b> ${d.manual ? '<span class="manual-badge">MANUAL</span>' : ''}
      <br>
      Ajustar %: <input type='number' step="1" value='${(d.porcentagem*100).toFixed(1)}' onchange='atualizarPorcentagemManual(${i},this.value)'>
      <br>
      <small>💰 Acumulado: R$ ${alocado.toFixed(2)} | 🎯 Separar Hoje: R$ ${hoje.toFixed(2)}</small>
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
