# Financeiro
Metas Financeiras

<Vai Dar Bom><html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX ULTRA</title>
<style>
body { font-family: Arial; background:#0f172a; color:white; padding:20px; }
.card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; }
button { padding:6px 10px; border:none; border-radius:8px; cursor:pointer; margin:3px; }
input { padding:6px; margin:3px; border-radius:8px; border:none; }
.green { background:#22c55e; }
.red { background:#ef4444; }
.blue { background:#3b82f6; }
.alert { margin-top:10px; font-weight:bold; }
</style>
</head>
<body><h1>📊 Metas Financeiras PRO MAX ULTRA</h1><div id="home" class="card">
  <h3>Suas Metas</h3>
  <div id="listaMetas"></div>  <h4>Criar Nova Meta</h4>
  Nome: <input id="nomeMeta">
  Valor: <input id="valorMeta" type="number">
  Dias: <input id="diasMeta" type="number">
  <button class="green" onclick="criarMeta()">Criar</button>
</div><div id="metaPage" style="display:none;">
  <div class="card">
    <button onclick="voltar()">⬅ Voltar</button>
    <button class="blue" onclick="desfazer()">↩ Desfazer</button><h2 id="tituloMeta"></h2>

<p id="diaria"></p>
<p id="proximoDia"></p>
<p id="dizimoInfo"></p>

<input id="ganhoInput" type="number">
<button class="green" onclick="registrar()">Registrar ganho</button>

<p class="alert" id="alerta"></p>

  </div>  <div class="card">
    <h3>Contas (Prioridade + % customizável)</h3><div style="margin-bottom:20px;">
  Nome: <input id="nomeDespesa">
  Valor: <input id="valorDespesa" type="number">
  <button class="blue" onclick="addDespesa()">Adicionar</button>
</div>

<div id="listaDespesas"></div>

  </div>
</div><script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;
let historicoBackup = [];

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }

function criarMeta(){
  metas.push({
    nome:nomeMeta.value,
    meta:Number(valorMeta.value),
    diasTotal:Number(diasMeta.value),
    historico:[],
    despesas:[]
  });
  salvar(); renderMetas();
}

function renderMetas(){
  let html='';
  metas.forEach((m,i)=>{
    html+=`<div>${m.nome}
    <button onclick='abrirMeta(${i})'>Abrir</button></div>`;
  });
  listaMetas.innerHTML=html;
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
  let restante = m.meta - ganho;
  let diasRest = m.diasTotal - dia;

  let diariaLiquida = restante / (diasRest || 1);
  let diariaBruta = diariaLiquida / 0.9;

  diaria.innerText = `Meta líquida: R$ ${diariaLiquida.toFixed(2)}`;
  proximoDia.innerText = `👉 Fazer amanhã: R$ ${diariaBruta.toFixed(2)}`;
  dizimoInfo.innerText = `🙏 Dízimo: R$ ${(diariaBruta*0.1).toFixed(2)}`;

  renderDespesas(diariaLiquida, ganho);
}

function registrar(){
  let m = metas[metaAtual];
  historicoBackup = [...m.historico];
  m.historico.push(Number(ganhoInput.value)*0.9);
  salvar(); atualizar();
}

function desfazer(){ metas[metaAtual].historico=[...historicoBackup]; salvar(); atualizar(); }

function addDespesa(){
  metas[metaAtual].despesas.push({
    nome:nomeDespesa.value,
    valor:Number(valorDespesa.value),
    prioridade: metas[metaAtual].despesas.length,
    porcentagem:null
  });
  salvar(); atualizar();
}

function mover(i,dir){
  let arr = metas[metaAtual].despesas;
  let novo = i+dir;
  if(novo<0||novo>=arr.length) return;
  [arr[i],arr[novo]]=[arr[novo],arr[i]];
  salvar(); atualizar();
}

function atualizarPorcentagem(i,val){
  let m = metas[metaAtual];
  m.despesas[i].porcentagem = Number(val)/100;

  let restante = 1 - m.despesas.reduce((s,d)=>s+(d.porcentagem||0),0);
  let semDef = m.despesas.filter(d=>!d.porcentagem);

  semDef.forEach(d=> d.porcentagem = restante/semDef.length);

  salvar(); atualizar();
}

function removerDespesa(i){ metas[metaAtual].despesas.splice(i,1); salvar(); atualizar(); }

function renderDespesas(diaria, ganho){
  let m = metas[metaAtual];

  // ordena por prioridade (posição)
  let despesas = m.despesas;

  let total = despesas.reduce((s,d)=>s+d.valor,0);

  let html='';

  despesas.forEach((d,i)=>{
    let perc = d.porcentagem ?? (d.valor/total);

    let alocado = ganho * perc;
    let hoje = diaria * perc;

    html+=`
    <div style='margin-bottom:15px;'>
      <b>${d.nome}</b> ( ${(perc*100).toFixed(1)}% )
      <br>
      👉 Hoje: R$ ${hoje.toFixed(2)}
      <br>
      💰 Alocado: R$ ${alocado.toFixed(2)}
      <br>

      %: <input type='number' value='${(perc*100).toFixed(1)}' onchange='atualizarPorcentagem(${i},this.value)'>

      <br>
      <button onclick='mover(${i},-1)'>⬆</button>
      <button onclick='mover(${i},1)'>⬇</button>
      <button class='red' onclick='removerDespesa(${i})'>X</button>
    </div>
    `;
  });

  listaDespesas.innerHTML=html;
}

renderMetas();
</script></body>
</html>
