<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Meta Financeira PRO MAX - Full Edition</title>
<style>
    :root { 
        --bg: #0f172a; --card: #1e293b; --text: #f8fafc; 
        --green: #22c55e; --blue: #3b82f6; --red: #ef4444; 
        --orange: #f59e0b; --purple: #a855f7; --slate: #475569;
    }
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: var(--bg); color: var(--text); margin: 0; padding: 15px; padding-bottom: 90px; line-height: 1.4; }
    .card { background: var(--card); padding: 18px; border-radius: 16px; margin-bottom: 15px; border: 1px solid #334155; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.3); }
    
    h1, h2, h3, h4 { margin: 0 0 10px 0; }
    input, select { background: #0f172a; border: 1px solid var(--slate); color: white; padding: 12px; border-radius: 8px; width: calc(100% - 26px); margin-bottom: 10px; font-size: 16px; }
    
    button { padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; font-size: 14px; transition: 0.2s; display: inline-flex; align-items: center; justify-content: center; gap: 5px; }
    button:active { transform: scale(0.98); opacity: 0.8; }
    
    .btn-full { width: 100%; margin-top: 5px; }
    .green { background: var(--green); color: white; }
    .blue { background: var(--blue); color: white; }
    .red { background: var(--red); color: white; }
    .orange { background: var(--orange); color: white; }
    .gray { background: #64748b; color: white; }
    
    .flex { display: flex; gap: 8px; align-items: center; justify-content: space-between; }
    .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin: 10px 0; }
    .stat-box { background: #0f172a; padding: 12px; border-radius: 10px; text-align: center; border: 1px solid #334155; }
    
    .despesa-item { background: #334155; padding: 15px; border-radius: 12px; margin-bottom: 12px; border-left: 5px solid var(--blue); }
    .manual-mode { border-left-color: var(--orange); }
    .quitada { opacity: 0.5; filter: grayscale(0.8); border-left-color: var(--green); text-decoration: line-through; }
    
    .badge { font-size: 10px; padding: 2px 6px; border-radius: 4px; text-transform: uppercase; }
    .badge-auto { background: var(--blue); }
    .badge-manual { background: var(--orange); }

    input[type="range"] { width: 100%; margin: 15px 0; accent-color: var(--blue); }

    /* Barra de Navegação */
    .nav-bar { position: fixed; bottom: 0; left: 0; width: 100%; background: #1e293b; display: flex; justify-content: space-around; padding: 10px 0; border-top: 2px solid #334155; z-index: 100; }
    .nav-item { color: #94a3b8; background: none; border: none; display: flex; flex-direction: column; align-items: center; font-size: 11px; cursor: pointer; }
    .nav-item.active { color: var(--blue); }
    .nav-item i { font-size: 20px; margin-bottom: 4px; }

    .historico-lista { max-height: 200px; overflow-y: auto; font-size: 13px; }
    .historico-item { display: flex; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #334155; }
</style>
</head>
<body>

<div id="configPage" style="display:none;">
    <h1>⚙️ Regras do Sistema</h1>
    <div class="card">
        <h3>Ajustes Globais</h3>
        <label>Custo Fixo Combustível (R$):</label>
        <input id="cfgGasolina" type="number" step="0.01">
        
        <label>% Dízimo / Reserva (Blindada):</label>
        <input id="cfgDizimo" type="number" step="0.1">
        
        <button class="green btn-full" onclick="salvarConfigGlobais()">Salvar Configurações</button>
    </div>
    <div class="card">
        <h3>Sobre o Sistema</h3>
        <p><small>Este código calcula o líquido real subtraindo primeiro o combustível e depois a porcentagem de reserva configurada acima.</small></p>
    </div>
</div>

<div id="homePage">
    <h1>📊 Minhas Metas</h1>
    <div id="listaMetas"></div>
    
    <div class="card">
        <h3>✨ Criar Nova Meta</h3>
        <input id="nomeMeta" placeholder="Nome da Meta (ex: Aluguel)">
        <input id="valorMeta" type="number" placeholder="Valor (Opcional - Soma das Contas)">
        <input id="diasMeta" type="number" placeholder="Prazo em Dias">
        <button class="blue btn-full" onclick="criarMeta()">Criar Meta Agora</button>
    </div>
</div>

<div id="metaPage" style="display:none;">
    <div class="flex" style="margin-bottom: 15px;">
        <button class="gray" onclick="navegar('home')">⬅ Voltar</button>
        <button id="btnDesfazer" class="orange" onclick="executarDesfazer()" style="display:none;">↩ Desfazer (0)</button>
    </div>

    <div class="card">
        <div class="flex">
            <h2 id="tituloMeta" style="margin:0;"></h2>
            <button class="gray" style="padding:5px 10px;" onclick="editarMetaInfo()">✏️</button>
        </div>
        <div class="stats-grid">
            <div class="stat-box"><small>Progresso</small><br><b id="txtProgresso" style="color:var(--blue)">0%</b></div>
            <div class="stat-box"><small>Dias Restantes</small><br><b id="txtDias">0</b></div>
        </div>
        <div class="stat-box" style="margin-bottom:10px;">
            <small>Acumulado Líquido</small><br>
            <span id="txtAcumulado" style="font-size: 1.8em; font-weight: bold; color: var(--green);">R$ 0,00</span>
        </div>
        <div class="flex">
            <div class="stat-box" style="width: 100%;">
                <small>Meta Diária (Líquida)</small><br>
                <b id="txtDiaria">R$ 0,00</b>
            </div>
        </div>
    </div>

    <div class="card">
        <h3>💰 Lançar Ganho Bruto</h3>
        <div class="flex">
            <input id="ganhoInput" type="number" placeholder="R$ 0,00" style="margin:0; flex-grow:1;">
            <button class="green" onclick="registrarGanho()">Lançar</button>
        </div>
        <p style="font-size: 11px; color: #94a3b8; margin-top: 8px;">
            * Descontando R$ <span id="labelGas"></span> de gasolina e <span id="labelDiz"></span>% de dízimo.
        </p>
    </div>

    <div class="card">
        <h3>📋 Alocação (Bola de Neve)</h3>
        <div class="flex">
            <input id="nomeDesp" placeholder="Conta" style="width:45%; margin:0;">
            <input id="valorDesp" type="number" placeholder="Total" style="width:30%; margin:0;">
            <button class="blue" onclick="adicionarDespesa()">+</button>
        </div>
        <div id="listaDespesasAtivas" style="margin-top:15px;"></div>
        
        <h4 style="margin-top: 20px; color: var(--green); border-bottom: 1px solid #334155;">✅ Quitadas</h4>
        <div id="listaDespesasQuitadas"></div>
    </div>

    <div class="card">
        <h3>📜 Histórico Recente</h3>
        <div id="historicoGanhos" class="historico-lista"></div>
    </div>

    <button class="red btn-full" onclick="excluirMetaTotal()" style="margin-top: 20px;">🗑️ Excluir Meta Permanente</button>
</div>

<nav class="nav-bar">
    <div class="nav-item active" id="nav-home" onclick="navegar('home')">🏠<span>Metas</span></div>
    <div class="nav-item" id="nav-config" onclick="navegar('config')">⚙️<span>Regras</span></div>
</nav>

<script>
/* =========================================
SISTEMA DE DADOS E MEMÓRIA (UNDO)
=========================================
*/
let metas = JSON.parse(localStorage.getItem('metas_v3')) || [];
let config = JSON.parse(localStorage.getItem('config_v3')) || { gasolina: 75, dizimo: 10 };
let metaAtivaIndex = null;
let pilhaUndo = [];

function gravarDados() {
    localStorage.setItem('metas_v3', JSON.stringify(metas));
    localStorage.setItem('config_v3', JSON.stringify(config));
}

function snapshot() {
    pilhaUndo.push(JSON.stringify(metas));
    if (pilhaUndo.length > 15) pilhaUndo.shift();
    atualizarUIUndo();
}

function executarDesfazer() {
    if (pilhaUndo.length > 0) {
        metas = JSON.parse(pilhaUndo.pop());
        gravarDados();
        if (metaAtivaIndex !== null) renderizarMetaDetalhe();
        renderizarHome();
        atualizarUIUndo();
    }
}

function atualizarUIUndo() {
    const btn = document.getElementById('btnDesfazer');
    if (btn) {
        btn.style.display = pilhaUndo.length > 0 ? 'block' : 'none';
        btn.innerText = `↩ Desfazer (${pilhaUndo.length})`;
    }
}

/* =========================================
NAVEGAÇÃO
=========================================
*/
function navegar(aba) {
    document.getElementById('homePage').style.display = aba === 'home' ? 'block' : 'none';
    document.getElementById('configPage').style.display = aba === 'config' ? 'block' : 'none';
    document.getElementById('metaPage').style.display = aba === 'meta' ? 'block' : 'none';
    
    document.getElementById('nav-home').classList.toggle('active', aba === 'home');
    document.getElementById('nav-config').classList.toggle('active', aba === 'config');
    
    if (aba === 'home') {
        metaAtivaIndex = null;
        renderizarHome();
    }
    window.scrollTo(0,0);
}

/* =========================================
LÓGICA DE CONFIGURAÇÕES
=========================================
*/
document.getElementById('cfgGasolina').value = config.gasolina;
document.getElementById('cfgDizimo').value = config.dizimo;

function salvarConfigGlobais() {
    config.gasolina = Number(document.getElementById('cfgGasolina').value);
    config.dizimo = Number(document.getElementById('cfgDizimo').value);
    gravarDados();
    alert("Regras atualizadas com sucesso!");
    navegar('home');
}

/* =========================================
LÓGICA DE METAS (HOME)
=========================================
*/
function criarMeta() {
    snapshot();
    const nome = document.getElementById('nomeMeta').value;
    const valor = document.getElementById('valorMeta').value;
    const dias = document.getElementById('diasMeta').value;
    
    // MODIFICADO: Agora só exige o nome obrigatório. O valor pode ficar vazio pois soma automático depois.
    if (!nome) return alert("Preencha o Nome da Meta");
    
    metas.push({
        nome: nome,
        alvo: Number(valor) || 0,
        prazo: Number(dias) || 30,
        ganhosLíquidos: [],
        contas: []
    });
    
    gravarDados();
    renderizarHome();
    document.getElementById('nomeMeta').value = '';
    document.getElementById('valorMeta').value = '';
    document.getElementById('diasMeta').value = '';
}

function renderizarHome() {
    let html = '';
    metas.forEach((m, i) => {
        const total = m.ganhosLíquidos.reduce((s, v) => s + v, 0);
        // MODIFICADO: Proteção matemática para não mostrar "NaN" se o alvo for zero
        const perc = m.alvo > 0 ? ((total / m.alvo) * 100).toFixed(0) : 0; 
        html += `
            <div class="card flex" onclick="abrirMeta(${i})" style="cursor:pointer">
                <div>
                    <b>${m.nome}</b><br>
                    <small>R$ ${total.toFixed(2)} / R$ ${m.alvo.toFixed(2)}</small>
                </div>
                <div style="text-align:right">
                    <b style="color:var(--blue)">${perc}%</b><br>
                    <small>Abrir ⮕</small>
                </div>
            </div>`;
    });
    document.getElementById('listaMetas').innerHTML = html || '<p style="text-align:center">Nenhuma meta criada.</p>';
}

/* =========================================
DETALHE DA META E CÁLCULOS
=========================================
*/
function abrirMeta(i) {
    metaAtivaIndex = i;
    navegar('meta');
    renderizarMetaDetalhe();
}

function registrarGanho() {
    snapshot();
    const input = document.getElementById('ganhoInput');
    const bruto = Number(input.value);
    if (bruto <= 0) return;

    // A REGRA DE OURO: Gasolina primeiro, depois dízimo
    const aposGasolina = Math.max(0, bruto - config.gasolina);
    const liquidoFinal = aposGasolina * (1 - (config.dizimo / 100));

    metas[metaAtivaIndex].ganhosLíquidos.push(liquidoFinal);
    input.value = '';
    gravarDados();
    renderizarMetaDetalhe();
}

function renderizarMetaDetalhe() {
    const m = metas[metaAtivaIndex];
    
    // MODIFICADO: AQUI ENTRA A SUA NOVA LÓGICA
    // Soma o valor de todas as contas para definir o alvo total da meta
    m.alvo = m.contas.reduce((s, c) => s + c.metaValor, 0); 

    const totalLíquido = m.ganhosLíquidos.reduce((s, v) => s + v, 0);
    const diasRestantes = Math.max(0, m.prazo - m.ganhosLíquidos.length);
    const progresso = m.alvo > 0 ? (totalLíquido / m.alvo) * 100 : 0;
    const falta = Math.max(0, m.alvo - totalLíquido);
    const sugerido = falta / (diasRestantes || 1);

    document.getElementById('tituloMeta').innerText = m.nome;
    document.getElementById('txtProgresso').innerText = progresso.toFixed(1) + '%';
    document.getElementById('txtDias').innerText = diasRestantes + ' d';
    document.getElementById('txtAcumulado').innerText = `R$ ${totalLíquido.toFixed(2)}`;
    document.getElementById('txtDiaria').innerText = `R$ ${sugerido.toFixed(2)}`;
    
    document.getElementById('labelGas').innerText = config.gasolina;
    document.getElementById('labelDiz').innerText = config.dizimo;

    renderizarContas(totalLíquido);
    renderizarHistoricoMeta();
}

/* =========================================
CONTAS E REDISTRIBUIÇÃO AUTOMÁTICA (%)
=========================================
*/
function adicionarDespesa() {
    snapshot();
    const n = document.getElementById('nomeDesp');
    const v = document.getElementById('valorDesp');
    if (!n.value) return;

    metas[metaAtivaIndex].contas.push({
        nome: n.value,
        metaValor: Number(v.value) || 0,
        percentual: 0,
        manual: false,
        quitada: false
    });
    
    n.value = ''; v.value = '';
    recalcularProporcoes();
}

function recalcularProporcoes() {
    const m = metas[metaAtivaIndex];
    const ativas = m.contas.filter(c => !c.quitada);
    const manuais = ativas.filter(c => c.manual);
    const autos = ativas.filter(c => !c.manual);

    let somaManual = manuais.reduce((s, c) => s + c.percentual, 0);
    let sobra = Math.max(0, 1 - somaManual);

    if (autos.length > 0) {
        const fatia = sobra / autos.length;
        autos.forEach(c => c.percentual = fatia);
    }
    
    gravarDados();
    renderizarMetaDetalhe();
}

function mudarPercentualManual(contaIdx, novoValor) {
    snapshot();
    const m = metas[metaAtivaIndex];
    m.contas[contaIdx].percentual = Number(novoValor) / 100;
    m.contas[contaIdx].manual = true;
    recalcularProporcoes();
}

function toggleModo(contaIdx) {
    snapshot();
    const m = metas[metaAtivaIndex];
    m.contas[contaIdx].manual = !m.contas[contaIdx].manual;
    recalcularProporcoes();
}

function quitarConta(contaIdx) {
    snapshot();
    const m = metas[metaAtivaIndex];
    m.contas[contaIdx].quitada = true;
    m.contas[contaIdx].percentual = 0;
    m.contas[contaIdx].manual = false;
    recalcularProporcoes();
}

function removerConta(contaIdx) {
    snapshot();
    metas[metaAtivaIndex].contas.splice(contaIdx, 1);
    recalcularProporcoes();
}

function renderizarContas(totalLíquido) {
    const m = metas[metaAtivaIndex];
    let htmlAtivas = '';
    let htmlQuitadas = '';

    m.contas.forEach((c, i) => {
        const saldoConta = totalLíquido * c.percentual;
        const item = `
            <div class="despesa-item ${c.manual ? 'manual-mode' : ''} ${c.quitada ? 'quitada' : ''}">
                <div class="flex">
                    <b>${c.nome}</b>
                    <span class="badge ${c.manual ? 'badge-manual' : 'badge-auto'}">
                        ${(c.percentual*100).toFixed(0)}% ${c.manual ? 'Manual' : 'Auto'}
                    </span>
                </div>
                <div class="flex" style="margin: 8px 0;">
                    <small>Meta: R$ ${c.metaValor.toFixed(2)}</small>
                    <b style="color:var(--green)">R$ ${saldoConta.toFixed(2)}</b>
                </div>
                
                ${!c.quitada ? `
                    <input type="range" min="0" max="100" value="${(c.percentual*100).toFixed(0)}" 
                           onchange="mudarPercentualManual(${i}, this.value)">
                    <div class="flex">
                        <button class="gray" onclick="toggleModo(${i})">⚙️ Modo</button>
                        
