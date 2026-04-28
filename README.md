<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financeiro PRO - Versão Definitiva</title>
    <style>
        :root { --bg: #0f172a; --card: #1e293b; --border: #334155; --green: #22c55e; --red: #ef4444; --blue: #3b82f6; --orange: #f59e0b; }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: white; margin: 0; padding: 10px; }
        .container { max-width: 650px; margin: auto; padding-bottom: 50px; }
        .card { background: var(--card); padding: 18px; border-radius: 12px; margin-bottom: 15px; border: 1px solid var(--border); box-shadow: 0 4px 10px rgba(0,0,0,0.3); }
        h1, h2, h3 { margin-top: 0; }
        button { padding: 10px 14px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.2s; margin: 2px; }
        button:hover { filter: brightness(1.2); }
        input { padding: 10px; border-radius: 8px; border: none; background: #0f172a; color: white; margin: 5px 0; border: 1px solid var(--border); }
        .green { background: var(--green); } .red { background: var(--red); } .blue { background: var(--blue); } .orange { background: var(--orange); } .gray { background: #64748b; }
        
        /* Painel de Sugestão */
        .finance-box { background: rgba(0,0,0,0.3); padding: 15px; border-radius: 8px; border: 1px solid var(--blue); margin: 15px 0; }
        .row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px dashed var(--border); }
        .row:last-child { border-bottom: none; font-size: 1.1em; font-weight: bold; }

        /* Abas */
        .nav-tabs { display: flex; gap: 5px; margin-bottom: 15px; position: sticky; top: 0; z-index: 10; background: var(--bg); padding: 5px 0; }
        .nav-tabs button { flex: 1; border-radius: 8px; opacity: 0.6; }
        .nav-tabs button.active { opacity: 1; border-bottom: 3px solid white; background: var(--card); }
        .tab-content { display: none; }
        .tab-content.active { display: block; }

        /* Itens */
        .item-edit { background: rgba(255,255,255,0.03); padding: 12px; border-radius: 10px; margin-bottom: 10px; border: 1px solid var(--border); }
        .quitada-item { background: #064e3b; border-left: 6px solid var(--green); padding: 10px; border-radius: 8px; margin-bottom: 8px; opacity: 0.8; }
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
            <h3>Novo Projeto de Quitação</h3>
            <input id="p_nome" placeholder="Ex: Abril 2026" style="width: 100%; box-sizing: border-box;">
            <input id="p_dias" type="number" placeholder="Quantos dias planeja?" style="width: 100%; box-sizing: border-box;">
            <button class="green" onclick="criarProjeto()" style="width: 100%; margin-top: 10px;">Começar Projeto</button>
        </div>
    </div>

    <div id="projetoPage" style="display:none;">
        <div class="nav-tabs">
            <button id="t1" class="active" onclick="switchTab(1)">Painel</button>
            <button id="t2" onclick="switchTab(2)">Contas/Edit</button>
            <button id="t3" onclick="switchTab(3)">Histórico</button>
            <button class="red" onclick="voltar()" style="flex:0.3">✖</button>
        </div>

        <div id="tab1" class="tab-content active">
            <div class="card">
                <h2 id="displayNome" style="margin-bottom:5px;"></h2>
                <div id="displayDias" style="font-size: 0.9em; opacity: 0.7;"></div>
                
                <div class="finance-box">
                    <div class="row"><span>🎯 Meta Diária (Contas):</span> <span id="metaLiq" style="color:var(--green)"></span></div>
                    <div class="row"><span>⛽ Combustível (Fixo):</span> <span>R$ 75,00</span></div>
                    <div class="row"><span>🙏 Dízimo Sugerido (10%):</span> <span id="metaDizimo" style="color:var(--orange)"></span></div>
                    <div class="row"><span>🚀 BRUTO NECESSÁRIO:</span> <span id="metaBruta" style="color:var(--blue)"></span></div>
                </div>

                <div id="ritmoAlert" class="alert"></div>

                <div style="background: rgba(255,255,255,0.05); padding: 15px; border-radius: 10px; border: 1px solid var(--border);">
                    <strong>Registrar Ganho Bruto Hoje:</strong><br>
                    <input id="inputBruto" type="number" placeholder="R$ 0,00" style="width: 100%; font-size: 1.2em; box-sizing: border-box;">
                    <button class="green" onclick="registrarGanho()" style="width: 100%; margin-top: 10px; font-size: 1.1em;">Confirmar Recebimento</button>
                </div>
            </div>

            <div class="card">
                <h3>💰 Divisão do Valor de Hoje</h3>
                <div id="divisaoHoje"></div>
            </div>
        </div>

        <div id="tab2" class="tab-content">
            <div class="card">
                <button class="orange" onclick="bolaDeNeve()" style="width: 100%; margin-bottom: 15px;">🚀 Priorização Bola de Neve (Automática)</button>
                <div id="editorContas"></div>
                
                <hr style="border-top: 1px solid var(--border); margin: 20px 0;">
                <h4>Adicionar Nova Conta</h4>
                <input id="c_nome" placeholder="Nome" style="width: 45%;">
                <input id="c_valor" type="number" placeholder="Valor R$" style="width: 45%;">
                <button class="blue" onclick="addConta()" style="width: 100%;">Adicionar à Lista</button>
            </div>

            <div class="card">
                <h3>✅ Contas Quitadas</h3>
                <div id="listaQuitadas"></div>
            </div>
        </div>

        <div id="tab3" class="tab-content">
            <div class="card">
                <h3>📜 Histórico de Ganhos</h3>
                <p><small>Apague registros específicos para corrigir o saldo:</small></p>
                <div id="listaHistorico"></div>
            </div>
            <button class="red" onclick="excluirProjetoTotal()" style="width: 100%; margin-top: 20px; opacity: 0.5;">🗑 Excluir Projeto Inteiro</button>
        </div>
    </div>
</div>

<script>
let dados = JSON.parse(localStorage.getItem('financas_ultra_v1')) || [];
let atual = null;
const GASOLINA = 75;

function salvar() { localStorage.setItem('financas_ultra_v1', JSON.stringify(dados)); }

// --- NAVEGAÇÃO ---
function renderHome() {
    let h = '';
    dados.forEach((p, i) => {
        let total = p.contas.reduce((s, c) => s + c.valor, 0);
        h += `<div class="card" style="display:flex; justify-content:space-between; align-items:center;">
            <span><b>${p.nome}</b><br><small>Meta Total: R$ ${total.toFixed(2)}</small></span>
            <button class="blue" onclick="abrirProjeto(${i})">Abrir</button>
        </div>`;
    });
    document.getElementById('listaProjetos').innerHTML = h || '<p style="opacity:0.5">Nenhum projeto ativo.</p>';
}

function criarProjeto() {
    let n = document.getElementById('p_nome').value;
    let d = document.getElementById('p_dias').value;
    if (!n || !d) return;
    dados.push({ nome: n, dias: Number(d), historico: [], contas: [] });
    salvar(); renderHome();
    document.getElementById('p_nome').value = ''; document.getElementById('p_dias').value = '';
}

function abrirProjeto(i) {
    atual = i;
    document.getElementById('homePage').style.display = 'none';
    document.getElementById('projetoPage').style.display = 'block';
    switchTab(1);
    atualizar();
}

function voltar() {
    document.getElementById('homePage').style.display = 'block';
    document.getElementById('projetoPage').style.display = 'none';
    renderHome();
}

function switchTab(n) {
    document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.nav-tabs button').forEach(b => b.classList.remove('active'));
    document.getElementById('tab'+n).classList.add('active');
    document.getElementById('t'+n).classList.add('active');
}

// --- LÓGICA CENTRAL ---
function atualizar() {
    let p = dados[atual];
    let totalLiq = p.historico.reduce((s, v) => s + v, 0);
    
    // Separar Ativas vs Quitadas (Margem de 1 centavo para erro de arredondamento)
    let ativas = p.contas.filter(c => (totalLiq * (c.perc/100)) < (c.valor - 0.01));
    let quitadas = p.contas.filter(c => (totalLiq * (c.perc/100)) >= (c.valor - 0.01));

    let faltaPagar = ativas.reduce((s, c) => s + (c.valor - (totalLiq * (c.perc/100))), 0);
    let metaTotalOriginal = p.contas.reduce((s, c) => s + c.valor, 0);
    let diasRest = Math.max(1, p.dias - p.historico.length);

    // Cálculos de Sugestão
    let diariaLiq = faltaPagar / diasRest;
    let brutaSug = (diariaLiq + GASOLINA) / 0.9;

    // Atualiza Painel
    document.getElementById('displayNome').innerText = p.nome;
    document.getElementById('displayDias').innerText = `Dia ${p.historico.length} de ${p.dias} | Falta quitar: R$ ${faltaPagar.toFixed(2)}`;
    document.getElementById('metaLiq').innerText = `R$ ${diariaLiq.toFixed(2)}`;
    document.getElementById('metaBruta').innerText = `R$ ${brutaSug.toFixed(2)}`;
    document.getElementById('metaDizimo').innerText = `R$ ${(brutaSug * 0.1).toFixed(2)}`;

    // Ritmo
    let esperado = (metaTotalOriginal / p.dias) * p.historico.length;
    let r = document.getElementById('ritmoAlert');
    r.innerText = totalLiq < esperado ? "⚠️ Você está abaixo do ritmo planejado!" : "🔥 Ritmo excelente! Continue assim.";
    r.className = "alert " + (totalLiq < esperado ? "red" : "green");

    renderListas(ativas, quitadas, totalLiq, diariaLiq);
    renderEditor(p.contas);
    renderHistorico(p.historico);
}

function registrarGanho() {
    let p = dados[atual];
    let b = Number(document.getElementById('inputBruto').value);
    if (!b || b <= 0) return;

    let d = b * 0.1;
    let l = (b - d) - GASOLINA;

    let msg = `RESUMO DO REGISTRO:\n\n` +
              `🙏 Dízimo (10%): R$ ${d.toFixed(2)}\n` +
              `⛽ Gasolina: R$ 75,00\n` +
              `---------------------------\n` +
              `💰 LÍQUIDO CONTAS: R$ ${l.toFixed(2)}\n\n` +
              `Confirmar entrada?`;

    if (confirm(msg)) {
        p.historico.push(l);
        document.getElementById('inputBruto').value = '';
        salvar(); atualizar();
    }
}

function renderListas(ativas, quitadas, totalLiq, diariaLiq) {
    let hA = '';
    ativas.forEach(c => {
        let jaTem = totalLiq * (c.perc/100);
        hA += `<div class="row">
            <span>${c.nome} <small>(${c.perc}%)</small></span>
            <b>R$ ${(diariaLiq * (c.perc/100)).toFixed(2)}</b>
        </div>`;
    });
    document.getElementById('divisaoHoje').innerHTML = hA || '<p>Nenhuma conta ativa.</p>';

    let hQ = '';
    quitadas.forEach(c => {
        hQ += `<div class="quitada-item">✅ <b>${c.nome}</b> (R$ ${c.valor.toFixed(2)})</div>`;
    });
    document.getElementById('listaQuitadas').innerHTML = hQ || '<p style="opacity:0.5">Nenhuma conta quitada.</p>';
}

function renderEditor(contas) {
    let h = '';
    let soma = 0;
    contas.forEach((c) => {
        soma += Number(c.perc);
        h += `<div class="item-edit">
            <input value="${c.nome}" onchange="editConta(${c.id}, 'nome', this.value)" style="width:40%">
            <input type="number" value="${c.valor}" onchange="editConta(${c.id}, 'valor', this.value)" style="width:25%">
            <input type="number" value="${c.perc}" onchange="editConta(${c.id}, 'perc', this.value)" style="width:15%"> %
            <button class="red" onclick="removerConta(${c.id})" style="padding:5px 10px;">🗑</button>
        </div>`;
    });
    document.getElementById('editorContas').innerHTML = h + `<p style="color:${soma != 100 ? 'var(--red)' : 'var(--green)'}">Soma das Porcentagens: ${soma.toFixed(1)}%</p>`;
}

function renderHistorico(hist) {
    let h = '';
    hist.slice().reverse().forEach((v, i) => {
        let realIdx = hist.length - 1 - i;
        h += `<div class="row">
            <span>Dia ${realIdx+1}: + R$ ${v.toFixed(2)}</span>
            <button class="red" onclick="removerGanho(${realIdx})" style="padding:2px 8px; font-size:10px;">Apagar</button>
        </div>`;
    });
    document.getElementById('listaHistorico').innerHTML = h || '<p>Sem registros.</p>';
}

// --- FUNÇÕES DE EDIÇÃO ---
function addConta() {
    let n = document.getElementById('c_nome').value;
    let v = document.getElementById('c_valor').value;
    if (!n || !v) return;
    dados[atual].contas.push({ id: Date.now(), nome: n, valor: Number(v), perc: 0 });
    document.getElementById('c_nome').value = ''; document.getElementById('c_valor').value = '';
    salvar(); atualizar();
}

function editConta(id, campo, valor) {
    let c = dados[atual].contas.find(x => x.id === id);
    c[campo] = campo === 'nome' ? valor : Number(valor);
    salvar(); atualizar();
}

function removerConta(id) {
    if (confirm("Excluir esta conta permanentemente?")) {
        dados[atual].contas = dados[atual].contas.filter(c => c.id !== id);
        salvar(); atualizar();
    }
}

function removerGanho(idx) {
    if (confirm("Apagar esse ganho? Os saldos serão recalculados.")) {
        dados[atual].historico.splice(idx, 1);
        salvar(); atualizar();
    }
}

function bolaDeNeve() {
    let p = dados[atual];
    let totalLiq = p.historico.reduce((s, v) => s + v, 0);
    let ativas = p.contas.filter(c => (totalLiq * (c.perc/100)) < (c.valor - 0.01));
    
    if (ativas.length === 0) return;
    
    // Ordena pela que falta menos dinheiro para quitar
    ativas.sort((a, b) => (a.valor - (totalLiq * (a.perc/100))) - (b.valor - (totalLiq * (b.perc/100))));
    
    p.contas.forEach(c => c.perc = 0); // Reseta todas
    
    // Atribui prioridade: 70% na primeira, 20% na segunda, 10% no resto
    ativas.forEach((c, i) => {
        if (i === 0) c.perc = 70;
        else if (i === 1) c.perc = 20;
        else c.perc = 10 / (ativas.length - 2 || 1);
    });
    
    salvar(); atualizar();
}

function excluirProjetoTotal() {
    if (confirm("⚠️ PERIGO: Isso apagará o projeto e todo o histórico. Confirmar?")) {
        dados.splice(atual, 1);
        salvar(); voltar();
    }
}

// Inicializar
renderHome();
</script>

</body>
</html>
