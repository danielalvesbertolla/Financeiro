<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Meta Financeira PRO - Versão Final de Ouro</title>
<style>
    body { font-family: 'Segoe UI', sans-serif; background:#0f172a; color:white; padding:20px; line-height: 1.6; }
    .card { background:#1e293b; padding:20px; border-radius:12px; margin-bottom:15px; border: 1px solid #334155; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
    button { padding:10px 15px; border:none; border-radius:8px; cursor:pointer; font-weight: bold; transition: 0.3s; margin: 4px 2px; }
    button:hover { filter: brightness(1.2); }
    input { padding:10px; border-radius:8px; border:none; background: #334155; color: white; width: 120px; margin: 5px 0; }
    .green { background:#22c55e; } .red { background:#ef4444; } .blue { background:#3b82f6; } .orange { background:#f59e0b; } .gray { background:#64748b; }
    .finance-box { background: #0f172a; padding: 15px; border-radius: 8px; margin: 15px 0; border: 1px solid #3b82f6; }
    .finance-row { display: flex; justify-content: space-between; padding: 6px 0; border-bottom: 1px dashed #334155; }
    .despesa-item { background: #1e293b; border: 1px solid #334155; padding: 15px; border-radius: 10px; margin-bottom: 10px; position: relative; }
    .quitada-item { background: #064e3b; border: 1px solid #22c55e; padding: 12px; border-radius: 10px; margin-bottom: 8px; border-left: 6px solid #22c55e; }
    .alert { padding: 12px; border-radius: 8px; font-weight: bold; text-align: center; margin: 10px 0; }
</style>
</head>
<body>

<div id="homePage">
    <h1>📊 Gestor Financeiro PRO</h1>
    <div class="card">
        <h3>Meus Projetos</h3>
        <div id="listaMetas"></div>
        <hr style="border: 0; border-top: 1px solid #334155; margin: 20px 0;">
        <h4>Novo Projeto</h4>
        <input id="nomeMeta" placeholder="Ex: Abril 2026">
        <input id="diasMeta" type="number" placeholder="Dias">
        <button class="green" onclick="criarProjeto()">Iniciar</button>
    </div>
</div>

<div id="metaPage" style="display:none;">
    <div class="card">
        <div style="display: flex; justify-content: space-between; flex-wrap: wrap;">
            <button class="gray" onclick="voltar()">⬅ Voltar</button>
            <div>
                <button class="blue" onclick="desfazerAlteracaoEstrutura()">↩ Desfazer Ajustes</button>
                <button class="blue" onclick="desfazerUltimoGanho()">↩ Desfazer Ganho</button>
                <button class="red" onclick="excluirProjetoAtual()">🗑 Excluir Tudo</button>
            </div>
        </div>
        
        <h2 id="tituloMetaDisplay"></h2>
        
        <div id="statsPrimarias">
            <div id="progressoTexto"></div>
            <div id="saldoAtual" style="font-size: 1.3em; color: #22c55e; font-weight: bold;"></div>
            <div id="faltaTotal" style="color: #ef4444;"></div>
        </div>

        <div class="finance-box">
            <div class="finance-row"><span>🎯 Meta Diária (Contas Ativas):</span> <span id="diariaLiquida" style="color:#22c55e"></span></div>
            <div class="finance-row"><span>⛽ Combustível (Fixo):</span> <span style="color:#ef4444">R$ 75,00</span></div>
            <div class="finance-row"><span>🙏 Dízimo Sugerido (10%):</span> <span id="valDizimoDisplay" style="color:#fbbf24"></span></div>
            <div class="finance-row" style="border:none; font-size:1.2em; margin-top:10px;">
                <span>🚀 <b>BRUTO NECESSÁRIO HOJE:</b></span> <span id="brutoSugerido" style="color:#3b82f6; font-weight:bold;"></span>
            </div>
        </div>

        <div id="alertaRitmo" class="alert"></div>

        <div style="background: #334155; padding: 15px; border-radius: 10px;">
            <strong>Registrar Ganho Bruto REAL:</strong><br>
            <input id="ganhoRealInput" type="number" placeholder="R$ 0,00" style="width: 150px;">
            <button class="green" onclick="registrarGanhoReal()">Registrar</button>
        </div>
    </div>

    <div class="card">
        <h3>📋 Contas em Aberto</h3>
        <button class="orange" onclick="priorizarBolaDeNeve()" style="width: 100%; margin-bottom: 15px;">🚀 Priorizar Quitação (Bola de Neve)</button>
        <div style="margin-bottom: 15px;">
            <input id="nomeDespesa" placeholder="Conta" style="width: 160px;">
            <input id="valorDespesa" type="number" placeholder="Valor R$">
            <button class="blue" onclick="adicionarDespesa()">Adicionar</button>
        </div>
        <div id="listaAtivas"></div>
    </div>

    <div class="card">
        <h3>✅ Contas Quitadas</h3>
        <div id="listaQuitadas"></div>
    </div>

    <div class="card">
        <h3>📜 Histórico Líquido</h3>
        <div id="historicoDisplay"></div>
    </div>
</div>

<script>
let projetos = JSON.parse(localStorage.getItem('financas_final')) || [];
let idxAtual = null;
let backupContas = null;
const GASOLINA_FIXA = 75;

function salvar() { localStorage.setItem('financas_final', JSON.stringify(projetos)); }

function renderHome() {
    let html = '';
    projetos.forEach((p, i) => {
        let total = p.despesas.reduce((s, d) => s + d.valor, 0);
        html += `<div class="card" style="display:flex; justify-content:space-between; align-items:center;">
            <div><strong>${p.nome}</strong><br><small>Meta: R$ ${total.toFixed(2)}</small></div>
            <button class="blue" onclick="abrirProjeto(${i})">Abrir</button>
        </div>`;
    });
    listaMetas.innerHTML = html || '<p>Nenhum projeto.</p>';
}

function criarProjeto() {
    if (!nomeMeta.value || !diasMeta.value) return;
    projetos.push({ nome: nomeMeta.value, dias: Number(diasMeta.value), historico: [], despesas: [] });
    salvar(); renderHome();
    nomeMeta.value = ''; diasMeta.value = '';
}

function abrirProjeto(i) {
    idxAtual = i;
    homePage.style.display = 'none';
    metaPage.style.display = 'block';
    atualizarTudo();
}

function voltar() { homePage.style.display = 'block'; metaPage.style.display = 'none'; renderHome(); }

function atualizarTudo() {
    let p = projetos[idxAtual];
    let totalAcumulado = p.historico.reduce((s, v) => s + v, 0);
    
    // Filtro inteligente: quem já recebeu sua parte total?
    let ativas = p.despesas.filter(d => (totalAcumulado * d.porcentagem) < (d.valor - 0.01));
    let quitadas = p.despesas.filter(d => (totalAcumulado * d.porcentagem) >= (d.valor - 0.01));

    // Meta Diária baseada no que FALTA nas ativas
    let faltaNasAtivas = ativas.reduce((s, d) => s + (d.valor - (totalAcumulado * d.porcentagem)), 0);
    let metaTotalOriginal = p.despesas.reduce((s, d) => s + d.valor, 0);
    let diasRestantes = Math.max(1, p.dias - p.historico.length);

    let diariaLiq = faltaNasAtivas / diasRestantes;
    let brutoSug = (diariaLiq + GASOLINA_FIXA) / 0.9;

    tituloMetaDisplay.innerText = p.nome;
    progressoTexto.innerText = `Dia ${p.historico.length} de ${p.dias}`;
    saldoAtual.innerText = `Líquido Acumulado: R$ ${totalAcumulado.toFixed(2)}`;
    faltaTotal.innerText = `Falta para Meta: R$ ${faltaNasAtivas.toFixed(2)}`;
    
    document.getElementById('diariaLiquida').innerText = `R$ ${diariaLiq.toFixed(2)}`;
    document.getElementById('brutoSugerido').innerText = `R$ ${brutoSug.toFixed(2)}`;
    document.getElementById('valDizimoDisplay').innerText = `R$ ${(brutoSug * 0.1).toFixed(2)}`;

    let ritmoIdeal = (metaTotalOriginal / p.dias) * p.historico.length;
    alertaRitmo.innerText = totalAcumulado < ritmoIdeal ? "⚠️ Ritmo Lento" : "🔥 No Ritmo!";
    alertaRitmo.className = "alert " + (totalAcumulado < ritmoIdeal ? "red" : "green");

    renderListas(ativas, quitadas, totalAcumulado, diariaLiq);
    renderHistorico(p.historico);
}

function registrarGanhoReal() {
    let p = projetos[idxAtual];
    let bruto = Number(ganhoRealInput.value);
    if (!bruto) return;

    let dizimo = bruto * 0.1;
    let liquido = (bruto - dizimo) - GASOLINA_FIXA;

    if (confirm(`REGISTRO:\n🙏 Dízimo: R$ ${dizimo.toFixed(2)}\n⛽ Gasolina: R$ 75.00\n💰 Líquido: R$ ${liquido.toFixed(2)}`)) {
        p.historico.push(liquido);
        ganhoRealInput.value = '';
        salvar(); atualizarTudo();
    }
}

function renderListas(ativas, quitadas, totalGeral, diariaLiq) {
    let hAtivas = '';
    ativas.sort((a,b) => b.porcentagem - a.porcentagem).forEach(d => {
        let alocado = totalGeral * d.porcentagem;
        hAtivas += `<div class="despesa-item">
            <b>${d.nome}</b> <span style="color:#fbbf24">${(d.porcentagem*100).toFixed(1)}%</span><br>
            Falta: R$ ${(d.valor - alocado).toFixed(2)} | <small>Hoje: R$ ${(diariaLiq * d.porcentagem).toFixed(2)}</small>
            <button class="red" style="position:absolute; top:5px; right:5px;" onclick="removerDespesa(${d.id})">X</button>
        </div>`;
    });
    listaAtivas.innerHTML = hAtivas || '<p>Tudo pronto!</p>';

    let hQuitadas = '';
    quitadas.forEach(d => {
        hQuitadas += `<div class="quitada-item">
            ✅ <b>${d.nome}</b> (R$ ${d.valor.toFixed(2)})
            <button class="gray" style="float:right; font-size:10px;" onclick="reabrirConta(${d.id})">Reabrir</button>
        </div>`;
    });
    listaQuitadas.innerHTML = hQuitadas;
}

function adicionarDespesa() {
    backupAcao();
    let p = projetos[idxAtual];
    p.despesas.push({ id: Date.now(), nome: nomeDespesa.value, valor: Number(valorDespesa.value), porcentagem: 0 });
    nomeDespesa.value = ''; valorDespesa.value = '';
    redistribuirAtivas();
}

function redistribuirAtivas() {
    let p = projetos[idxAtual];
    let totalAcumulado = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.despesas.filter(d => (totalAcumulado * d.porcentagem) < (d.valor - 0.01));
    if (ativas.length > 0) {
        let fatia = 1 / ativas.length;
        p.despesas.forEach(d => {
            if (ativas.find(a => a.id === d.id)) d.porcentagem = fatia;
            else if ((totalAcumulado * d.porcentagem) < d.valor) d.porcentagem = 0; 
        });
    }
    salvar(); atualizarTudo();
}

function priorizarBolaDeNeve() {
    backupAcao();
    let p = projetos[idxAtual];
    let totalGeral = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.despesas.filter(d => (totalGeral * d.porcentagem) < (d.valor - 0.01));
    if (ativas.length === 0) return;
    ativas.sort((a, b) => (a.valor - (totalGeral * a.porcentagem)) - (b.valor - (totalGeral * b.porcentagem)));
    
    ativas.forEach((d, i) => {
        if (i === 0) d.porcentagem = 0.70;
        else if (i === 1) d.porcentagem = 0.20;
        else d.porcentagem = 0.10 / (ativas.length - 2 || 1);
    });
    salvar(); atualizarTudo();
}

function backupAcao() { backupContas = JSON.parse(JSON.stringify(projetos[idxAtual].despesas)); }
function desfazerAlteracaoEstrutura() { if(backupContas){ projetos[idxAtual].despesas = backupContas; backupContas = null; salvar(); atualizarTudo(); } }
function desfazerUltimoGanho() { if(confirm("Remover último registro?")){ projetos[idxAtual].historico.pop(); salvar(); atualizarTudo(); } }
function reabrirConta(id) { backupAcao(); projetos[idxAtual].despesas.find(x => x.id === id).porcentagem = 0; redistribuirAtivas(); }
function removerDespesa(id) { backupAcao(); projetos[idxAtual].despesas = projetos[idxAtual].despesas.filter(d => d.id !== id); redistribuirAtivas(); }
function renderHistorico(hist) { historicoDisplay.innerHTML = hist.slice().reverse().map((v, i) => `<div>Dia ${hist.length - i}: + R$ ${v.toFixed(2)}</div>`).join(''); }
function excluirProjetoAtual() { if(confirm("Excluir projeto?")){ projetos.splice(idxAtual, 1); salvar(); voltar(); } }

renderHome();
</script>
</body>
</html>
