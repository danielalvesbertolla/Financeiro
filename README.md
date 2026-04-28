<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Meta Financeira PRO MAX</title>
<style>
    :root { --bg: #0f172a; --card: #1e293b; --text: #f8fafc; --green: #22c55e; --blue: #3b82f6; --red: #ef4444; --orange: #f59e0b; --purple: #a855f7; }
    body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); margin: 0; padding: 15px; padding-bottom: 80px; }
    .card { background: var(--card); padding: 18px; border-radius: 16px; margin-bottom: 15px; border: 1px solid #334155; }
    input, select { background: #0f172a; border: 1px solid #475569; color: white; padding: 12px; border-radius: 8px; width: calc(100% - 26px); margin-bottom: 10px; font-size: 16px; }
    button { padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; font-size: 14px; }
    .btn-full { width: 100%; margin-top: 5px; }
    .green { background: var(--green); color: white; }
    .blue { background: var(--blue); color: white; }
    .red { background: var(--red); color: white; }
    .orange { background: var(--orange); color: white; }
    .purple { background: var(--purple); color: white; }
    .gray { background: #64748b; color: white; }
    .flex { display: flex; gap: 8px; align-items: center; justify-content: space-between; }
    .despesa-item { background: #334155; padding: 12px; border-radius: 10px; margin-bottom: 10px; }
    .quitada { opacity: 0.5; background: #1e293b; border: 1px dashed #475569; }
    .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 10px; }
    .stat-box { background: #0f172a; padding: 10px; border-radius: 8px; text-align: center; }
    
    /* Menu Inferior */
    .nav-bar { position: fixed; bottom: 0; left: 0; width: 100%; background: #1e293b; display: flex; justify-content: space-around; padding: 10px 0; border-top: 1px solid #334155; }
    .nav-item { color: #94a3b8; text-decoration: none; font-size: 12px; display: flex; flex-direction: column; align-items: center; background: none; border: none; }
    .nav-item.active { color: var(--blue); }
</style>
</head>
<body>

<div id="configPage" style="display:none;">
    <h2>⚙️ Regras do Sistema</h2>
    <div class="card">
        <label>Custo Fixo Gasolina (R$):</label>
        <input id="cfgGasolina" type="number" onchange="salvarConfig()">
        
        <label>% Dízimo / Reserva Blindada:</label>
        <input id="cfgDizimo" type="number" onchange="salvarConfig()">
        <p><small style="color: #94a3b8;">* Essas regras serão aplicadas em todos os novos lançamentos.</small></p>
    </div>
    <button class="gray btn-full" onclick="showTab('home')">Voltar</button>
</div>

<div id="homePage">
    <h1>📊 Financeiro Pro</h1>
    <div id="listaMetas"></div>
    <div class="card">
        <h3>Nova Meta</h3>
        <input id="nomeMeta" placeholder="Nome">
        <input id="valorMeta" type="number" placeholder="Valor total">
        <input id="diasMeta" type="number" placeholder="Dias">
        <button class="green btn-full" onclick="criarMeta()">Criar</button>
    </div>
</div>

<div id="metaPage" style="display:none;">
    <div class="flex" style="margin-bottom: 10px;">
        <button class="gray" onclick="showTab('home')">⬅</button>
        <button id="btnDesfazer" class="orange" onclick="desfazer()">↩ (0)</button>
    </div>
    
    <div class="card">
        <div class="flex">
            <h2 id="tituloMeta"></h2>
            <button class="gray" onclick="editarNomeMeta()">✏️</button>
        </div>
        <div class="stats-grid">
            <div class="stat-box"><small>Progresso</small><br><b id="percentual">0%</b></div>
            <div class="stat-box"><small>Dias</small><br><b id="diasRest">0</b></div>
        </div>
        <p id="valorAtual" style="font-size: 1.6em; text-align: center; margin: 15px 0; color: var(--green); font-weight: bold;"></p>
        <div class="stat-box" style="background: #334155;">
            <small>Meta Diária Recomendada</small><br><b id="diaria">R$ 0,00</b>
        </div>
    </div>

    <div class="card">
        <h3>💰 Lançar Ganho Bruto</h3>
        <input id="ganhoInput" type="number" placeholder="R$ 0,00">
        <button class="green btn-full" onclick="registrar()">Registrar</button>
    </div>

    <div class="card">
        <h3>📋 Alocação de Contas</h3>
        <div class="flex">
            <input id="nomeDespesa" placeholder="Conta" style="width: 50%;">
            <input id="valorDespesa" type="number" placeholder="R$" style="width: 30%;">
            <button class="blue" onclick="addDespesa()">+</button>
        </div>
        <div id="listaDespesas"></div>
        <h4 style="color: #94a3b8;">✅ Quitadas</h4>
        <div id="listaQuitadas"></div>
    </div>
    <button class="red btn-full" onclick="excluirMeta()">Excluir Meta Permanente</button>
</div>

<nav class="nav-bar">
    <button class="nav-item active" id="nav-home" onclick="showTab('home')">🏠<span>Início</span></button>
    <button class="nav-item" id="nav-config" onclick="showTab('config')">⚙️<span>Regras</span></button>
</nav>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let config = JSON.parse(localStorage.getItem('config_financeiro')) || { gasolina: 75, dizimo: 10 };
let metaAtual = null;
let historicoEstados = [];

// INICIALIZAÇÃO
document.getElementById('cfgGasolina').value = config.gasolina;
document.getElementById('cfgDizimo').value = config.dizimo;

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }
function salvarConfig(){
    config.gasolina = Number(document.getElementById('cfgGasolina').value);
    config.dizimo = Number(document.getElementById('cfgDizimo').value);
    localStorage.setItem('config_financeiro', JSON.stringify(config));
}

function showTab(tab) {
    document.getElementById('homePage').style.display = tab === 'home' ? 'block' : 'none';
    document.getElementById('configPage').style.display = tab === 'config' ? 'block' : 'none';
    document.getElementById('metaPage').style.display = tab === 'meta' ? 'block' : 'none';
    
    document.getElementById('nav-home').classList.toggle('active', tab === 'home');
    document.getElementById('nav-config').classList.toggle('active', tab === 'config');
    renderMetas();
}

function salvarEstado() {
    historicoEstados.push(JSON.stringify(metas));
    if (historicoEstados.length > 15) historicoEstados.shift();
    atualizarBotaoDesfazer();
}

function desfazer() {
    if (historicoEstados.length > 0) {
        metas = JSON.parse(historicoEstados.pop());
        salvar(); atualizar(); atualizarBotaoDesfazer();
    }
}

function atualizarBotaoDesfazer() {
    const btn = document.getElementById('btnDesfazer');
    btn.style.display = historicoEstados.length > 0 ? 'block' : 'none';
    btn.innerText = `↩ (${historicoEstados.length})`;
}

function criarMeta(){
    salvarEstado();
    const nome = document.getElementById('nomeMeta').value;
    const valor = document.getElementById('valorMeta').value;
    if(!nome || !valor) return;
    metas.push({ nome, meta: Number(valor), diasTotal: Number(document.getElementById('diasMeta').value) || 30, historico: [], despesas: [] });
    salvar(); renderMetas();
    document.getElementById('nomeMeta').value = ''; document.getElementById('valorMeta').value = '';
}

function abrirMeta(i){
    metaAtual = i;
    showTab('meta');
    atualizar();
}

function registrar(){
    salvarEstado();
    let bruto = Number(document.getElementById('ganhoInput').value);
    if(bruto <= 0) return;
    let liquido = Math.max(0, bruto - config.gasolina) * (1 - (config.dizimo / 100));
    metas[metaAtual].historico.push(liquido);
    document.getElementById('ganhoInput').value = '';
    salvar(); atualizar();
}

function atualizar(){
    let m = metas[metaAtual];
    let acumulado = m.historico.reduce((s, v) => s + v, 0);
    let restante = m.meta - acumulado;
    let diasRest = Math.max(0, m.diasTotal - m.historico.length);
    
    document.getElementById('tituloMeta').innerText = m.nome;
    document.getElementById('percentual').innerText = ((acumulado/m.meta)*100).toFixed(1) + '%';
    document.getElementById('diasRest').innerText = diasRest;
    document.getElementById('valorAtual').innerText = `R$ ${acumulado.toFixed(2)}`;
    document.getElementById('diaria').innerText = `R$ ${(restante / (diasRest || 1)).toFixed(2)}`;
    renderDespesas(acumulado);
}

function addDespesa(){
    salvarEstado();
    let n = document.getElementById('nomeDespesa');
    let v = document.getElementById('valorDespesa');
    metas[metaAtual].despesas.push({ nome: n.value, totalMeta: Number(v.value), percentual: 0, manual: false, quitada: false });
    n.value = ''; v.value = '';
    recalcularPorcentagens();
}

function recalcularPorcentagens(){
    let m = metas[metaAtual];
    let ativas = m.despesas.filter(d => !d.quitada);
    let manuais = ativas.filter(d => d.manual);
    let autos = ativas.filter(d => !d.manual);
    let somaManual = manuais.reduce((s, d) => s + d.percentual, 0);
    let fatia = Math.max(0, 1 - somaManual) / (autos.length || 1);
    autos.forEach(d => d.percentual = fatia);
    salvar(); atualizar();
}

function renderMetas(){
    let html = '';
    metas.forEach((m, i) => {
        html += `<div class="card flex"><span><b>${m.nome}</b><br>R$ ${m.meta}</span><button class="blue" onclick="abrirMeta(${i})">Abrir</button></div>`;
    });
    document.getElementById('listaMetas').innerHTML = html;
}

function renderDespesas(ganhoTotal){
    let m = metas[metaAtual];
    let h1 = ''; let h2 = '';
    m.despesas.forEach((d, i) => {
        let item = `<div class="despesa-item ${d.quitada ? 'quitada' : ''}">
            <div class="flex"><b>${d.nome}</b> <small>${(d.percentual*100).toFixed(0)}%</small></div>
            <small>Saldo: R$ ${(ganhoTotal * d.percentual).toFixed(2)}</small>
            ${!d.quitada ? `<input type="range" value="${d.percentual*100}" onchange="salvarEstado(); metas[metaAtual].despesas[${i}].percentual=this.value/100; metas[metaAtual].despesas[${i}].manual=true; recalcularPorcentagens()">
            <div class="flex"><button class="green" onclick="quitarConta(${i})">Quitar</button><button class="red" onclick="removerDespesa(${i})">🗑</button></div>` : ''}
        </div>`;
        if(d.quitada) h2 += item; else h1 += item;
    });
    document.getElementById('listaDespesas').innerHTML = h1;
    document.getElementById('listaQuitadas').innerHTML = h2;
}

function quitarConta(i){ salvarEstado(); metas[metaAtual].despesas[i].quitada = true; metas[metaAtual].despesas[i].percentual = 0; recalcularPorcentagens(); }
function removerDespesa(i){ salvarEstado(); metas[metaAtual].despesas.splice(i, 1); recalcularPorcentagens(); }
function excluirMeta(){ if(confirm("Apagar meta?")){ metas.splice(metaAtual, 1); salvar(); showTab('home'); } }
function editarNomeMeta(){ let n = prompt("Novo nome:", metas[metaAtual].nome); if(n){ metas[metaAtual].nome = n; salvar(); atualizar(); } }

renderMetas();
</script>
</body>
</html>
