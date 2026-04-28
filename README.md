<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meta Financeira PRO - Master Control</title>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background:#0f172a; color:white; padding:10px; margin:0; }
        .container { max-width: 600px; margin: auto; padding: 15px; }
        .card { background:#1e293b; padding:18px; border-radius:12px; margin-bottom:15px; border: 1px solid #334155; position: relative; }
        button { padding:10px 14px; border:none; border-radius:8px; cursor:pointer; font-weight: bold; margin: 4px 2px; transition: 0.2s; }
        input { padding:10px; border-radius:8px; border:none; background: #334155; color: white; margin: 5px 0; }
        .green { background:#22c55e; } .red { background:#ef4444; } .blue { background:#3b82f6; } .orange { background:#f59e0b; } .gray { background:#64748b; }
        .finance-box { background: #0f172a; padding: 15px; border-radius: 8px; border: 1px solid #3b82f6; margin: 15px 0; }
        .row { display: flex; justify-content: space-between; padding: 6px 0; border-bottom: 1px dashed #334155; }
        .item-edit { background: #1e293b; border: 1px solid #475569; padding: 12px; border-radius: 10px; margin-bottom: 10px; }
        .quitada-item { background: #064e3b; border-left: 6px solid #22c55e; padding: 10px; border-radius: 8px; margin-bottom: 5px; font-size: 0.9em; }
        .badge { background: #3b82f6; padding: 2px 6px; border-radius: 4px; font-size: 11px; }
        .nav-tabs { display: flex; gap: 5px; margin-bottom: 15px; }
        .nav-tabs button { flex: 1; border-radius: 8px 8px 0 0; margin: 0; }
        .tab-content { display: none; }
        .active-tab { display: block; }
        .selected-btn { border-bottom: 3px solid white !important; filter: brightness(1.3); }
    </style>
</head>
<body>

<div class="container">
    <div id="homePage">
        <h1>📊 Meus Projetos</h1>
        <div class="card">
            <div id="listaMetas"></div>
            <hr style="border-top: 1px solid #334155; margin: 20px 0;">
            <h4>Novo Projeto</h4>
            <input id="nomeMeta" placeholder="Nome do Projeto" style="width: 100%;">
            <input id="diasMeta" type="number" placeholder="Total de Dias" style="width: 100%;">
            <button class="green" onclick="criarProjeto()" style="width: 100%;">Criar Projeto</button>
        </div>
    </div>

    <div id="metaPage" style="display:none;">
        <div class="nav-tabs">
            <button id="btnTab1" class="gray selected-btn" onclick="switchTab(1)">Painel</button>
            <button id="btnTab2" class="gray" onclick="switchTab(2)">Editar Contas</button>
            <button id="btnTab3" class="gray" onclick="switchTab(3)">Histórico</button>
            <button class="red" onclick="voltar()" style="flex: 0.5;">✖</button>
        </div>

        <div id="tab1" class="tab-content active-tab">
            <div class="card">
                <h2 id="txtTitulo" style="margin:0"></h2>
                <div id="txtProgresso" style="font-size: 0.9em; opacity: 0.8;"></div>
                
                <div class="finance-box">
                    <div class="row"><span>🎯 Meta Diária Contas:</span> <span id="valDiariaLiq" style="color:#22c55e"></span></div>
                    <div class="row"><span>⛽ Combustível (Fixo):</span> <span>R$ 75,00</span></div>
                    <div class="row"><span>🙏 Dízimo (10%):</span> <span id="valDizimoSug" style="color:#fbbf24"></span></div>
                    <div class="row" style="border:none; font-size:1.2em; font-weight:bold; margin-top:10px;">
                        <span>🚀 BRUTO HOJE:</span> <span id="valBrutoSug" style="color:#3b82f6"></span>
                    </div>
                </div>

                <div style="background: #334155; padding: 15px; border-radius: 10px;">
                    <strong>Ganho Bruto do Dia:</strong><br>
                    <input id="inGanhoReal" type="number" placeholder="R$ 0,00" style="width: 100%; box-sizing: border-box;">
                    <button class="green" onclick="registrarGanho()" style="width: 100%; margin-top: 10px;">Registrar Agora</button>
                </div>
            </div>

            <div class="card">
                <h3>📋 Divisão de Hoje</h3>
                <div id="listaAtivasSimples"></div>
            </div>
        </div>

        <div id="tab2" class="tab-content">
            <div class="card">
                <h3>⚙️ Ajustar Prioridades (%)</h3>
                <p><small>A soma deve ser 100. Digite a % e o valor da conta.</small></p>
                <div id="editorContas"></div>
                <hr style="border-top: 1px solid #334155;">
                <h4>Adicionar Nova Conta</h4>
                <input id="inNomeConta" placeholder="Nome" style="width: 45%;">
                <input id="inValorConta" type="number" placeholder="R$" style="width: 45%;">
                <button class="blue" onclick="addConta()" style="width: 100%;">Adicionar à Lista</button>
            </div>
            
            <div class="card">
                <h3>✅ Contas Finalizadas</h3>
                <div id="listaQuitadas"></div>
            </div>
        </div>

        <div id="tab3" class="tab-content">
            <div class="card">
                <h3>📜 Registros de Ganhos</h3>
                <p><small>Se registrou um valor errado, apague-o abaixo:</small></p>
                <div id="listaHistoricoEdit"></div>
            </div>
            <button class="red" onclick="excluirProjeto()" style="width: 100%;">🗑 Apagar Projeto Inteiro</button>
        </div>
    </div>
</div>

<script>
let db = JSON.parse(localStorage.getItem('financas_v5')) || [];
let cur = null;
const GAS = 75;

function salvar() { localStorage.setItem('financas_v5', JSON.stringify(db)); }

function renderHome() {
    let h = '';
    db.forEach((p, i) => {
        let metaTotal = p.contas.reduce((s, c) => s + c.valor, 0);
        h += `<div class="card" style="display:flex; justify-content:space-between; align-items:center;">
            <span><b>${p.nome}</b><br><small>R$ ${metaTotal.toFixed(2)}</small></span>
            <button class="blue" onclick="abrir(${i})">Abrir</button>
        </div>`;
    });
    listaMetas.innerHTML = h || '<p>Crie seu primeiro projeto.</p>';
}

function criarProjeto() {
    if (!nomeMeta.value || !diasMeta.value) return;
    db.push({ nome: nomeMeta.value, dias: Number(diasMeta.value), historico: [], contas: [] });
    salvar(); renderHome();
}

function abrir(i) { cur = i; homePage.style.display='none'; metaPage.style.display='block'; switchTab(1); atualizar(); }
function voltar() { homePage.style.display='block'; metaPage.style.display='none'; renderHome(); }

function switchTab(n) {
    document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active-tab'));
    document.querySelectorAll('.nav-tabs button').forEach(b => b.classList.remove('selected-btn'));
    document.getElementById('tab'+n).classList.add('active-tab');
    document.getElementById('btnTab'+n).classList.add('selected-btn');
}

function atualizar() {
    let p = db[cur];
    let totalLiq = p.historico.reduce((s, v) => s + v, 0);
    
    let ativas = p.contas.filter(c => (totalLiq * (c.perc/100)) < (c.valor - 0.01));
    let quitadas = p.contas.filter(c => (totalLiq * (c.perc/100)) >= (c.valor - 0.01));

    let faltaPagar = ativas.reduce((s, c) => s + (c.valor - (totalLiq * (c.perc/100))), 0);
    let diasRest = Math.max(1, p.dias - p.historico.length);

    let diariaLiq = faltaPagar / diasRest;
    let brutoSug = (diariaLiq + GAS) / 0.9;

    txtTitulo.innerText = p.nome;
    txtProgresso.innerText = `Dia ${p.historico.length} de ${p.dias} | Falta: R$ ${faltaPagar.toFixed(2)}`;
    valDiariaLiq.innerText = `R$ ${diariaLiq.toFixed(2)}`;
    valBrutoSug.innerText = `R$ ${brutoSug.toFixed(2)}`;
    valDizimoSug.innerText = `R$ ${(brutoSug * 0.1).toFixed(2)}`;

    renderListas(ativas, quitadas, totalLiq, diariaLiq);
    renderEditor(p.contas);
    renderHistorico(p.historico);
}

function registrarGanho() {
    let p = db[cur];
    let b = Number(inGanhoReal.value);
    if (!b) return;
    let d = b * 0.1;
    let l = (b - d) - GAS;
    if (confirm(`Confirmar: R$ ${l.toFixed(2)} líquido para as contas?`)) {
        p.historico.push(l);
        inGanhoReal.value = ''; salvar(); atualizar();
    }
}

function renderListas(ativas, quitadas, totalLiq, diariaLiq) {
    let hA = '';
    ativas.forEach(c => {
        let jaTem = totalLiq * (c.perc/100);
        hA += `<div class="row"><span>${c.nome} (${c.perc}%)</span> <b>R$ ${(diariaLiq * (c.perc/100)).toFixed(2)}</b></div>`;
    });
    listaAtivasSimples.innerHTML = hA || '<p>Tudo pago! 🎉</p>';

    let hQ = '';
    quitadas.forEach(c => {
        hQ += `<div class="quitada-item">✅ ${c.nome} (R$ ${c.valor.toFixed(2)}) <button class="gray" style="font-size:9px; float:right;" onclick="removerConta(${c.id})">Remover</button></div>`;
    });
    listaQuitadas.innerHTML = hQ;
}

function renderEditor(contas) {
    let h = '';
    let somaPerc = 0;
    contas.forEach((c, i) => {
        somaPerc += Number(c.perc);
        h += `<div class="item-edit">
            <input value="${c.nome}" onchange="editConta(${c.id}, 'nome', this.value)" style="width:40%">
            <input type="number" value="${c.valor}" onchange="editConta(${c.id}, 'valor', this.value)" style="width:25%">
            <input type="number" value="${c.perc}" onchange="editConta(${c.id}, 'perc', this.value)" style="width:15%"> %
            <button class="red" onclick="removerConta(${c.id})" style="padding:5px 8px;">🗑</button>
        </div>`;
    });
    editorContas.innerHTML = h + `<p style="color:${somaPerc != 100 ? '#ef4444' : '#22c55e'}">Soma atual: ${somaPerc}%</p>`;
}

function editConta(id, campo, valor) {
    let c = db[cur].contas.find(x => x.id === id);
    c[campo] = campo === 'nome' ? valor : Number(valor);
    salvar(); atualizar();
}

function addConta() {
    let p = db[cur];
    p.contas.push({ id: Date.now(), nome: inNomeConta.value, valor: Number(inValorConta.value), perc: 0 });
    inNomeConta.value = ''; inValorConta.value = '';
    salvar(); atualizar();
}

function removerConta(id) {
    if(confirm("Remover esta conta?")) {
        db[cur].contas = db[cur].contas.filter(c => c.id !== id);
        salvar(); atualizar();
    }
}

function renderHistorico(hist) {
    let h = '';
    hist.slice().reverse().forEach((v, i) => {
        let realIdx = hist.length - 1 - i;
        h += `<div class="row">
            <span>Dia ${realIdx+1}: R$ ${v.toFixed(2)}</span>
            <button class="red" onclick="removerGanho(${realIdx})" style="padding:2px 8px;">Apagar</button>
        </div>`;
    });
    listaHistoricoEdit.innerHTML = h || '<p>Nenhum ganho registrado.</p>';
}

function removerGanho(idx) {
    if(confirm("Apagar este registro de ganho? O saldo das contas será recalculado.")) {
        db[cur].historico.splice(idx, 1);
        salvar(); atualizar();
    }
}

function excluirProjeto() {
    if(confirm("APAGAR TUDO? Isso não pode ser desfeito.")) {
        db.splice(cur, 1);
        salvar(); voltar();
    }
}

renderHome();
</script>
</body>
</html>
