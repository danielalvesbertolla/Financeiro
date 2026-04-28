<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>Meta Financeira PRO MAX - Intelligence</title>
<style>
    :root { --bg: #0f172a; --card: #1e293b; --accent: #3b82f6; --green: #22c55e; --red: #ef4444; }
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: var(--bg); color: white; padding: 20px; margin: 0; }
    .container { max-width: 800px; margin: auto; }
    .card { background: var(--card); padding: 20px; border-radius: 12px; margin-bottom: 15px; border: 1px solid #334155; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1); }
    h1, h2, h3 { margin-top: 0; color: #f8fafc; }
    button { padding: 10px 15px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.3s; margin: 4px 2px; }
    button:hover { filter: brightness(1.2); transform: translateY(-1px); }
    input { padding: 10px; border-radius: 8px; border: 1px solid #334155; background: #0f172a; color: white; width: 120px; margin: 5px 0; }
    .grid-inputs { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 10px; margin-bottom: 15px; }
    .green { background: var(--green); } .red { background: var(--red); } .blue { background: var(--accent); }
    .despesa-item { background: #334155; padding: 12px; border-radius: 8px; margin-bottom: 10px; position: relative; }
    .quitada { opacity: 0.6; background: #1e293b; border: 1px dashed #475569; }
    .badge { font-size: 0.7em; padding: 3px 6px; border-radius: 4px; text-transform: uppercase; margin-left: 5px; }
    .badge-auto { background: #6366f1; } .badge-man { background: #f59e0b; }
    .progress-bar { height: 8px; background: #0f172a; border-radius: 4px; margin-top: 8px; overflow: hidden; }
    .progress-fill { height: 100%; background: var(--green); transition: width 0.5s; }
    .stats-row { display: flex; justify-content: space-between; flex-wrap: wrap; gap: 10px; }
    .stats-box { flex: 1; min-width: 150px; background: #334155; padding: 10px; border-radius: 8px; text-align: center; }
</style>
</head>
<body>

<div class="container">
    <div id="home">
        <div class="card">
            <h1>📊 Metas Financeiras PRO MAX</h1>
            <div id="listaMetas"></div>
            <hr style="border: 0; border-top: 1px solid #334155; margin: 20px 0;">
            <h3>Nova Meta de Faturamento</h3>
            <div class="grid-inputs">
                <input id="nomeMeta" placeholder="Ex: Viagem Japão">
                <input id="valorMeta" type="number" placeholder="Valor Total R$">
                <input id="diasMeta" type="number" placeholder="Prazo (Dias)">
            </div>
            <button class="green" onclick="criarMeta()" style="width: 100%;">Criar Nova Meta</button>
        </div>
    </div>

    <div id="metaPage" style="display:none;">
        <div class="card">
            <button onclick="voltar()">⬅ Voltar</button>
            <h2 id="tituloMeta" style="display:inline; margin-left: 15px;"></h2>
            
            <div class="stats-row" style="margin-top:20px;">
                <div class="stats-box"><small>Progresso</small><div id="percTxt" style="font-weight:bold; font-size:1.2em;"></div></div>
                <div class="stats-box"><small>Acumulado</small><div id="valAtuTxt" style="color:var(--green); font-weight:bold;"></div></div>
                <div class="stats-box"><small>Restante</small><div id="valRestTxt" style="color:var(--red);"></div></div>
            </div>

            <div class="progress-bar"><div id="barraProgresso" class="progress-fill"></div></div>

            <div id="infoDinamica" style="margin-top:15px; font-size: 0.9em; color: #cbd5e1;"></div>
        </div>

        <div class="card">
            <h3>💰 Registrar Aporte Bruto</h3>
            <div style="display: flex; gap: 10px;">
                <input id="ganhoInput" type="number" placeholder="R$ 0,00" style="flex-grow: 1; font-size: 1.2em;">
                <button class="green" onclick="registrarAporte()">Registrar e Distribuir</button>
            </div>
            <small style="color: #94a3b8;">* O sistema desconta R$ 75 de combustível e aplica a divisão.</small>
        </div>

        <div class="card">
            <h3>📋 Divisão de Despesas</h3>
            <div class="grid-inputs">
                <input id="nomeDespesa" placeholder="Nome da Conta">
                <input id="valorDespesa" type="number" placeholder="Valor Total (R$)">
                <button class="blue" onclick="addDespesa()">Adicionar</button>
            </div>
            
            <h4>Contas Ativas</h4>
            <div id="listaDespesas"></div>

            <h4 style="margin-top:20px; color: #94a3b8;">✅ Contas Quitadas</h4>
            <div id="listaQuitadas"></div>
        </div>

        <div class="card">
            <button class="blue" onclick="resetarManual()">Resetar Todas % para Automático</button>
            <button class="red" onclick="excluirMeta()">Excluir Meta</button>
        </div>
    </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas_v2')) || [];
let metaIdx = null;

function salvar() { localStorage.setItem('metas_v2', JSON.stringify(metas)); }

function criarMeta() {
    const nome = document.getElementById('nomeMeta').value;
    const valor = Number(document.getElementById('valorMeta').value);
    const dias = Number(document.getElementById('diasMeta').value);
    if(!nome || !valor) return alert("Preencha os dados da meta!");

    metas.push({
        nome, meta: valor, dias, 
        acumuladoLíquido: 0, 
        diasPassados: 0,
        despesas: [
            { nome: "Combustível", valor: 2250, acumulado: 0, porcentagem: 0, manual: false, fixo: 75 }
        ]
    });
    salvar(); renderHome();
}

function renderHome() {
    let html = '';
    metas.forEach((m, i) => {
        html += `<div class="card" style="display:flex; justify-content:space-between; align-items:center;">
            <div><strong>${m.nome}</strong><br><small>R$ ${m.acumuladoLíquido.toFixed(2)} / R$ ${m.meta}</small></div>
            <button class="blue" onclick="abrirMeta(${i})">Abrir</button>
        </div>`;
    });
    document.getElementById('listaMetas').innerHTML = html || '<p>Nenhuma meta criada.</p>';
}

function abrirMeta(i) {
    metaIdx = i;
    document.getElementById('home').style.display = 'none';
    document.getElementById('metaPage').style.display = 'block';
    recalcularDistribuicao();
    atualizarUI();
}

function voltar() {
    document.getElementById('home').style.display = 'block';
    document.getElementById('metaPage').style.display = 'none';
    renderHome();
}

function addDespesa() {
    const nome = document.getElementById('nomeDespesa').value;
    const valor = Number(document.getElementById('valorDespesa').value);
    if(!nome || !valor) return;

    metas[metaIdx].despesas.push({
        nome, valor, acumulado: 0, porcentagem: 0, manual: false
    });
    
    document.getElementById('nomeDespesa').value = '';
    document.getElementById('valorDespesa').value = '';
    recalcularDistribuicao();
    atualizarUI();
}

function recalcularDistribuicao() {
    let m = metas[metaIdx];
    // Filtra apenas despesas que ainda não foram quitadas
    let ativas = m.despesas.filter(d => d.acumulado < d.valor && !d.fixo);
    
    let ocupadoManual = ativas.reduce((acc, d) => acc + (d.manual ? d.porcentagem : 0), 0);
    
    if (ocupadoManual > 1) {
        alert("Soma manual maior que 100%! Ajustando...");
        ativas.forEach(d => { if(d.manual) d.porcentagem = d.porcentagem / ocupadoManual; });
        ocupadoManual = 1;
    }

    let restante = 1 - ocupadoManual;
    let auto = ativas.filter(d => !d.manual);
    
    if(auto.length > 0) {
        let fatia = restante / auto.length;
        auto.forEach(d => d.porcentagem = fatia);
    }
    salvar();
}

function registrarAporte() {
    let m = metas[metaIdx];
    let bruto = Number(document.getElementById('ganhoInput').value);
    if(bruto <= 0) return;

    // 1. Desconto Fixo (Combustível R$ 75)
    let liquido = bruto - 75;
    let comb = m.despesas.find(d => d.fixo);
    if(comb) comb.acumulado += 75;

    if(liquido < 0) liquido = 0;

    // 2. Distribuir entre ativas
    m.despesas.forEach(d => {
        if(!d.fixo && d.acumulado < d.valor) {
            d.acumulado += liquido * d.porcentagem;
        }
    });

    m.acumuladoLíquido += liquido;
    m.diasPassados++;
    
    document.getElementById('ganhoInput').value = '';
    salvar();
    atualizarUI();
}

function atualizarUI() {
    let m = metas[metaIdx];
    document.getElementById('tituloMeta').innerText = m.nome;
    
    let perc = (m.acumuladoLíquido / m.meta) * 100;
    document.getElementById('percTxt').innerText = perc.toFixed(1) + "%";
    document.getElementById('valAtuTxt').innerText = "R$ " + m.acumuladoLíquido.toFixed(2);
    document.getElementById('valRestTxt').innerText = "R$ " + (m.meta - m.acumuladoLíquido).toFixed(2);
    document.getElementById('barraProgresso').style.width = Math.min(perc, 100) + "%";

    let diaria = (m.meta - m.acumuladoLíquido) / Math.max((m.dias - m.diasPassados), 1);
    document.getElementById('infoDinamica').innerHTML = `
        📅 Dia ${m.diasPassados} de ${m.dias} | 🎯 Meta Diária Sugerida: <strong>R$ ${diaria.toFixed(2)}</strong>
    `;

    renderDespesas();
}

function renderDespesas() {
    let m = metas[metaIdx];
    let htmlAtivas = '';
    let htmlQuitadas = '';

    m.despesas.forEach((d, i) => {
        let percExibida = (d.porcentagem * 100).toFixed(1);
        let progress = (d.acumulado / d.valor) * 100;
        
        let itemHtml = `
            <div class="despesa-item ${d.acumulado >= d.valor ? 'quitada' : ''}">
                <div style="display:flex; justify-content:space-between;">
                    <span><strong>${d.nome}</strong> ${d.manual ? '<span class="badge badge-man">Manual</span>' : '<span class="badge badge-auto">Auto</span>'}</span>
                    <span>R$ ${d.acumulado.toFixed(2)} / ${d.valor}</span>
                </div>
                ${!d.fixo && d.acumulado < d.valor ? `
                    <div style="margin-top:8px;">
                        % na divisão: <input type="number" value="${percExibida}" 
                        onchange="ajustarManual(${i}, this.value)" style="width:60px; padding:4px;"> 
                        <button class="red" onclick="removerDespesa(${i})" style="float:right; padding:2px 8px;">X</button>
                    </div>
                ` : d.fixo ? '<small>Custo Fixo Diário: R$ 75,00</small>' : ''}
                <div class="progress-bar"><div class="progress-fill" style="width:${Math.min(progress, 100)}%; background:${progress>=100?'#facc15':''}"></div></div>
            </div>
        `;

        if(d.acumulado >= d.valor && !d.fixo) htmlQuitadas += itemHtml;
        else htmlAtivas += itemHtml;
    });

    document.getElementById('listaDespesas').innerHTML = htmlAtivas;
    document.getElementById('listaQuitadas').innerHTML = htmlQuitadas;
}

function ajustarManual(i, val) {
    metas[metaIdx].despesas[i].porcentagem = Number(val) / 100;
    metas[metaIdx].despesas[i].manual = true;
    recalcularDistribuicao();
    atualizarUI();
}

function resetarManual() {
    metas[metaIdx].despesas.forEach(d => d.manual = false);
    recalcularDistribuicao();
    atualizarUI();
}

function removerDespesa(i) {
    metas[metaIdx].despesas.splice(i, 1);
    recalcularDistribuicao();
    atualizarUI();
}

function excluirMeta() {
    if(confirm("Deseja deletar esta meta?")) {
        metas.splice(metaIdx, 1);
        salvar();
        voltar();
    }
}

renderHome();
</script>
</body>
</html>
