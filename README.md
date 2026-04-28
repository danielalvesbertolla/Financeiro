<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financeiro Master - Versão 3.0</title>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; --border: #334155; --green: #22c55e; --red: #ef4444; --blue: #3b82f6; --orange: #f59e0b; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: white; margin: 0; padding: 10px; }
        .container { max-width: 650px; margin: auto; padding-bottom: 30px; }
        .card { background: var(--card); padding: 18px; border-radius: 12px; margin-bottom: 15px; border: 1px solid var(--border); box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        button { padding: 10px 14px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.2s; margin: 2px; }
        button:hover { filter: brightness(1.2); }
        input { padding: 10px; border-radius: 8px; border: none; background: #0f172a; color: white; margin: 5px 0; border: 1px solid var(--border); }
        .green { background: var(--green); } .red { background: var(--red); } .blue { background: var(--blue); } .orange { background: var(--orange); } .gray { background: #64748b; }
        
        .nav-tabs { display: flex; gap: 5px; margin-bottom: 15px; position: sticky; top: 0; z-index: 10; background: var(--bg); padding: 5px 0; }
        .nav-tabs button { flex: 1; opacity: 0.6; }
        .nav-tabs button.active { opacity: 1; border-bottom: 3px solid white; background: var(--card); }
        .tab-content { display: none; }
        .tab-content.active { display: block; }

        .finance-box { background: rgba(0,0,0,0.3); padding: 15px; border-radius: 8px; border: 1px solid var(--blue); margin: 15px 0; }
        .row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px dashed var(--border); }
        .item-edit { background: rgba(255,255,255,0.03); padding: 12px; border-radius: 10px; margin-bottom: 10px; border: 1px solid var(--border); }
        .quitada-item { background: #064e3b; border-left: 6px solid var(--green); padding: 10px; border-radius: 8px; margin-bottom: 8px; }
        .alert { padding: 10px; border-radius: 8px; text-align: center; font-weight: bold; margin: 10px 0; }
    </style>
</head>
<body>

<div class="container">
    <div id="homePage">
        <h1>📊 Meus Projetos</h1>
        <div class="card">
            <div id="listaProjetos"></div>
            <hr style="border-top: 1px solid var(--border); margin: 20px 0;">
            <input id="p_nome" placeholder="Nome do Projeto" style="width: 100%; box-sizing: border-box;">
            <input id="p_dias" type="number" placeholder="Total de Dias" style="width: 100%; box-sizing: border-box;">
            <button class="green" onclick="criarProjeto()" style="width: 100%; margin-top: 10px;">Criar Projeto</button>
        </div>
    </div>

    <div id="projetoPage" style="display:none;">
        <div class="nav-tabs">
            <button id="t1" class="active" onclick="switchTab(1)">Painel</button>
            <button id="t2" onclick="switchTab(2)">Editar/Prioridade</button>
            <button id="t3" onclick="switchTab(3)">Histórico</button>
            <button class="red" onclick="voltar()" style="flex:0.3">✖</button>
        </div>

        <div id="tab1" class="tab-content active">
            <div class="card">
                <h2 id="displayNome"></h2>
                <div class="finance-box">
                    <div class="row"><span>🎯 Meta Total (Soma das Contas):</span> <span id="metaTotalDisplay"></span></div>
                    <div class="row"><span>⛽ Combustível Diário:</span> <span>R$ 75,00</span></div>
                    <div class="row"><span>🙏 Dízimo Previsto:</span> <span id="metaDizimo" style="color:var(--orange)"></span></div>
                    <div class="row" style="font-size:1.2em; border:none;"><span>🚀 BRUTO NECESSÁRIO HOJE:</span> <span id="metaBruta" style="color:var(--blue)"></span></div>
                </div>

                <div style="background: rgba(255,255,255,0.05); padding: 15px; border-radius: 10px;">
                    <strong>Registrar Ganho Bruto:</strong><br>
                    <input id="inputBruto" type="number" placeholder="R$ 0,00" style="width: 100%; box-sizing: border-box;">
                    <button class="green" onclick="registrarGanho()" style="width: 100%; margin-top: 10px;">Confirmar Entrada</button>
                </div>
            </div>
            <div class="card">
                <h3>💰 Divisão de Hoje</h3>
                <div id="divisaoHoje"></div>
            </div>
        </div>

        <div id="tab2" class="tab-content">
            <div class="card">
                <div style="display:flex; justify-content: space-between;">
                    <button class="orange" onclick="bolaDeNeve()">🚀 Bola de Neve (Menor Valor)</button>
                    <button class="blue" onclick="desfazer()">↩ Desfazer Alterações</button>
                </div>
                <p><small>* Alterar uma % redistribui o restante automaticamente.</small></p>
                <div id="editorContas"></div>
                <hr style="border-top: 1px solid var(--border); margin: 15px 0;">
                <input id="c_nome" placeholder="Nova Conta" style="width: 45%;">
                <input id="c_valor" type="number" placeholder="Valor R$" style="width: 45%;">
                <button class="blue" onclick="addConta()" style="width: 100%;">Adicionar Conta</button>
            </div>
            <div class="card">
                <h3>✅ Contas Quitadas</h3>
                <div id="listaQuitadas"></div>
            </div>
        </div>

        <div id="tab3" class="tab-content">
            <div class="card">
                <h3>📜 Histórico</h3>
                <div id="listaHistorico"></div>
            </div>
            <button class="red" onclick="excluirProjetoTotal()" style="width:100%; opacity:0.5;">Excluir Projeto</button>
        </div>
    </div>
</div>

<script>
let dados = JSON.parse(localStorage.getItem('financas_v3_final')) || [];
let atual = null;
let bkpEstrutura = null;
const GASOLINA = 75;

function salvar() { localStorage.setItem('financas_v3_final', JSON.stringify(dados)); }

function criarProjeto() {
    let n = document.getElementById('p_nome').value;
    let d = document.getElementById('p_dias').value;
    if (!n || !d) return;
    dados.push({ nome: n, dias: Number(d), historico: [], contas: [] });
    salvar(); renderHome();
}

function renderHome() {
    let h = '';
    dados.forEach((p, i) => {
        h += `<div class="card" style="display:flex; justify-content:space-between; align-items:center;">
            <span><b>${p.nome}</b></span>
            <button class="blue" onclick="abrirProjeto(${i})">Abrir</button>
        </div>`;
    });
    document.getElementById('listaProjetos').innerHTML = h || '<p>Crie um projeto.</p>';
}

function abrirProjeto(i) {
    atual = i;
    document.getElementById('homePage').style.display = 'none';
    document.getElementById('projetoPage').style.display = 'block';
    atualizar();
}

function voltar() { document.getElementById('homePage').style.display = 'block'; document.getElementById('projetoPage').style.display = 'none'; renderHome(); }

function switchTab(n) {
    document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.nav-tabs button').forEach(b => b.classList.remove('active'));
    document.getElementById('tab'+n).classList.add('active');
    document.getElementById('t'+n).classList.add('active');
}

function atualizar() {
    let p = dados[atual];
    let totalAcumulado = p.historico.reduce((s, v) => s + v, 0);
    
    // Filtro: Ativas e Quitadas
    let ativas = p.contas.filter(c => (totalAcumulado * (c.perc/100)) < (c.valor - 0.01));
    let quitadas = p.contas.filter(c => (totalAcumulado * (c.perc/100)) >= (c.valor - 0.01));

    // Meta Baseada nas Contas Ativas
    let faltaPagar = ativas.reduce((s, c) => s + (c.valor - (totalAcumulado * (c.perc/100))), 0);
    let metaTotalContas = p.contas.reduce((s, c) => s + c.valor, 0);
    let diasRest = Math.max(1, p.dias - p.historico.length);
    
    let diariaLiq = faltaPagar / diasRest;
    let brutaSug = (diariaLiq + GASOLINA) / 0.9;

    document.getElementById('displayNome').innerText = p.nome;
    document.getElementById('metaTotalDisplay').innerText = `R$ ${metaTotalContas.toFixed(2)}`;
    document.getElementById('metaBruta').innerText = `R$ ${brutaSug.toFixed(2)}`;
    document.getElementById('metaDizimo').innerText = `R$ ${(brutaSug * 0.1).toFixed(2)}`;

    renderListas(ativas, quitadas, totalAcumulado, diariaLiq);
    renderEditor(p.contas, totalAcumulado);
    renderHistorico(p.historico);
}

function registrarGanho() {
    let p = dados[atual];
    let bruto = Number(document.getElementById('inputBruto').value);
    if (!bruto) return;

    // Regra: Dízimo não incide sobre gasolina
    let dizimo = (bruto - GASOLINA) * 0.1;
    let liquido = (bruto - GASOLINA) - dizimo;

    if (confirm(`ALOCAÇÃO REAL:\n🙏 Dízimo: R$ ${dizimo.toFixed(2)}\n⛽ Gasolina: R$ 75.00\n💰 Líquido Contas: R$ ${liquido.toFixed(2)}`)) {
        p.historico.push(liquido);
        document.getElementById('inputBruto').value = '';
        salvar(); atualizar();
    }
}

function renderListas(ativas, quitadas, totalAcum, diariaLiq) {
    // Ordenar ativas por prioridade (%)
    ativas.sort((a,b) => b.perc - a.perc);
    
    let hA = '';
    ativas.forEach(c => {
        hA += `<div class="row"><span>${c.nome} (${c.perc}%)</span> <b>R$ ${(diariaLiq * (c.perc/100)).toFixed(2)}</b></div>`;
    });
    document.getElementById('divisaoHoje').innerHTML = hA || '<p>Tudo pago!</p>';

    let hQ = '';
    quitadas.forEach(c => {
        hQ += `<div class="quitada-item">✅ <b>${c.nome}</b> - Quitado com R$ ${c.valor.toFixed(2)}</div>`;
    });
    document.getElementById('listaQuitadas').innerHTML = hQ;
}

function renderEditor(contas, totalAcum) {
    let h = '';
    contas.forEach((c) => {
        let isQuitada = (totalAcum * (c.perc/100)) >= (c.valor - 0.01);
        if(!isQuitada) {
            h += `<div class="item-edit">
                <input value="${c.nome}" onchange="editConta(${c.id}, 'nome', this.value)" style="width:35%">
                <input type="number" value="${c.valor}" onchange="editConta(${c.id}, 'valor', this.value)" style="width:25%">
                <input type="number" value="${c.perc}" onchange="autoRedistribuir(${c.id}, this.value)" style="width:15%"> %
                <button class="red" onclick="removerConta(${c.id})">🗑</button>
            </div>`;
        }
    });
    document.getElementById('editorContas').innerHTML = h;
}

function autoRedistribuir(id, novoValor) {
    bkpAcao();
    let p = dados[atual];
    let valor = Math.min(100, Math.max(0, Number(novoValor)));
    let resto = 100 - valor;
    
    let outrasAtivas = p.contas.filter(c => c.id !== id && (p.historico.reduce((s,v)=>s+v,0)*(c.perc/100) < c.valor - 0.01));
    
    p.contas.find(c => c.id === id).perc = valor;
    if(outrasAtivas.length > 0) {
        outrasAtivas.forEach(c => c.perc = Number((resto / outrasAtivas.length).toFixed(2)));
    }
    salvar(); atualizar();
}

function bolaDeNeve() {
    bkpAcao();
    let p = dados[atual];
    let totalAcum = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.contas.filter(c => (totalAcum * (c.perc/100)) < (c.valor - 0.01));
    
    if (ativas.length === 0) return;
    
    // Ordenar pelo que falta pagar (Crescente)
    ativas.sort((a, b) => (a.valor - (totalAcum * (a.perc/100))) - (b.valor - (totalAcum * (b.perc/100))));
    
    ativas.forEach((c, i) => {
        if (i === 0) c.perc = 70;
        else if (i === 1) c.perc = 20;
        else c.perc = Number((10 / (ativas.length - 2 || 1)).toFixed(2));
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
function desfazer() { if(bkpEstrutura) { dados[atual].contas = bkpEstrutura; salvar(); atualizar(); } }

function editConta(id, campo, valor) {
    bkpAcao();
    let c = dados[atual].contas.find(x => x.id === id);
    c[campo] = campo === 'nome' ? valor : Number(valor);
    salvar(); atualizar();
}

function removerConta(id) {
    bkpAcao();
    dados[atual].contas = dados[atual].contas.filter(c => c.id !== id);
    salvar(); atualizar();
}

function renderHistorico(hist) {
    document.getElementById('listaHistorico').innerHTML = hist.slice().reverse().map((v, i) => `
        <div class="row"><span>Dia ${hist.length - i}: R$ ${v.toFixed(2)}</span>
        <button class="red" onclick="removerGanho(${hist.length - 1 - i})">🗑</button></div>
    `).join('');
}

function removerGanho(idx) {
    if(confirm("Remover este registro?")) { dados[atual].historico.splice(idx, 1); salvar(); atualizar(); }
}

function excluirProjetoTotal() { if(confirm("Excluir projeto?")) { dados.splice(atual,1); salvar(); voltar(); } }

renderHome();
</script>
</body>
</html>
