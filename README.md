<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financeiro Master PRO - Tudo em Um</title>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; --border: #334155; --green: #22c55e; --red: #ef4444; --blue: #3b82f6; --orange: #f59e0b; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: white; margin: 0; padding: 10px; }
        .container { max-width: 800px; margin: auto; padding-bottom: 50px; }
        .card { background: var(--card); padding: 20px; border-radius: 12px; margin-bottom: 20px; border: 1px solid var(--border); box-shadow: 0 4px 15px rgba(0,0,0,0.4); }
        h1, h2, h3 { margin-top: 0; color: #f8fafc; }
        button { padding: 12px 16px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.2s; margin: 4px 2px; }
        button:hover { filter: brightness(1.2); transform: translateY(-1px); }
        input { padding: 12px; border-radius: 8px; border: 1px solid var(--border); background: #0f172a; color: white; margin: 5px 0; }
        .green { background: var(--green); } .red { background: var(--red); } .blue { background: var(--blue); } .orange { background: var(--orange); } .gray { background: #64748b; }
        
        /* Dashboard Layout */
        .grid-layout { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        @media (max-width: 768px) { .grid-layout { grid-template-columns: 1fr; } }

        .finance-box { background: rgba(0,0,0,0.3); padding: 15px; border-radius: 8px; border: 1px solid var(--blue); margin-bottom: 15px; }
        .row { display: flex; justify-content: space-between; padding: 10px 0; border-bottom: 1px dashed var(--border); }
        .row:last-child { border-bottom: none; font-size: 1.2em; font-weight: bold; }

        .item-edit { background: rgba(255,255,255,0.03); padding: 15px; border-radius: 10px; margin-bottom: 10px; border: 1px solid var(--border); }
        .quitada-item { background: #064e3b; border-left: 6px solid var(--green); padding: 12px; border-radius: 8px; margin-bottom: 8px; opacity: 0.9; }
        
        .section-title { border-left: 4px solid var(--blue); padding-left: 10px; margin: 25px 0 15px 0; display: flex; justify-content: space-between; align-items: center; }
    </style>
</head>
<body>

<div class="container">
    
    <div id="projectSelector" class="card">
        <h1>📊 Meus Projetos</h1>
        <div id="listaProjetos"></div>
        <hr style="border: 0; border-top: 1px solid var(--border); margin: 20px 0;">
        <div style="display: flex; gap: 10px; flex-wrap: wrap;">
            <input id="p_nome" placeholder="Nome do Projeto (Ex: Maio 2026)" style="flex: 2;">
            <input id="p_dias" type="number" placeholder="Dias" style="flex: 0.5;">
            <button class="green" onclick="criarProjeto()" style="flex: 1;">Criar Novo</button>
        </div>
    </div>

    <div id="mainDashboard" style="display: none;">
        
        <div class="section-title">
            <h2 id="displayNome" style="margin:0"></h2>
            <button class="gray" onclick="fecharProjeto()">Voltar à Lista</button>
        </div>

        <div class="grid-layout">
            <div>
                <div class="card">
                    <h3>🚀 Painel de Metas</h3>
                    <div class="finance-box">
                        <div class="row"><span>🎯 Meta Total (Dívidas):</span> <span id="metaTotalDisplay"></span></div>
                        <div class="row"><span>⛽ Combustível:</span> <span>R$ 75,00</span></div>
                        <div class="row"><span>🙏 Dízimo Previsto (10%):</span> <span id="metaDizimo" style="color:var(--orange)"></span></div>
                        <div class="row"><span>💰 BRUTO NECESSÁRIO HOJE:</span> <span id="metaBruta" style="color:var(--blue)"></span></div>
                    </div>
                    
                    <div style="background: rgba(255,255,255,0.05); padding: 15px; border-radius: 10px; border: 1px solid var(--border);">
                        <strong>Registrar Ganho Bruto REAL:</strong>
                        <input id="inputBruto" type="number" placeholder="R$ 0,00" style="width: 100%; box-sizing: border-box; font-size: 1.2em;">
                        <button class="green" onclick="registrarGanho()" style="width: 100%; margin-top: 10px;">Confirmar Entrada</button>
                    </div>
                </div>

                <div class="card">
                    <h3>📜 Histórico Recente</h3>
                    <div id="listaHistorico"></div>
                </div>
            </div>

            <div>
                <div class="card">
                    <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
                        <h3 style="margin:0">⚙️ Gerenciar Contas</h3>
                        <button class="blue" onclick="desfazer()" style="font-size: 12px; padding: 5px 10px;">↩ Desfazer</button>
                    </div>
                    
                    <div style="display:flex; gap:5px; margin-bottom: 15px;">
                        <button class="orange" onclick="bolaDeNeve()" style="flex:1; font-size: 13px;">🚀 Bola de Neve (Priorizar Menores)</button>
                    </div>

                    <div id="editorContas"></div>
                    
                    <hr style="border: 0; border-top: 1px solid var(--border); margin: 15px 0;">
                    <h4>Nova Conta</h4>
                    <div style="display:flex; gap:5px;">
                        <input id="c_nome" placeholder="Nome" style="flex:2;">
                        <input id="c_valor" type="number" placeholder="R$" style="flex:1;">
                        <button class="blue" onclick="addConta()">+</button>
                    </div>
                </div>

                <div class="card">
                    <h3>💰 Divisão do Líquido de Hoje</h3>
                    <div id="divisaoHoje"></div>
                </div>
            </div>
        </div>

        <div class="card">
            <h3>✅ Contas Quitadas (Fora da Divisão)</h3>
            <div id="listaQuitadas"></div>
        </div>

        <button class="red" onclick="excluirProjetoTotal()" style="width: 100%; opacity: 0.4; margin-top: 20px;">🗑 Excluir Projeto Inteiro</button>
    </div>
</div>

<script>
let dados = JSON.parse(localStorage.getItem('financas_final_v5')) || [];
let atual = null;
let bkpEstrutura = null;
const GASOLINA = 75;

function salvar() { localStorage.setItem('financas_final_v5', JSON.stringify(dados)); }

// --- GERENCIAMENTO DE PROJETOS ---
function renderHome() {
    let h = '';
    dados.forEach((p, i) => {
        let metaTotal = p.contas.reduce((s, c) => s + c.valor, 0);
        h += `<div class="card" style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;">
            <span><b>${p.nome}</b><br><small>Total: R$ ${metaTotal.toFixed(2)}</small></span>
            <button class="blue" onclick="abrirProjeto(${i})">Abrir Dashboard</button>
        </div>`;
    });
    document.getElementById('listaProjetos').innerHTML = h || '<p>Nenhum projeto criado.</p>';
}

function criarProjeto() {
    let n = document.getElementById('p_nome').value;
    let d = document.getElementById('p_dias').value;
    if (!n || !d) return;
    dados.push({ nome: n, dias: Number(d), historico: [], contas: [] });
    salvar(); renderHome();
}

function abrirProjeto(i) {
    atual = i;
    document.getElementById('projectSelector').style.display = 'none';
    document.getElementById('mainDashboard').style.display = 'block';
    atualizar();
}

function fecharProjeto() {
    document.getElementById('projectSelector').style.display = 'block';
    document.getElementById('mainDashboard').style.display = 'none';
    renderHome();
}

// --- LÓGICA PRINCIPAL ---
function atualizar() {
    let p = dados[atual];
    let totalAcumulado = p.historico.reduce((s, v) => s + v, 0);
    
    // Regra: Separar Quitadas (Saldo Acumulado * % >= Valor da Conta)
    let ativas = p.contas.filter(c => (totalAcumulado * (c.perc/100)) < (c.valor - 0.01));
    let quitadas = p.contas.filter(c => (totalAcumulado * (c.perc/100)) >= (c.valor - 0.01));

    // Ordenar Ativas por Prioridade (%)
    ativas.sort((a,b) => b.perc - a.perc);

    let faltaPagar = ativas.reduce((s, c) => s + (c.valor - (totalAcumulado * (c.perc/100))), 0);
    let metaTotalContas = p.contas.reduce((s, c) => s + c.valor, 0);
    let diasRest = Math.max(1, p.dias - p.historico.length);
    
    let diariaLiq = faltaPagar / diasRest;
    let brutaSug = (diariaLiq + GASOLINA) / 0.9;

    document.getElementById('displayNome').innerText = p.nome;
    document.getElementById('metaTotalDisplay').innerText = `R$ ${metaTotalContas.toFixed(2)}`;
    document.getElementById('metaBruta').innerText = `R$ ${brutaSug.toFixed(2)}`;
    document.getElementById('metaDizimo').innerText = `R$ ${(brutaSug * 0.1).toFixed(2)}`;

    renderAtivas(ativas, diariaLiq, totalAcumulado);
    renderQuitadas(quitadas);
    renderEditor(p.contas, totalAcumulado);
    renderHistorico(p.historico);
}

function registrarGanho() {
    let p = dados[atual];
    let bruto = Number(document.getElementById('inputBruto').value);
    if (!bruto) return;

    // REGRA: Dízimo NÃO incide sobre os R$ 75 de gasolina
    let dizimo = (bruto - GASOLINA) * 0.1;
    if (dizimo < 0) dizimo = 0; // Evita dízimo negativo se ganhar menos de 75
    let liquido = (bruto - GASOLINA) - dizimo;

    if (confirm(`CONFIRMAÇÃO DE ALOCAÇÃO:\n\n🙏 Dízimo: R$ ${dizimo.toFixed(2)}\n⛽ Gasolina: R$ 75,00\n💰 Líquido p/ Contas: R$ ${liquido.toFixed(2)}`)) {
        p.historico.push(liquido);
        document.getElementById('inputBruto').value = '';
        salvar(); atualizar();
    }
}

function renderAtivas(ativas, diariaLiq, totalAcum) {
    let h = '';
    ativas.forEach(c => {
        h += `<div class="row">
            <span>${c.nome} (${c.perc}%)</span>
            <b>R$ ${(diariaLiq * (c.perc/100)).toFixed(2)}</b>
        </div>`;
    });
    document.getElementById('divisaoHoje').innerHTML = h || '<p>Nenhuma conta pendente! 🎉</p>';
}

function renderQuitadas(quitadas) {
    let h = '';
    quitadas.forEach(c => {
        h += `<div class="quitada-item">✅ <b>${c.nome}</b> - Quitado com R$ ${c.valor.toFixed(2)}</div>`;
    });
    document.getElementById('listaQuitadas').innerHTML = h || '<p style="opacity:0.5">Nenhuma conta quitada ainda.</p>';
}

function renderEditor(contas, totalAcum) {
    let h = '';
    let soma = 0;
    contas.forEach((c) => {
        let isQuitada = (totalAcum * (c.perc/100)) >= (c.valor - 0.01);
        if(!isQuitada) {
            soma += Number(c.perc);
            h += `<div class="item-edit">
                <input value="${c.nome}" onchange="editConta(${c.id}, 'nome', this.value)" style="width:30%">
                <input type="number" value="${c.valor}" onchange="editConta(${c.id}, 'valor', this.value)" style="width:25%">
                <input type="number" value="${c.perc}" onchange="autoRedistribuir(${c.id}, this.value)" style="width:15%"> %
                <button class="red" onclick="removerConta(${c.id})" style="padding:5px 8px;">🗑</button>
            </div>`;
        }
    });
    document.getElementById('editorContas').innerHTML = h + `<p style="text-align:right; color:${Math.abs(soma-100)>0.1 ? 'var(--red)' : 'var(--green)'}">Soma: ${soma.toFixed(1)}%</p>`;
}

// REGRA: Redistribuição automática de %
function autoRedistribuir(id, novoValor) {
    bkpAcao();
    let p = dados[atual];
    let valor = Math.min(100, Math.max(0, Number(novoValor)));
    let resto = 100 - valor;
    
    let totalAcum = p.historico.reduce((s,v)=>s+v,0);
    let outrasAtivas = p.contas.filter(c => c.id !== id && (totalAcum*(c.perc/100) < c.valor - 0.01));
    
    p.contas.find(c => c.id === id).perc = valor;
    if(outrasAtivas.length > 0) {
        outrasAtivas.forEach(c => c.perc = Number((resto / outrasAtivas.length).toFixed(1)));
    }
    salvar(); atualizar();
}

// REGRA: Bola de Neve (Prioriza menor saldo devedor)
function bolaDeNeve() {
    bkpAcao();
    let p = dados[atual];
    let totalAcum = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.contas.filter(c => (totalAcum * (c.perc/100)) < (c.valor - 0.01));
    
    if (ativas.length === 0) return;
    
    // Ordena pelo que falta pagar (Crescente)
    ativas.sort((a, b) => (a.valor - (totalAcum * (a.perc/100))) - (b.valor - (totalAcum * (b.perc/100))));
    
    p.contas.forEach(c => c.perc = 0); // Reseta
    
    ativas.forEach((c, i) => {
        if (i === 0) c.perc = 70;
        else if (i === 1) c.perc = 20;
        else c.perc = Number((10 / (ativas.length - 2 || 1)).toFixed(1));
    });
    salvar(); atualizar();
}

function addConta() {
    bkpAcao();
    let n = document.getElementById('c_nome').value;
    let v = document.getElementById('c_valor').value;
    if (!n || !v) return;
    dados[atual].contas.push({ id: Date.now(), nome: n, valor: Number(v), perc: 0 });
    document.getElementById('c_nome').value = ''; document.getElementById('c_valor').value = '';
    salvar(); atualizar();
}

function bkpAcao() { bkpEstrutura = JSON.parse(JSON.stringify(dados[atual].contas)); }
function desfazer() { if(bkpEstrutura) { dados[atual].contas = bkpEstrutura; salvar(); atualizar(); alert("Alterações desfeitas!"); } }

function editConta(id, campo, valor) {
    bkpAcao();
    let c = dados[atual].contas.find(x => x.id === id);
    c[campo] = campo === 'nome' ? valor : Number(valor);
    salvar(); atualizar();
}

function removerConta(id) {
    if(confirm("Excluir esta conta?")) {
        bkpAcao();
        dados[atual].contas = dados[atual].contas.filter(c => c.id !== id);
        salvar(); atualizar();
    }
}

function renderHistorico(hist) {
    document.getElementById('listaHistorico').innerHTML = hist.slice(-5).reverse().map((v, i) => `
        <div class="row" style="font-size:0.9em; opacity:0.8;">
            <span>Ganhos R$ ${v.toFixed(2)}</span>
            <button class="red" onclick="removerGanho(${hist.length - 1 - i})" style="padding:2px 5px; font-size:10px;">X</button>
        </div>
    `).join('') || '<p style="opacity:0.5">Nenhum registro hoje.</p>';
}

function removerGanho(idx) {
    if(confirm("Apagar esse registro?")) { dados[atual].historico.splice(idx, 1); salvar(); atualizar(); }
}

function excluirProjetoTotal() { if(confirm("Deseja EXCLUIR TUDO deste projeto?")) { dados.splice(atual, 1); salvar(); fecharProjeto(); } }

renderHome();
</script>
</body>
</html>
