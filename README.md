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

    .nav-bar { position: fixed; bottom: 0; left: 0; width: 100%; background: #1e293b; display: flex; justify-content: space-around; padding: 10px 0; border-top: 2px solid #334155; z-index: 100; }
    .nav-item { color: #94a3b8; background: none; border: none; display: flex; flex-direction: column; align-items: center; font-size: 11px; cursor: pointer; }
    .nav-item.active { color: var(--blue); }

    .historico-lista { max-height: 200px; overflow-y: auto; font-size: 13px; }
    .historico-item { display: flex; justify-content: space-between; padding: 5px 0; border-bottom: 1px solid #334155; }
</style>

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
</div>

<div id="homePage">
    <h1>📊 Minhas Metas</h1>
    <div id="listaMetas"></div>
    
    <div class="card">
        <h3>✨ Criar Nova Meta</h3>
        <input id="nomeMeta" placeholder="Nome da Meta (ex: Aluguel)">
        <input id="valorMeta" type="number" placeholder="Valor (Soma automática depois)" disabled style="opacity: 0.5;">
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
        <h3>📋 Alocação de Contas (Soma = Total da Meta)</h3>
        <div class="flex">
            <input id="nomeDesp" placeholder="Conta" style="width:45%; margin:0;">
            <input id="valorDesp" type="number" placeholder="Valor" style="width:30%; margin:0;">
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
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let config = JSON.parse(localStorage.getItem('config_financeiro')) || { gasolina: 75, dizimo: 10 };
let metaAtual = null;
let historicoEstados = [];

function gravarDados() {
    localStorage.setItem('metas', JSON.stringify(metas));
    localStorage.setItem('config_financeiro', JSON.stringify(config));
}

function snapshot() {
    historicoEstados.push(JSON.stringify(metas));
    if (historicoEstados.length > 15) historicoEstados.shift();
    atualizarUIUndo();
}

function executarDesfazer() {
    if (historicoEstados.length > 0) {
        metas = JSON.parse(historicoEstados.pop());
        gravarDados();
        if (metaAtual !== null) renderizarMetaDetalhe();
        renderizarHome();
        atualizarUIUndo();
    }
}

function atualizarUIUndo() {
    const btn = document.getElementById('btnDesfazer');
    if (btn) {
        btn.style.display = historicoEstados.length > 0 ? 'block' : 'none';
        btn.innerText = `↩ Desfazer (${historicoEstados.length})`;
    }
}

function navegar(aba) {
    document.getElementById('homePage').style.display = aba === 'home' ? 'block' : 'none';
    document.getElementById('configPage').style.display = aba === 'config' ? 'block' : 'none';
    document.getElementById('metaPage').style.display = aba === 'meta' ? 'block' : 'none';
    
    document.getElementById('nav-home').classList.toggle('active', aba === 'home');
    document.getElementById('nav-config').classList.toggle('active', aba === 'config');
    
    if (aba === 'home') {
        metaAtual = null;
        renderizarHome();
    }
    window.scrollTo(0,0);
}

document.getElementById('cfgGasolina').value = config.gasolina;
document.getElementById('cfgDizimo').value = config.dizimo;

function salvarConfigGlobais() {
    config.gasolina = Number(document.getElementById('cfgGasolina').value);
    config.dizimo = Number(document.getElementById('cfgDizimo').value);
    gravarDados();
    alert("Regras atualizadas!");
    navegar('home');
}

function criarMeta() {
    snapshot();
    const nome = document.getElementById('nomeMeta').value;
    const dias = document.getElementById('diasMeta').value;
    
    if (!nome) return alert("Preencha o Nome da Meta");
    
    metas.push({
        nome: nome,
        meta: 0, // Inicia zerado, pois vai somar as despesas depois
        diasTotal: Number(dias) || 30,
        historico: [],
        despesas: []
    });
    
    gravarDados();
    renderizarHome();
    document.getElementById('nomeMeta').value = '';
    document.getElementById('diasMeta').value = '';
}

function renderizarHome() {
    let html = '';
    metas.forEach((m, i) => {
        if (!m.historico) m.historico = [];
        if (!m.despesas) m.despesas = [];
        
        const total = m.historico.reduce((s, v) => s + v, 0);
        const perc = m.meta > 0 ? ((total / m.meta) * 100).toFixed(0) : 0; 
        html += `
            <div class="card flex" onclick="abrirMeta(${i})" style="cursor:pointer">
                <div>
                    <b>${m.nome}</b><br>
                    <small>R$ ${total.toFixed(2)} / R$ ${(m.meta || 0).toFixed(2)}</small>
                </div>
                <div style="text-align:right">
                    <b style="color:var(--blue)">${perc}%</b><br>
                    <small>Abrir ⮕</small>
                </div>
            </div>`;
    });
    document.getElementById('listaMetas').innerHTML = html || '<p style="text-align:center">Nenhuma meta criada.</p>';
}

function abrirMeta(i) {
    metaAtual = i;
    navegar('meta');
    renderizarMetaDetalhe();
}

function registrarGanho() {
    snapshot();
    const input = document.getElementById('ganhoInput');
    const bruto = Number(input.value);
    if (bruto <= 0) return;

    const aposGasolina = Math.max(0, bruto - config.gasolina);
    const liquidoFinal = aposGasolina * (1 - (config.dizimo / 100));

    metas[metaAtual].historico.push(liquidoFinal);
    input.value = '';
    gravarDados();
    renderizarMetaDetalhe();
}

function renderizarMetaDetalhe() {
    const m = metas[metaAtual];
    
    if (!m.historico) m.historico = [];
    if (!m.despesas) m.despesas = [];

    // O VALOR DA META AGORA É A SOMA DAS DESPESAS
    m.meta = m.despesas.reduce((s, d) => s + (d.totalMeta || 0), 0); 

    const totalLíquido = m.historico.reduce((s, v) => s + v, 0);
    const diasRestantes = Math.max(0, m.diasTotal - m.historico.length);
    const progresso = m.meta > 0 ? (totalLíquido / m.meta) * 100 : 0;
    const falta = Math.max(0, m.meta - totalLíquido);
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

function adicionarDespesa() {
    snapshot();
    const n = document.getElementById('nomeDesp');
    const v = document.getElementById('valorDesp');
    if (!n.value || !v.value) return alert("Preencha nome e valor da conta");

    metas[metaAtual].despesas.push({
        nome: n.value,
        totalMeta: Number(v.value) || 0,
        percentual: 0,
        manual: false,
        quitada: false
    });
    
    n.value = ''; v.value = '';
    recalcularProporcoes();
}

function recalcularProporcoes() {
    const m = metas[metaAtual];
    const ativas = m.despesas.filter(d => !d.quitada);
    const manuais = ativas.filter(d => d.manual);
    const autos = ativas.filter(d => !d.manual);

    let somaManual = manuais.reduce((s, d) => s + d.percentual, 0);
    let sobra = Math.max(0, 1 - somaManual);

    if (autos.length > 0) {
        const fatia = sobra / autos.length;
        autos.forEach(d => d.percentual = fatia);
    }
    
    gravarDados();
    renderizarMetaDetalhe();
}

function mudarPercentualManual(contaIdx, novoValor) {
    snapshot();
    const m = metas[metaAtual];
    m.despesas[contaIdx].percentual = Number(novoValor) / 100;
    m.despesas[contaIdx].manual = true;
    recalcularProporcoes();
}

function toggleModo(contaIdx) {
    snapshot();
    const m = metas[metaAtual];
    m.despesas[contaIdx].manual = !m.despesas[contaIdx].manual;
    recalcularProporcoes();
}

function quitarConta(contaIdx) {
    snapshot();
    const m = metas[metaAtual];
    m.despesas[contaIdx].quitada = true;
    m.despesas[contaIdx].percentual = 0;
    m.despesas[contaIdx].manual = false;
    recalcularProporcoes();
}

function removerConta(contaIdx) {
    snapshot();
    metas[metaAtual].despesas.splice(contaIdx, 1);
    recalcularProporcoes();
}

function renderizarContas(totalLíquido) {
    const m = metas[metaAtual];
    let htmlAtivas = '';
    let htmlQuitadas = '';

    m.despesas.forEach((d, i) => {
        const saldoConta = totalLíquido * d.percentual;
        const item = `
            <div class="despesa-item ${d.manual ? 'manual-mode' : ''} ${d.quitada ? 'quitada' : ''}">
                <div class="flex">
                    <b>${d.nome}</b>
                    <span class="badge ${d.manual ? 'badge-manual' : 'badge-auto'}">
                        ${(d.percentual*100).toFixed(0)}% ${d.manual ? 'Manual' : 'Auto'}
                    </span>
                </div>
                <div class="flex" style="margin: 8px 0;">
                    <small>Meta: R$ ${(d.totalMeta || 0).toFixed(2)}</small>
                    <b style="color:var(--green)">R$ ${saldoConta.toFixed(2)}</b>
                </div>
                
                ${!d.quitada ? `
                    <input type="range" min="0" max="100" value="${(d.percentual*100).toFixed(0)}" 
                           onchange="mudarPercentualManual(${i}, this.value)">
                    <div class="flex">
                        <button class="gray" onclick="toggleModo(${i})">⚙️ Modo</button>
                        <button class="green" onclick="quitarConta(${i})">✔️ Quitar</button>
                        <button class="red" onclick="removerConta(${i})">🗑️</button>
                    </div>
                ` : `<small>CONTA PAGA - Saldo redistribuído</small>`}
            </div>
        `;
        if (d.quitada) htmlQuitadas += item; else htmlAtivas += item;
    });

    document.getElementById('listaDespesasAtivas').innerHTML = htmlAtivas || '<p><small>Nenhuma conta adicionada.</small></p>';
    document.getElementById('listaDespesasQuitadas').innerHTML = htmlQuitadas || '<p>-</p>';
}

function renderizarHistoricoMeta() {
    const m = metas[metaAtual];
    let html = '';
    [...m.historico].reverse().forEach((v, i) => {
        html += `
            <div class="historico-item">
                <span>Lançamento ${m.historico.length - i}</span>
                <b>R$ ${v.toFixed(2)}</b>
            </div>`;
    });
    document.getElementById('historicoGanhos').innerHTML = html || 'Sem histórico.';
}

function editarMetaInfo() {
    snapshot();
    const m = metas[metaAtual];
    const novoNome = prompt("Novo nome:", m.nome);
    const novoPrazo = prompt("Novo prazo (dias):", m.diasTotal);
    
    if (novoNome) m.nome = novoNome;
    if (novoPrazo) m.diasTotal = Number(novoPrazo);
    
    gravarDados();
    renderizarMetaDetalhe();
}

function excluirMetaTotal() {
    if (confirm("ATENÇÃO: Você vai apagar todos os dados desta meta permanentemente. Confirma?")) {
        snapshot();
        metas.splice(metaAtual, 1);
        gravarDados();
        navegar('home');
    }
}

// Inicia o app
renderizarHome();
</script>
