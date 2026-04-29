<style>
    :root { 
        --bg: #0f172a; --card: #1e293b; --text: #f8fafc; 
        --green: #22c55e; --blue: #3b82f6; --red: #ef4444; 
        --orange: #f59e0b; --purple: #a855f7; --slate: #475569;
    }
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: var(--bg); color: var(--text); margin: 0; padding: 15px; padding-bottom: 90px; line-height: 1.4; }
    .card { background: var(--card); padding: 18px; border-radius: 16px; margin-bottom: 15px; border: 1px solid #334155; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.3); }
    
    input { background: #0f172a; border: 1px solid var(--slate); color: white; padding: 12px; border-radius: 8px; width: calc(100% - 26px); margin-bottom: 10px; font-size: 16px; }
    button { padding: 12px; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; transition: 0.2s; display: inline-flex; align-items: center; justify-content: center; gap: 5px; }
    button:active { transform: scale(0.98); }
    
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
    .quitada { opacity: 0.6; border-left-color: var(--green); }
    
    .nav-bar { position: fixed; bottom: 0; left: 0; width: 100%; background: #1e293b; display: flex; justify-content: space-around; padding: 10px 0; border-top: 2px solid #334155; z-index: 100; }
    .nav-item { color: #94a3b8; background: none; border: none; display: flex; flex-direction: column; align-items: center; font-size: 11px; }
    .nav-item.active { color: var(--blue); }

    .historico-lista { max-height: 200px; overflow-y: auto; font-size: 11px; margin-top: 10px; }
    .historico-item { background: #0f172a; padding: 8px; border-radius: 6px; margin-bottom: 5px; border: 1px solid #334155; }
</style>

<div id="homePage">
    <h1>📊 Minhas Metas</h1>
    <div id="listaMetas"></div>
    <div class="card">
        <h3>✨ Nova Meta</h3>
        <input id="nomeMeta" placeholder="Ex: Contas de Maio">
        <input id="diasMeta" type="number" placeholder="Prazo em dias">
        <button class="blue btn-full" onclick="criarMeta()">Criar Meta</button>
    </div>
</div>

<div id="metaPage" style="display:none;">
    <button class="gray" onclick="navegar('home')" style="margin-bottom:15px;">⬅ Voltar</button>

    <div class="card">
        <h2 id="tituloMeta"></h2>
        <div class="stat-box" style="margin-top:10px; border-color: var(--green);">
            <small>Saldo Disponível para Aporte</small><br>
            <span id="txtSaldoDisponivel" style="font-size: 1.8em; font-weight: bold; color: var(--green);">R$ 0,00</span>
        </div>
        <div class="stats-grid">
            <div class="stat-box"><small>Meta Total</small><br><b id="txtMetaTotal">R$ 0,00</b></div>
            <div class="stat-box"><small>Meta Bruta Diária</small><br><b id="txtDiaria" style="color:var(--orange)">R$ 0,00</b></div>
        </div>
    </div>

    <div class="card">
        <h3>💰 Receber Ganho Bruto</h3>
        <div class="flex">
            <input id="ganhoInput" type="number" placeholder="Valor Bruto" style="margin:0;">
            <button class="green" onclick="registrarGanho()">Receber</button>
        </div>
    </div>

    <div class="card">
        <h3>📋 Gestão de Despesas</h3>
        <div class="flex">
            <input id="nomeDesp" placeholder="Conta" style="width:50%; margin:0;">
            <input id="valorDesp" type="number" placeholder="Total" style="width:30%; margin:0;">
            <button class="blue" onclick="adicionarDespesa()">+</button>
        </div>
        <div id="listaDespesas" style="margin-top:15px;"></div>
    </div>

    <div class="card">
        <h3>📜 Histórico de Ganhos (Bruto → Líquido)</h3>
        <div id="historicoGanhos" class="historico-lista"></div>
    </div>
</div>

<script>
let metas = JSON.parse(localStorage.getItem('metas')) || [];
let config = JSON.parse(localStorage.getItem('config_financeiro')) || { gasolina: 75, dizimo: 10 };
let metaAtual = null;

function gravar() { localStorage.setItem('metas', JSON.stringify(metas)); }

function navegar(aba) {
    document.getElementById('homePage').style.display = aba === 'home' ? 'block' : 'none';
    document.getElementById('metaPage').style.display = aba === 'meta' ? 'block' : 'none';
    if(aba === 'home') renderHome();
}

function criarMeta() {
    const n = document.getElementById('nomeMeta').value;
    if(!n) return;
    metas.push({ nome: n, dias: Number(document.getElementById('diasMeta').value)||30, saldoLivre: 0, despesas: [], ganhos: [] });
    gravar(); renderHome();
    document.getElementById('nomeMeta').value = '';
}

function renderHome() {
    let h = '';
    metas.forEach((m, i) => {
        const totalPago = m.despesas.reduce((s, d) => s + (d.pagamentos?.reduce((a,b)=>a+b.valor,0)||0), 0);
        const metaV = m.despesas.reduce((s, d) => s + d.total, 0);
        h += `<div class="card flex" onclick="abrirMeta(${i})">
            <div><b>${m.nome}</b><br><small>R$ ${totalPago.toFixed(2)} / R$ ${metaV.toFixed(2)}</small></div>
            <b style="color:var(--blue)">${metaV>0?((totalPago/metaV)*100).toFixed(0):0}%</b>
        </div>`;
    });
    document.getElementById('listaMetas').innerHTML = h || 'Nenhuma meta.';
}

function abrirMeta(i) { metaAtual = i; navegar('meta'); renderMeta(); }

function registrarGanho() {
    const val = Number(document.getElementById('ganhoInput').value);
    if(val <= 0) return;
    const m = metas[metaAtual];
    const liq = (val - config.gasolina) * (1 - config.dizimo/100);
    m.ganhos.push({ bruto: val, liquido: liq, data: new Date().toLocaleDateString('pt-BR') });
    m.saldoLivre += liq;
    document.getElementById('ganhoInput').value = '';
    gravar(); renderMeta();
}

function adicionarDespesa() {
    const n = document.getElementById('nomeDesp'), v = document.getElementById('valorDesp');
    if(!n.value || !v.value) return;
    metas[metaAtual].despesas.push({ nome: n.value, total: Number(v.value), pagamentos: [], quitada: false });
    n.value = ''; v.value = '';
    gravar(); renderMeta();
}

function aportar(idx) {
    const m = metas[metaAtual];
    const d = m.despesas[idx];
    const falta = d.total - d.pagamentos.reduce((s,p)=>s+p.valor, 0);
    const valor = prompt(`Quanto deseja aportar em ${d.nome}?\nSaldo disponível: R$ ${m.saldoLivre.toFixed(2)}`, Math.min(m.saldoLivre, falta).toFixed(2));
    
    if(valor && Number(valor) <= m.saldoLivre && Number(valor) > 0) {
        const v = Number(valor);
        d.pagamentos.push({ valor: v, data: new Date().toLocaleDateString('pt-BR') });
        m.saldoLivre -= v;
        if(d.pagamentos.reduce((s,p)=>s+p.valor,0) >= d.total) d.quitada = true;
        gravar(); renderMeta();
    } else if (Number(valor) > m.saldoLivre) {
        alert("Saldo insuficiente!");
    }
}

function removerDespesa(idx) {
    if(confirm("Excluir conta?")) {
        const d = metas[metaAtual].despesas[idx];
        metas[metaAtual].saldoLivre += d.pagamentos.reduce((s,p)=>s+p.valor, 0); // Devolve o dinheiro para o saldo livre
        metas[metaAtual].despesas.splice(idx, 1);
        gravar(); renderMeta();
    }
}

function renderMeta() {
    const m = metas[metaAtual];
    const metaTotal = m.despesas.reduce((s,d)=>s+d.total, 0);
    const totalAportado = m.despesas.reduce((s,d)=>s+d.pagamentos.reduce((a,b)=>a+b.valor,0), 0);
    const faltaTotal = Math.max(0, metaTotal - totalAportado - m.saldoLivre);
    
    const diasR = Math.max(1, m.dias - m.ganhos.length);
    const diariaLiq = faltaTotal / diasR;
    const diariaBruta = (diariaLiq / (1 - config.dizimo/100)) + config.gasolina;

    document.getElementById('tituloMeta').innerText = m.nome;
    document.getElementById('txtSaldoDisponivel').innerText = `R$ ${m.saldoLivre.toFixed(2)}`;
    document.getElementById('txtMetaTotal').innerText = `R$ ${metaTotal.toFixed(2)}`;
    document.getElementById('txtDiaria').innerText = `R$ ${diariaBruta.toFixed(2)}`;

    let hD = '';
    m.despesas.forEach((d, i) => {
        const pago = d.pagamentos.reduce((s,p)=>s+p.valor, 0);
        hD += `<div class="despesa-item ${d.quitada?'quitada':''}">
            <div class="flex"><b>${d.nome}</b> <span>${d.quitada?'✅':'⏳'}</span></div>
            <div class="flex" style="margin:5px 0"><small>Total: R$ ${d.total.toFixed(2)}</small> <b>Pago: R$ ${pago.toFixed(2)}</b></div>
            
            <div class="historico-lista">
                ${d.pagamentos.map(p => `<div class="historico-item flex"><span>${p.data}</span> <b>+ R$ ${p.valor.toFixed(2)}</b></div>`).join('') || '<small>Sem aportes</small>'}
            </div>

            <div class="flex" style="margin-top:10px;">
                ${!d.quitada ? `<button class="green" onclick="aportar(${i})" style="flex:1">Aportar</button>` : ''}
                <button class="red" onclick="removerDespesa(${i})">🗑️</button>
            </div>
        </div>`;
    });
    document.getElementById('listaDespesas').innerHTML = hD || 'Sem despesas.';

    let hG = '';
    [...m.ganhos].reverse().forEach(g => {
        hG += `<div class="historico-item flex">
            <span>${g.data}</span>
            <span>B: ${g.bruto.toFixed(2)} → <b style="color:var(--green)">L: ${g.liquido.toFixed(2)}</b></span>
        </div>`;
    });
    document.getElementById('historicoGanhos').innerHTML = hG || 'Sem ganhos.';
}

renderHome();
</script>
