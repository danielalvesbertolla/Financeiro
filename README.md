<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX - Versão Final Unificada</title>
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
        <h3>✏️ Editar Configurações</h3>
        Valor: <input id="editarValor" type="number">
        Dias: <input id="editarDias" type="number">
        <button class="blue" onclick="salvarEdicaoMeta()">Salvar Edição</button>
    </div>

    <div id="infoEstatisticas" style="margin-top:20px;">
        <p id="diaAtual"></p>
        <p id="diasRestantes"></p>
        <p id="percentual"></p>
        <p id="valorDevia" style="color: #94a3b8;"></p>
        <p id="valorAtual" style="font-size: 1.2em; font-weight: bold;"></p>
        <p id="restante"></p>
        <hr>
        <p id="diaria" style="color: #22c55e; font-weight: bold;"></p>
        <p id="proximoDia"></p>
        <p id="dizimoInfo" style="color: #fbbf24;"></p>
    </div>

    <div class="alert" id="alerta"></div>

    <div style="margin-top:20px; background: #334155; padding: 15px; border-radius: 8px;">
        <strong>Registrar Ganho Bruto (R$):</strong>
        <input id="ganhoInput" type="number" placeholder="Ex: 100">
        <button class="green" onclick="registrar()">Registrar</button>
    </div>
  </div>

  <div class="card">
    <h3>📋 Contas e Distribuição</h3>
    <div style="margin-bottom:20px;">
      Nome: <input id="nomeDespesa" style="width:120px">
      Valor: <input id="valorDespesa" type="number">
      <button class="blue" onclick="addDespesa()">Adicionar Conta</button>
    </div>
    <div id="listaDespesas"></div>
  </div>

  <div class="card">
    <h3>📜 Histórico de Ganhos (Líquido)</h3>
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
  diasRestantes.innerText = `⏳ Dias restantes: ${diasRest < 0 ? 0 : diasRest}`;
  percentual.innerText = `📈 Progresso: ${progresso.toFixed(1)}%`;
  valorDevia.innerText = `📌 Deveria ter: R$ ${deveria.toFixed(2)}`;
  valorAtual.innerText = `💰 Você tem: R$ ${ganho.toFixed(2)}`;
  restante.innerText = `❗ Falta: R$ ${restanteVal.toFixed(2)}`;
  
  diaria.innerText = `Meta diária (LÍQUIDA): R$ ${diariaLiquida.toFixed(2)}`;
  proximoDia.innerText = `👉 Próximo Bruto sugerido: R$ ${diariaBruta.toFixed(2)}`;
  dizimoInfo.innerText = `🙏 Dízimo sugerido (10%): R$ ${(diariaBruta*0.1).toFixed(2)}`;

  alerta.innerText = ganho < deveria ? '⚠️ Abaixo do esperado' : '🔥 No ritmo ou Meta Batida!';
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

function desfazer(){ 
    if(historicoBackup.length > 0 || metas[metaAtual].historico.length > 0) {
        metas[metaAtual].historico = [...historicoBackup]; 
        salvar(); atualizar(); 
    }
}

function renderHistorico(){
  let m = metas[metaAtual];
  let html='';
  m.historico.slice().reverse().forEach((v, idx)=>{
    let originalIdx = m.historico.length - 1 - idx;
    html+=`<div style="margin-bottom:5px;">Dia ${originalIdx+1}: 
    <input type='number' value='${(v/0.9).toFixed(2)}' onchange='editarDia(${originalIdx},this.value)'> (Bruto)
    <button class='red' onclick='apagarDia(${originalIdx})'>🗑</button></div>`;
  });
  historico.innerHTML=html;
}

function editarDia(i, val){
  let m = metas[metaAtual];
  historicoBackup = [...m.historico];
  m.historico[i] = Number(val) * 0.9;
  salvar(); atualizar();
}

function apagarDia(i){
  let m = metas[metaAtual];
  historicoBackup = [...m.historico];
  m.historico.splice(i,1);
  salvar(); atualizar();
}

function addDespesa(){
  if(!nomeDespesa.value) return;
  metas[metaAtual].despesas.push({
    nome: nomeDespesa.value,
    valor: Number(valorDespesa.value),
    porcentagem: null
  });
  nomeDespesa.value = ''; valorDespesa.value = '';
  salvar(); atualizar();
}

function mover(i, dir){
  let arr = metas[metaAtual].despesas;
  let novo = i + dir;
  if(novo < 0 || novo >= arr.length) return;
  [arr[i], arr[novo]] = [arr[novo], arr[i]];
  salvar(); atualizar();
}

function atualizarPorcentagem(i, val){
  let m = metas[metaAtual];
  m.despesas[i].porcentagem = Number(val)/100;
  
  // Recalcular automáticos (quem está com null ou zero)
  let definido = m.despesas.reduce((s,d)=>s+(d.porcentagem||0),0);
  let restante = 1 - definido;
  if(restante < 0) { alert("Porcentagem total excedeu 100%!"); }
  
  salvar(); atualizar();
}

function removerDespesa(i){ metas[metaAtual].despesas.splice(i,1); salvar(); atualizar(); }

function renderDespesas(diaria, ganhoTotal){
  let m = metas[metaAtual];
  let despesas = m.despesas;
  let totalCustos = despesas.reduce((s,d)=>s+d.valor,0);
  let html='';

  despesas.forEach((d, i)=>{
    // Se não tiver porcentagem manual, calcula baseado no peso do valor total das contas
    let perc = d.porcentagem ?? (totalCustos > 0 ? d.valor / totalCustos : 0);
    let alocado = ganhoTotal * perc;
    let hoje = diaria * perc;

    html+=`
    <div class="despesa-item">
      <b>${d.nome}</b> (Meta: R$ ${d.valor}) | <b>${(perc*100).toFixed(1)}%</b>
      <br>
      <small>💰 Acumulado: R$ ${alocado.toFixed(2)} | 🎯 Separar Hoje: R$ ${hoje.toFixed(2)}</small>
      <br>
      Ajustar %: <input type='number' step="0.1" value='${(perc*100).toFixed(1)}' onchange='atualizarPorcentagem(${i},this.value)'>
      <button onclick='mover(${i},-1)'>⬆</button>
      <button onclick='mover(${i},1)'>⬇</button>
      <button class='red' onclick='removerDespesa(${i})'>X</button>
    </div>`;
  });
  listaDespesas.innerHTML = html || '<p>Nenhuma conta adicionada.</p>';
}

function resetarMeta(){ if(confirm("Zerar histórico?")) { metas[metaAtual].historico=[]; salvar(); atualizar(); } }
function excluirMeta(){ if(confirm("Excluir meta permanentemente?")) { metas.splice(metaAtual,1); salvar(); voltar(); } }

renderMetas();
</script>

</body>
</html>
