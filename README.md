<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Meta Financeira PRO - Sistema Integrado</title>
    <style>
        body { font-family: 'Segoe UI', sans-serif; background:#0f172a; color:white; padding:20px; }
        .card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; border: 1px solid #334155; }
        button { padding:10px 15px; border:none; border-radius:8px; cursor:pointer; font-weight: bold; margin: 4px 2px; }
        input { padding:10px; border-radius:8px; border:none; background: #334155; color: white; width: 140px; margin: 5px 0; }
        .green { background:#22c55e; } .red { background:#ef4444; } .blue { background:#3b82f6; } .orange { background:#f59e0b; } .gray { background:#64748b; }
        .finance-box { background: #0f172a; padding: 15px; border-radius: 8px; border: 1px solid #3b82f6; margin: 15px 0; }
        .row { display: flex; justify-content: space-between; padding: 5px 0; border-bottom: 1px dashed #334155; }
        .despesa-item { background: #1e293b; border: 1px solid #334155; padding: 12px; border-radius: 10px; margin-bottom: 10px; position: relative; }
        .quitada-item { background: #064e3b; border: 1px solid #22c55e; padding: 10px; border-radius: 8px; margin-bottom: 5px; border-left: 5px solid #22c55e; }
        .alert { padding: 10px; border-radius: 8px; text-align: center; font-weight: bold; margin: 10px 0; }
    </style>
</head>
<body>

<div id="homePage">
    <h1>📊 Meta Financeira PRO</h1>
    <div class="card">
        <div id="listaMetas"></div>
        <hr style="border-top: 1px solid #334155; margin: 20px 0;">
        <h4>Criar Novo Projeto</h4>
        <input id="nomeMeta" placeholder="Nome (Ex: Abril)">
        <input id="diasMeta" type="number" placeholder="Dias">
        <button class="green" onclick="criarProjeto()">Criar</button>
    </div>
</div>

<div id="metaPage" style="display:none;">
    <div class="card">
        <button class="gray" onclick="voltar()">⬅ Voltar</button>
        <button class="blue" onclick="desfazerEstrutura()">↩ Desfazer Ajustes</button>
        <button class="blue" onclick="desfazerUltimoGanho()">↩ Desfazer Ganho</button>
        <button class="red" onclick="excluirProjeto()">🗑 Excluir Projeto</button>
        
        <h2 id="txtTitulo"></h2>
        <div id="txtProgresso"></div>
        <div id="txtSaldo" style="font-size: 1.4em; color: #22c55e; font-weight: bold;"></div>
        
        <div class="finance-box">
            <div class="row"><span>🎯 Meta Diária Líquida:</span> <span id="valDiariaLiq" style="color:#22c55e"></span></div>
            <div class="row"><span>⛽ Combustível (Fixo):</span> <span>R$ 75,00</span></div>
            <div class="row"><span>🙏 Dízimo Previsto (10%):</span> <span id="valDizimoSug" style="color:#fbbf24"></span></div>
            <div class="row" style="border:none; font-size:1.2em; font-weight:bold; margin-top:10px;">
                <span>🚀 BRUTO SUGERIDO:</span> <span id="valBrutoSug" style="color:#3b82f6"></span>
            </div>
        </div>

        <div id="statusRitmo" class="alert"></div>

        <div style="background: #334155; padding: 15px; border-radius: 10px;">
            <strong>Registrar Ganho Bruto REAL:</strong><br>
            <input id="inGanhoReal" type="number" placeholder="R$ 0,00">
            <button class="green" onclick="registrarGanho()">Registrar Dia</button>
        </div>
    </div>

    <div class="card">
        <h3>📋 Contas Ativas</h3>
        <button class="orange" onclick="aplicarBolaDeNeve()" style="width: 100%; margin-bottom: 15px;">🚀 Priorizar Bola de Neve (Mais Próximas)</button>
        <input id="inNomeConta" placeholder="Nome da Conta">
        <input id="inValorConta" type="number" placeholder="R$ Valor">
        <button class="blue" onclick="addConta()">Adicionar</button>
        <div id="listaAtivas" style="margin-top: 15px;"></div>
    </div>

    <div class="card">
        <h3>✅ Contas Quitadas (Fora da Divisão)</h3>
        <div id="listaQuitadas"></div>
    </div>

    <div class="card">
        <h3>📜 Histórico</h3>
        <div id="txtHistorico" style="font-size: 0.9em; opacity: 0.8;"></div>
    </div>
</div>

<script>
let db = JSON.parse(localStorage.getItem('financas_v4')) || [];
let cur = null;
let bkp = null;
const GAS = 75;

function salvar() { localStorage.setItem('financas_v4', JSON.stringify(db)); }

// --- LOGICA DE PROJETOS ---
function renderHome() {
    let h = '';
    db.forEach((p, i) => {
        let metaTotal = p.contas.reduce((s, c) => s + c.valor, 0);
        h += `<div class="card" style="display:flex; justify-content:space-between; align-items:center;">
            <span><b>${p.nome}</b><br><small>Meta: R$ ${metaTotal.toFixed(2)}</small></span>
            <button class="blue" onclick="abrir(${i})">Abrir</button>
        </div>`;
    });
    listaMetas.innerHTML = h || '<p>Crie um projeto para começar.</p>';
}

function criarProjeto() {
    if (!nomeMeta.value || !diasMeta.value) return;
    db.push({ nome: nomeMeta.value, dias: Number(diasMeta.value), historico: [], contas: [] });
    salvar(); renderHome();
}

function abrir(i) { cur = i; homePage.style.display='none'; metaPage.style.display='block'; atualizar(); }
function voltar() { homePage.style.display='block'; metaPage.style.display='none'; renderHome(); }

// --- O CORAÇÃO DO CÓDIGO (ONDE TUDO ACONTECE) ---
function atualizar() {
    let p = db[cur];
    let totalLiq = p.historico.reduce((s, v) => s + v, 0);
    
    // REGRA: Separar Ativas de Quitadas (Margem de erro de 1 centavo para arredondamento)
    let ativas = p.contas.filter(c => (totalLiq * c.perc) < (c.valor - 0.01));
    let quitadas = p.contas.filter(c => (totalLiq * c.perc) >= (c.valor - 0.01));

    // REGRA: Meta Total é a soma das contas
    let metaTotal = p.contas.reduce((s, c) => s + c.valor, 0);
    let faltaPagar = ativas.reduce((s, c) => s + (c.valor - (totalLiq * c.perc)), 0);
    let diasRest = Math.max(1, p.dias - p.historico.length);

    // REGRA: Cálculo do Bruto Sugerido (Blindando Gasolina e Dízimo)
    let diariaLiq = faltaPagar / diasRest;
    let brutoSug = (diariaLiq + GAS) / 0.9;

    // Atualização da UI
    txtTitulo.innerText = p.nome;
    txtProgresso.innerText = `Dia ${p.historico.length} de ${p.dias} | Faltam R$ ${faltaPagar.toFixed(2)} para quitar tudo.`;
    txtSaldo.innerText = `Saldo p/ Contas: R$ ${totalLiq.toFixed(2)}`;
    valDiariaLiq.innerText = `R$ ${diariaLiq.toFixed(2)}`;
    valBrutoSug.innerText = `R$ ${brutoSug.toFixed(2)}`;
    valDizimoSug.innerText = `R$ ${(brutoSug * 0.1).toFixed(2)}`;

    // Ritmo
    let esperado = (metaTotal / p.dias) * p.historico.length;
    statusRitmo.innerText = totalLiq < esperado ? "⚠️ Você está abaixo da meta ideal" : "🔥 Ritmo excelente!";
    statusRitmo.className = "alert " + (totalLiq < esperado ? "red" : "green");

    renderListas(ativas, quitadas, totalLiq, diariaLiq);
    txtHistorico.innerHTML = p.historico.slice().reverse().map((v, i) => `Dia ${p.historico.length - i}: +R$ ${v.toFixed(2)}`).join('<br>');
}

// REGRA: O Dízimo não incide sobre a gasolina no registro real
function registrarGanho() {
    let p = db[cur];
    let b = Number(inGanhoReal.value);
    if (!b) return;
    let d = b * 0.1;
    let l = (b - d) - GAS; // Tira o dízimo do bruto, depois tira a gasolina cheia
    if (confirm(`CONFIRMAR:\n🙏 Dízimo: R$ ${d.toFixed(2)}\n⛽ Gasolina: R$ 75,00\n💰 Líquido Contas: R$ ${l.toFixed(2)}`)) {
        p.historico.push(l);
        inGanhoReal.value = ''; salvar(); atualizar();
    }
}

function renderListas(ativas, quitadas, totalLiq, diariaLiq) {
    let hA = '';
    ativas.sort((a,b) => b.perc - a.perc).forEach(c => {
        let jaTem = totalLiq * c.perc;
        hA += `<div class="despesa-item">
            <b>${c.nome}</b> <span style="color:#fbbf24">${(c.perc*100).toFixed(1)}%</span><br>
            Falta: R$ ${(c.valor - jaTem).toFixed(2)} | <small>Separar hoje: R$ ${(diariaLiq * c.perc).toFixed(2)}</small>
            <button class="red" style="position:absolute; top:5px; right:5px; padding:2px 5px;" onclick="removerConta(${c.id})">X</button>
        </div>`;
    });
    listaAtivas.innerHTML = hA || '<p>Nenhuma conta ativa.</p>';

    let hQ = '';
    quitadas.forEach(c => {
        hQ += `<div class="quitada-item">✅ <b>${c.nome}</b> (R$ ${c.valor.toFixed(2)}) 
        <button class="gray" style="float:right; font-size:10px;" onclick="reabrir(${c.id})">Reabrir</button></div>`;
    });
    listaQuitadas.innerHTML = hQ;
}

// REGRA: Adicionar conta redistribui apenas entre as ATIVAS
function addConta() {
    bkpAcao();
    let p = db[cur];
    p.contas.push({ id: Date.now(), nome: inNomeConta.value, valor: Number(inValorConta.value), perc: 0 });
    inNomeConta.value = ''; inValorConta.value = '';
    distribuirIgual();
}

function distribuirIgual() {
    let p = db[cur];
    let totalLiq = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.contas.filter(c => (totalLiq * c.perc) < (c.valor - 0.01));
    if (ativas.length > 0) {
        let fatia = 1 / ativas.length;
        p.contas.forEach(c => { if(ativas.find(a => a.id === c.id)) c.perc = fatia; });
    }
    salvar(); atualizar();
}

// REGRA: Bola de neve foca em quem está mais perto de quitar
function aplicarBolaDeNeve() {
    bkpAcao();
    let p = db[cur];
    let totalLiq = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.contas.filter(c => (totalLiq * c.perc) < (c.valor - 0.01));
    if (ativas.length === 0) return;
    ativas.sort((a, b) => (a.valor - (totalLiq * a.perc)) - (b.valor - (totalLiq * b.perc)));
    ativas.forEach((c, i) => {
        if (i === 0) c.perc = 0.7; 
        else if (i === 1) c.perc = 0.2;
        else c.perc = 0.1 / (ativas.length - 2 || 1);
    });
    salvar(); atualizar();
}

// AUXILIARES
function bkpAcao() { bkp = JSON.parse(JSON.stringify(db[cur].contas)); }
function desfazerEstrutura() { if(bkp){ db[cur].contas = bkp; bkp = null; salvar(); atualizar(); } }
function desfazerUltimoGanho() { if(confirm("Apagar último registro?")){ db[cur].historico.pop(); salvar(); atualizar(); } }
function reabrir(id) { bkpAcao(); db[cur].contas.find(c => c.id === id).perc = 0; distribuirIgual(); }
function removerConta(id) { bkpAcao(); db[cur].contas = db[cur].contas.filter(c => c.id !== id); distribuirIgual(); }
function excluirProjeto() { if(confirm("Excluir projeto?")){ db.splice(cur,1); salvar(); voltar(); } }

renderHome();
</script>
</body>
</html>
