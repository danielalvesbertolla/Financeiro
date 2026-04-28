<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Meta Financeira PRO MAX</title>
<style>
    :root { --bg: #0f172a; --card: #1e293b; --text: #f8fafc; --green: #22c55e; --blue: #3b82f6; --red: #ef4444; }
    body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); margin: 0; padding: 15px; }
    .card { background: var(--card); padding: 18px; border-radius: 16px; margin-bottom: 15px; border: 1px solid #334155; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.2); }
    h1, h2, h3 { margin-top: 0; }
    input { background: #0f172a; border: 1px solid #475569; color: white; padding: 12px; border-radius: 8px; width: calc(100% - 26px); margin-bottom: 10px; font-size: 16px; }
    button { padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.2s; font-size: 14px; }
    .btn-full { width: 100%; margin-top: 5px; }
    .green { background: var(--green); color: white; }
    .blue { background: var(--blue); color: white; }
    .red { background: var(--red); color: white; }
    .gray { background: #64748b; color: white; }
    .flex { display: flex; gap: 8px; align-items: center; justify-content: space-between; }
    .despesa-item { background: #334155; padding: 12px; border-radius: 10px; margin-bottom: 10px; }
    .quitada { opacity: 0.6; text-decoration: line-through; background: #1e293b; border: 1px dashed #475569; }
    .badge-auto { background: #0ea5e9; font-size: 10px; padding: 2px 6px; border-radius: 4px; }
    .badge-manual { background: #f59e0b; font-size: 10px; padding: 2px 6px; border-radius: 4px; }
    .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 10px; }
    .stat-box { background: #0f172a; padding: 10px; border-radius: 8px; text-align: center; }
</style>
</head>
<body>

<div id="home">
    <h1>📊 Financeiro Pro</h1>
    <div class="card">
        <h3>Minhas Metas</h3>
        <div id="listaMetas"></div>
        <hr style="border-color: #334155;">
        <h4>Nova Meta</h4>
        <input id="nomeMeta" placeholder="Ex: Viagem de Fim de Ano">
        <input id="valorMeta" type="number" placeholder="Valor Total (R$)">
        <input id="diasMeta" type="number" placeholder="Prazo em Dias">
        <button class="green btn-full" onclick="criarMeta()">Criar Meta</button>
    </div>
</div>

<div id="metaPage" style="display:none;">
    <button class="gray" onclick="voltar()">⬅ Sair</button>
    
    <div class="card" style="margin-top:15px;">
        <h2 id="tituloMeta" style="color: var(--blue); margin-bottom: 5px;"></h2>
        <div id="infoEstatisticas">
            <div class="stats-grid">
                <div class="stat-box"><small>Progresso</small><br><b id="percentual">0%</b></div>
                <div class="stat-box"><small>Dias Restantes</small><br><b id="diasRest">0</b></div>
            </div>
            <p id="valorAtual" style="font-size: 1.4em; text-align: center; margin: 15px 0; color: var(--green);"></p>
            <div class="stat-box" style="background: #334155;">
                <small>Meta Líquida Diária</small><br>
                <b id="diaria" style="font-size: 1.2em;">R$ 0,00</b>
            </div>
        </div>
    </div>

    <div class="card">
        <h3>💰 Registrar Ganho do Dia</h3>
        <p style="font-size: 12px; color: #94a3b8;">* Sistema desconta automaticamente R$ 75 de combustível.</p>
        <input id="ganhoInput" type="number" placeholder="Valor Bruto (R$)">
        <button class="green btn-full" onclick="registrar()">Lançar Ganho</button>
    </div>

    <div class="card">
        <h3>📋 Alocação de Contas</h3>
        <div class="flex">
            <input id="nomeDespesa" placeholder="Nome" style="width: 50%;">
            <input id="valorDespesa" type="number" placeholder="R$" style="width: 30%;">
            <button class="blue" onclick="addDespesa()" style="margin-bottom: 10px;">+</button>
        </div>
        <div id="listaDespesas"></div>
        <h4 style="margin-top: 20px; color: #94a3b8;">✅ Contas Quitadas</h4>
        <div id="listaQuitadas"></div>
    </div>

    <div class="card" style="border-color: var(--red);">
        <button class="red btn-full" onclick="excluirMeta()">Excluir Toda a Meta</button>
    </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let metaAtual = null;

function salvar(){ localStorage.setItem('metas', JSON.stringify(metas)); }

function renderMetas(){
    let html = '';
    metas.forEach((m, i) => {
        html += `<div class="card flex">
            <span><b>${m.nome}</b><br><small>R$ ${m.meta}</small></span>
            <button class="blue" onclick="abrirMeta(${i})">Abrir</button>
        </div>`;
    });
    document.getElementById('listaMetas').innerHTML = html || '<p>Nenhuma meta criada.</p>';
}

function criarMeta(){
    const nome = document.getElementById('nomeMeta').value;
    const valor = document.getElementById('valorMeta').value;
    const dias = document.getElementById('diasMeta').value;
    if(!nome || !valor) return;
    metas.push({
        nome: nome,
        meta: Number(valor),
        diasTotal: Number(dias),
        historico: [], // Salva apenas o líquido
        despesas: []
    });
    salvar(); renderMetas();
    document.getElementById('nomeMeta').value = ''; 
    document.getElementById('valorMeta').value = '';
}

function abrirMeta(i){
    metaAtual = i;
    document.getElementById('home').style.display = 'none';
    document.getElementById('metaPage').style.display = 'block';
    atualizar();
}

function voltar(){
    document.getElementById('home').style.display = 'block';
    document.getElementById('metaPage').style.display = 'none';
    renderMetas();
}

function registrar(){
    let m = metas[metaAtual];
    let bruto = Number(document.getElementById('ganhoInput').value);
    if(bruto <= 0) return;

    // REGRA 1: Dízimo Blindado - Tira gasolina primeiro (R$ 75 fixo)
    let posGasolina = bruto - 75;
    if(posGasolina < 0) posGasolina = 0;

    // REGRA 2: 10% de Reserva/Dízimo sobre o que sobrou da gasolina
    let liquidoReal = posGasolina * 0.9;

    m.historico.push(liquidoReal);
    document.getElementById('ganhoInput').value = '';
    salvar(); atualizar();
}

function atualizar(){
    let m = metas[metaAtual];
    let ganhoAcumulado = m.historico.reduce((s, v) => s + v, 0);
    let diasPassados = m.historico.length;
    let restanteVal = m.meta - ganhoAcumulado;
    let diasRest = m.diasTotal - diasPassados;
    
    let diariaLiquida = restanteVal / (diasRest > 0 ? diasRest : 1);
    let progresso = (ganhoAcumulado / m.meta) * 100;

    document.getElementById('tituloMeta').innerText = m.nome;
    document.getElementById('percentual').innerText = progresso.toFixed(1) + '%';
    document.getElementById('diasRest').innerText = diasRest > 0 ? diasRest : 0;
    document.getElementById('valorAtual').innerText = `R$ ${ganhoAcumulado.toFixed(2)}`;
    document.getElementById('diaria').innerText = `R$ ${diariaLiquida.toFixed(2)}`;

    renderDespesas(ganhoAcumulado);
}

function addDespesa(){
    let n = document.getElementById('nomeDespesa');
    let v = document.getElementById('valorDespesa');
    if(!n.value) return;
    metas[metaAtual].despesas.push({
        nome: n.value,
        totalMeta: Number(v.value),
        percentual: 0,
        manual: false,
        quitada: false
    });
    n.value = ''; v.value = '';
    recalcularPorcentagens();
}

function alternarManual(i){
    let d = metas[metaAtual].despesas[i];
    d.manual = !d.manual;
    recalcularPorcentagens();
}

function mudarPerc(i, valor){
    let d = metas[metaAtual].despesas[i];
    d.percentual = Number(valor) / 100;
    d.manual = true;
    recalcularPorcentagens();
}

function recalcularPorcentagens(){
    let m = metas[metaAtual];
    let ativas = m.despesas.filter(d => !d.quitada);
    let manuais = ativas.filter(d => d.manual);
    let autos = ativas.filter(d => !d.manual);

    let somaManual = manuais.reduce((s, d) => s + d.percentual, 0);
    let restante = Math.max(0, 1 - somaManual);

    if(autos.length > 0){
        let fatia = restante / autos.length;
        autos.forEach(d => d.percentual = fatia);
    }
    salvar(); atualizar();
}

function quitarConta(i){
    metas[metaAtual].despesas[i].quitada = true;
    metas[metaAtual].despesas[i].percentual = 0;
    recalcularPorcentagens();
}

function renderDespesas(ganhoTotal){
    let m = metas[metaAtual];
    let html = '';
    let htmlQuitadas = '';

    m.despesas.forEach((d, i) => {
        let acumuladoConta = ganhoTotal * d.percentual;
        let itemHtml = `
            <div class="despesa-item ${d.quitada ? 'quitada' : ''}">
                <div class="flex">
                    <b>${d.nome}</b>
                    <span class="${d.manual ? 'badge-manual' : 'badge-auto'}">${d.manual ? 'MANUAL' : 'AUTO'}</span>
                </div>
                <div class="flex" style="margin: 8px 0;">
                    <small>Meta: R$ ${d.totalMeta}</small>
                    <small>Aporte: ${(d.percentual*100).toFixed(1)}%</small>
                </div>
                ${!d.quitada ? `
                    <input type="range" value="${d.percentual*100}" onchange="mudarPerc(${i}, this.value)">
                    <div class="flex">
                        <button class="blue" onclick="alternarManual(${i})">⚙️ Modo</button>
                        <button class="green" onclick="quitarConta(${i})">✔️ Quitar</button>
                        <button class="red" onclick="removerDespesa(${i})">🗑️</button>
                    </div>
                ` : ''}
            </div>`;
        
        if(d.quitada) htmlQuitadas += itemHtml;
        else html += itemHtml;
    });

    document.getElementById('listaDespesas').innerHTML = html || '<p>Sem contas ativas.</p>';
    document.getElementById('listaQuitadas').innerHTML = htmlQuitadas || '<p>-</p>';
}

function removerDespesa(i){
    metas[metaAtual].despesas.splice(i, 1);
    recalcularPorcentagens();
}

function excluirMeta(){
    if(confirm("Deseja apagar tudo desta meta?")){
        metas.splice(metaAtual, 1);
        salvar(); voltar();
    }
}

renderMetas();
</script>
</body>
</html>
