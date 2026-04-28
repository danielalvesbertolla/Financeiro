<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financeiro PRO - Versão Definitiva</title>
    <style>
        :root {
            --bg: #0f172a;
            --card: #1e293b;
            --border: #334155;
            --green: #22c55e;
            --red: #ef4444;
            --blue: #3b82f6;
            --orange: #f59e0b;
        }

        * {
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', sans-serif;
            background: var(--bg);
            color: white;
            margin: 0;
            padding: 10px;
        }

        .container {
            max-width: 650px;
            margin: auto;
            padding-bottom: 50px;
        }

        .page {
            animation: fadeIn 0.3s;
        }

        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }

        .card {
            background: var(--card);
            padding: 18px;
            border-radius: 12px;
            margin-bottom: 15px;
            border: 1px solid var(--border);
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
        }

        h1, h2, h3, h4 {
            margin-top: 0;
        }

        button {
            padding: 10px 14px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
            margin: 2px;
            font-size: 1em;
        }

        button:hover {
            filter: brightness(1.2);
        }

        button:active {
            transform: scale(0.98);
        }

        input {
            padding: 10px;
            border-radius: 8px;
            border: 1px solid var(--border);
            background: #0f172a;
            color: white;
            margin: 5px 0;
            width: 100%;
            font-size: 1em;
        }

        input:focus {
            outline: none;
            border-color: var(--blue);
            box-shadow: 0 0 8px rgba(59, 130, 246, 0.3);
        }

        form {
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .green { background: var(--green); color: black; }
        .red { background: var(--red); }
        .blue { background: var(--blue); }
        .orange { background: var(--orange); color: black; }
        .gray { background: #64748b; }

        .finance-box {
            background: rgba(0, 0, 0, 0.3);
            padding: 15px;
            border-radius: 8px;
            border: 1px solid var(--blue);
            margin: 15px 0;
        }

        .row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px 0;
            border-bottom: 1px dashed var(--border);
        }

        .row:last-child {
            border-bottom: none;
            font-size: 1.1em;
            font-weight: bold;
        }

        .nav-tabs {
            display: flex;
            gap: 5px;
            margin-bottom: 15px;
            position: sticky;
            top: 0;
            z-index: 10;
            background: var(--bg);
            padding: 5px 0;
        }

        .nav-tabs button {
            flex: 1;
            border-radius: 8px;
            opacity: 0.6;
            transition: opacity 0.2s;
        }

        .nav-tabs button.active {
            opacity: 1;
            border-bottom: 3px solid white;
            background: var(--card);
        }

        .tab-content {
            display: none;
        }

        .tab-content.active {
            display: block;
            animation: fadeIn 0.3s;
        }

        .item-edit {
            background: rgba(255, 255, 255, 0.03);
            padding: 12px;
            border-radius: 10px;
            margin-bottom: 10px;
            border: 1px solid var(--border);
            display: flex;
            gap: 8px;
            align-items: center;
            flex-wrap: wrap;
        }

        .item-edit input {
            margin: 0;
        }

        .quitada-item {
            background: #064e3b;
            border-left: 6px solid var(--green);
            padding: 10px;
            border-radius: 8px;
            margin-bottom: 8px;
            opacity: 0.8;
        }

        .alert {
            padding: 10px;
            border-radius: 8px;
            text-align: center;
            font-weight: bold;
            margin: 10px 0;
        }

        .alert.red {
            background: rgba(239, 68, 68, 0.2);
            border: 1px solid var(--red);
        }

        .alert.green {
            background: rgba(34, 197, 94, 0.2);
            border: 1px solid var(--green);
        }

        hr {
            border-top: 1px solid var(--border);
            margin: 20px 0;
            border: none;
        }

        .status-text {
            font-size: 0.9em;
            opacity: 0.7;
            color: var(--orange);
            font-weight: 500;
        }

        @media (max-width: 480px) {
            .container {
                max-width: 100%;
            }

            .item-edit {
                flex-direction: column;
            }

            .item-edit input {
                width: 100%;
            }
        }
    </style>
</head>
<body>

<div class="container">
    <!-- HOME PAGE -->
    <div id="homePage" class="page">
        <h1>📊 Meus Projetos</h1>
        <div class="card">
            <div id="listaProjetos"></div>
            <hr>
            <h3>Novo Projeto de Quitação</h3>
            <form id="formNovoProjeto" onsubmit="App.criarProjeto(event)">
                <input id="p_nome" placeholder="Ex: Abril 2026" required>
                <input id="p_dias" type="number" placeholder="Quantos dias planeja?" required min="1">
                <button type="submit" class="green" style="width: 100%; margin-top: 10px;">Começar Projeto</button>
            </form>
        </div>
    </div>

    <!-- PROJETO PAGE -->
    <div id="projetoPage" class="page" style="display:none;">
        <div class="nav-tabs">
            <button id="t1" class="active" onclick="App.switchTab(1)">Painel</button>
            <button id="t2" onclick="App.switchTab(2)">Contas/Edit</button>
            <button id="t3" onclick="App.switchTab(3)">Histórico</button>
            <button class="red" onclick="App.voltar()" style="flex:0.3">✖</button>
        </div>

        <!-- TAB 1: PAINEL -->
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
                    <form onsubmit="App.registrarGanho(event)">
                        <input id="inputBruto" type="number" placeholder="R$ 0,00" required min="0" step="0.01">
                        <button type="submit" class="green" style="width: 100%; margin-top: 10px; font-size: 1.1em;">Confirmar Recebimento</button>
                    </form>
                </div>
            </div>

            <div class="card">
                <h3>💰 Divisão do Valor de Hoje</h3>
                <div id="divisaoHoje"></div>
            </div>
        </div>

        <!-- TAB 2: CONTAS -->
        <div id="tab2" class="tab-content">
            <div class="card">
                <button class="orange" onclick="App.bolaDeNeve()" style="width: 100%; margin-bottom: 15px;">🚀 Priorização Bola de Neve (Automática)</button>
                <div id="editorContas"></div>
                
                <hr>
                <h4>Adicionar Nova Conta</h4>
                <form id="formNovaConta" onsubmit="App.addConta(event)">
                    <input id="c_nome" placeholder="Nome" required>
                    <input id="c_valor" type="number" placeholder="Valor R$" required min="0" step="0.01">
                    <button type="submit" class="blue" style="width: 100%;">Adicionar à Lista</button>
                </form>
            </div>

            <div class="card">
                <h3>✅ Contas Quitadas</h3>
                <div id="listaQuitadas"></div>
            </div>
        </div>

        <!-- TAB 3: HISTÓRICO -->
        <div id="tab3" class="tab-content">
            <div class="card">
                <h3>📜 Histórico de Ganhos</h3>
                <p><small>Apague registros específicos para corrigir o saldo:</small></p>
                <div id="listaHistorico"></div>
            </div>
            <button class="red" onclick="App.excluirProjetoTotal()" style="width: 100%; margin-top: 20px; opacity: 0.5;">🗑 Excluir Projeto Inteiro</button>
        </div>
    </div>
</div>

<script>
/**
 * Aplicação de Gestão Financeira
 * Versão refatorada com melhor arquitetura
 */

class FinanceApp {
    constructor() {
        this.STORAGE_KEY = 'financas_ultra_v1';
        this.GASOLINA = 75;
        this.DIZIMO_PERC = 0.1;
        this.dados = this.loadData();
        this.atual = null;
    }

    // ===== STORAGE =====
    loadData() {
        try {
            return JSON.parse(localStorage.getItem(this.STORAGE_KEY)) || [];
        } catch (error) {
            console.error('Erro ao carregar dados:', error);
            return [];
        }
    }

    saveData() {
        try {
            localStorage.setItem(this.STORAGE_KEY, JSON.stringify(this.dados));
        } catch (error) {
            console.error('Erro ao salvar dados:', error);
            alert('Erro ao salvar dados. Verifique o espaço disponível.');
        }
    }

    // ===== VALIDAÇÃO =====
    validateProjeto(nome, dias) {
        if (!nome?.trim()) throw new Error('Nome do projeto é obrigatório');
        if (!dias || dias < 1) throw new Error('Número de dias deve ser maior que 0');
        return true;
    }

    validateConta(nome, valor) {
        if (!nome?.trim()) throw new Error('Nome da conta é obrigatório');
        if (!valor || valor <= 0) throw new Error('Valor deve ser maior que 0');
        return true;
    }

    validateGanho(bruto) {
        if (!bruto || bruto <= 0) throw new Error('Valor bruto deve ser maior que 0');
        return true;
    }

    // ===== NAVEGAÇÃO =====
    renderHome() {
        const html = this.dados
            .map((p, i) => {
                const total = p.contas.reduce((s, c) => s + c.valor, 0);
                return `
                    <div class="card" style="display:flex; justify-content:space-between; align-items:center;">
                        <div>
                            <b>${this.escapeHtml(p.nome)}</b><br>
                            <small>Meta Total: R$ ${total.toFixed(2)}</small>
                        </div>
                        <button class="blue" onclick="App.abrirProjeto(${i})">Abrir</button>
                    </div>
                `;
            })
            .join('') || '<p style="opacity:0.5">Nenhum projeto ativo.</p>';

        document.getElementById('listaProjetos').innerHTML = html;
    }

    criarProjeto(event) {
        event.preventDefault();
        try {
            const nome = document.getElementById('p_nome').value;
            const dias = Number(document.getElementById('p_dias').value);

            this.validateProjeto(nome, dias);

            this.dados.push({
                nome,
                dias,
                historico: [],
                contas: [],
                criado: new Date().toISOString()
            });

            this.saveData();
            this.renderHome();
            document.getElementById('formNovoProjeto').reset();
        } catch (error) {
            alert(`Erro: ${error.message}`);
        }
    }

    abrirProjeto(i) {
        if (i < 0 || i >= this.dados.length) {
            alert('Projeto não encontrado');
            return;
        }
        this.atual = i;
        this.showPage('projetoPage');
        this.switchTab(1);
        this.atualizar();
    }

    voltar() {
        this.atual = null;
        this.showPage('homePage');
        this.renderHome();
    }

    showPage(pageId) {
        document.querySelectorAll('.page').forEach(p => p.style.display = 'none');
        document.getElementById(pageId).style.display = 'block';
    }

    switchTab(n) {
        document.querySelectorAll('.tab-content').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.nav-tabs button').forEach(b => b.classList.remove('active'));
        document.getElementById(`tab${n}`)?.classList.add('active');
        document.getElementById(`t${n}`)?.classList.add('active');
    }

    // ===== LÓGICA CENTRAL =====
    atualizar() {
        const p = this.dados[this.atual];
        if (!p) return;

        const totalLiq = p.historico.reduce((s, v) => s + v, 0);
        const ativas = p.contas.filter(c => this.calcPago(c, totalLiq) < c.valor - 0.01);
        const quitadas = p.contas.filter(c => this.calcPago(c, totalLiq) >= c.valor - 0.01);

        const faltaPagar = ativas.reduce((s, c) => s + (c.valor - this.calcPago(c, totalLiq)), 0);
        const metaTotalOriginal = p.contas.reduce((s, c) => s + c.valor, 0);
        const diasRest = Math.max(1, p.dias - p.historico.length);

        const { diariaLiq, brutaSug } = this.calcularMetas(faltaPagar, diasRest);

        this.renderPainel(p, totalLiq, faltaPagar, diariaLiq, brutaSug, metaTotalOriginal);
        this.renderListas(ativas, quitadas, totalLiq, diariaLiq);
        this.renderEditor(p.contas);
        this.renderHistorico(p.historico);
    }

    calcPago(conta, totalLiq) {
        return totalLiq * (conta.perc / 100);
    }

    calcularMetas(faltaPagar, diasRest) {
        const diariaLiq = faltaPagar / diasRest;
        const brutaSug = (diariaLiq + this.GASOLINA) / (1 - this.DIZIMO_PERC);
        return { diariaLiq, brutaSug };
    }

    renderPainel(p, totalLiq, faltaPagar, diariaLiq, brutaSug, metaTotalOriginal) {
        document.getElementById('displayNome').textContent = p.nome;
        document.getElementById('displayDias').innerHTML = 
            `Dia ${p.historico.length} de ${p.dias} | Falta quitar: <span class="status-text">R$ ${faltaPagar.toFixed(2)}</span>`;
        
        document.getElementById('metaLiq').textContent = `R$ ${diariaLiq.toFixed(2)}`;
        document.getElementById('metaBruta').textContent = `R$ ${brutaSug.toFixed(2)}`;
        document.getElementById('metaDizimo').textContent = `R$ ${(brutaSug * this.DIZIMO_PERC).toFixed(2)}`;

        this.renderRitmo(totalLiq, metaTotalOriginal, p);
    }

    renderRitmo(totalLiq, metaTotalOriginal, p) {
        const esperado = (metaTotalOriginal / p.dias) * p.historico.length;
        const ritmoElement = document.getElementById('ritmoAlert');
        const estaAbaixo = totalLiq < esperado;

        ritmoElement.textContent = estaAbaixo 
            ? "⚠️ Você está abaixo do ritmo planejado!" 
            : "🔥 Ritmo excelente! Continue assim.";
        
        ritmoElement.className = `alert ${estaAbaixo ? 'red' : 'green'}`;
    }

    registrarGanho(event) {
        event.preventDefault();
        try {
            const bruto = Number(document.getElementById('inputBruto').value);
            this.validateGanho(bruto);

            const dizimo = bruto * this.DIZIMO_PERC;
            const liquido = (bruto - dizimo) - this.GASOLINA;

            const confirmacao = this.gerarMensagemConfirmacao(bruto, dizimo, liquido);

            if (confirm(confirmacao)) {
                this.dados[this.atual].historico.push(liquido);
                document.getElementById('inputBruto').value = '';
                this.saveData();
                this.atualizar();
            }
        } catch (error) {
            alert(`Erro: ${error.message}`);
        }
    }

    gerarMensagemConfirmacao(bruto, dizimo, liquido) {
        return `RESUMO DO REGISTRO:\n\n` +
               `🙏 Dízimo (10%): R$ ${dizimo.toFixed(2)}\n` +
               `⛽ Gasolina: R$ ${this.GASOLINA.toFixed(2)}\n` +
               `---------------------------\n` +
               `💰 LÍQUIDO CONTAS: R$ ${liquido.toFixed(2)}\n\n` +
               `Confirmar entrada?`;
    }

    renderListas(ativas, quitadas, totalLiq, diariaLiq) {
        const divisaoHtml = ativas
            .map(c => {
                const valor = diariaLiq * (c.perc / 100);
                return `
                    <div class="row">
                        <span>${this.escapeHtml(c.nome)} <small>(${c.perc}%)</small></span>
                        <b>R$ ${valor.toFixed(2)}</b>
                    </div>
                `;
            })
            .join('') || '<p>Nenhuma conta ativa.</p>';

        document.getElementById('divisaoHoje').innerHTML = divisaoHtml;

        const quitadasHtml = quitadas
            .map(c => `
                <div class="quitada-item">✅ <b>${this.escapeHtml(c.nome)}</b> (R$ ${c.valor.toFixed(2)})</div>
            `)
            .join('') || '<p style="opacity:0.5">Nenhuma conta quitada.</p>';

        document.getElementById('listaQuitadas').innerHTML = quitadasHtml;
    }

    renderEditor(contas) {
        const html = contas.map(c => `
            <div class="item-edit">
                <input value="${this.escapeHtml(c.nome)}" 
                       onchange="App.editConta(${c.id}, 'nome', this.value)" 
                       placeholder="Nome da conta"
                       style="flex: 2; min-width: 100px;">
                <input type="number" value="${c.valor}" 
                       onchange="App.editConta(${c.id}, 'valor', this.value)" 
                       placeholder="Valor"
                       style="flex: 1; min-width: 70px;"
                       step="0.01">
                <input type="number" value="${c.perc}" 
                       onchange="App.editConta(${c.id}, 'perc', this.value)" 
                       placeholder="%"
                       style="flex: 0.8; min-width: 50px;"
                       min="0" max="100">
                <span>%</span>
                <button class="red" onclick="App.removerConta(${c.id})" style="padding:5px 10px; flex: 0.5;">🗑</button>
            </div>
        `).join('');

        const somaPerc = contas.reduce((s, c) => s + c.perc, 0);
        const statusCor = somaPerc !== 100 ? 'var(--red)' : 'var(--green)';
        const statusTexto = somaPerc !== 100 ? '⚠️' : '✅';

        document.getElementById('editorContas').innerHTML = 
            html + `<p style="color:${statusCor}; text-align:center; font-weight:bold;">${statusTexto} Soma das Porcentagens: ${somaPerc.toFixed(1)}%</p>`;
    }

    renderHistorico(historico) {
        const html = historico
            .map((v, i) => {
                const diaNum = i + 1;
                return `
                    <div class="row">
                        <span>Dia ${diaNum}: + R$ ${v.toFixed(2)}</span>
                        <button class="red" onclick="App.removerGanho(${i})" style="padding:2px 8px; font-size:10px;">Apagar</button>
                    </div>
                `;
            })
            .reverse()
            .join('') || '<p>Sem registros.</p>';

        document.getElementById('listaHistorico').innerHTML = html;
    }

    // ===== EDIÇÃO =====
    addConta(event) {
        event.preventDefault();
        try {
            const nome = document.getElementById('c_nome').value;
            const valor = Number(document.getElementById('c_valor').value);

            this.validateConta(nome, valor);

            this.dados[this.atual].contas.push({
                id: Date.now(),
                nome,
                valor,
                perc: 0
            });

            document.getElementById('formNovaConta').reset();
            this.saveData();
            this.atualizar();
        } catch (error) {
            alert(`Erro: ${error.message}`);
        }
    }

    editConta(id, campo, valor) {
        try {
            const conta = this.dados[this.atual].contas.find(c => c.id === id);
            if (!conta) throw new Error('Conta não encontrada');

            if (campo === 'valor') {
                const numVal = Number(valor);
                if (numVal <= 0) throw new Error('Valor deve ser maior que 0');
                conta[campo] = numVal;
            } else if (campo === 'perc') {
                const numVal = Number(valor);
                if (numVal < 0 || numVal > 100) throw new Error('Porcentagem deve estar entre 0 e 100');
                conta[campo] = numVal;
            } else {
                conta[campo] = valor;
            }

            this.saveData();
            this.atualizar();
        } catch (error) {
            alert(`Erro: ${error.message}`);
        }
    }

    removerConta(id) {
        if (confirm("Excluir esta conta permanentemente?")) {
            this.dados[this.atual].contas = this.dados[this.atual].contas.filter(c => c.id !== id);
            this.saveData();
            this.atualizar();
        }
    }

    removerGanho(idx) {
        if (confirm("Apagar esse ganho? Os saldos serão recalculados.")) {
            this.dados[this.atual].historico.splice(idx, 1);
            this.saveData();
            this.atualizar();
        }
    }

    bolaDeNeve() {
        try {
            const p = this.dados[this.atual];
            const totalLiq = p.historico.reduce((s, v) => s + v, 0);
            const ativas = p.contas.filter(c => this.calcPago(c, totalLiq) < c.valor - 0.01);

            if (ativas.length === 0) {
                alert('Nenhuma conta ativa para priorizar');
                return;
            }

            // Ordena pela que falta menos dinheiro para quitar
            ativas.sort((a, b) => 
                (a.valor - this.calcPago(a, totalLiq)) - (b.valor - this.calcPago(b, totalLiq))
            );

            p.contas.forEach(c => c.perc = 0);

            ativas.forEach((c, i) => {
                if (i === 0) c.perc = 70;
                else if (i === 1) c.perc = 20;
                else c.perc = 10 / (ativas.length - 2 || 1);
            });

            this.saveData();
            this.atualizar();
            alert('✅ Contas priorizadas com sucesso!');
        } catch (error) {
            alert(`Erro: ${error.message}`);
        }
    }

    excluirProjetoTotal() {
        if (confirm("⚠️ PERIGO: Isso apagará o projeto e todo o histórico. Confirmar?")) {
            this.dados.splice(this.atual, 1);
            this.saveData();
            this.voltar();
        }
    }

    // ===== UTILITÁRIOS =====
    escapeHtml(text) {
        const map = {
            '&': '&amp;',
            '<': '&lt;',
            '>': '&gt;',
            '"': '&quot;',
            "'": '&#039;'
        };
        return text?.replace(/[&<>"']/g, m => map[m]) || '';
    }
}

// Instanciar aplicação globalmente
const App = new FinanceApp();
App.renderHome();
</script>

</body>
</html>
