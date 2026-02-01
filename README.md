<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="apple-mobile-web-app-title" content="Gestão ASB ENG">
    <meta name="application-name" content="Gestão ASB ENG">
    <meta name="theme-color" content="#1a1a1a">
    <meta name="mobile-web-app-capable" content="yes">
    
    <title>Gestão ASB ENG - v106.0</title>
    
    <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-database-compat.js"></script>

    <style>
        :root {
            --asb-dark: #1a1a1a;
            --asb-blue: #0056b3;
            --asb-blue-grad: linear-gradient(135deg, #0056b3 0%, #003d7a 100%);
            --asb-success: #28a745;
            --asb-success-grad: linear-gradient(135deg, #28a745 0%, #1e7e34 100%);
            --asb-danger: #dc3545;
            --asb-warning: #ff9800;
            --asb-bg: #eef2f3;
            --shadow-soft: 0 4px 15px rgba(0,0,0,0.08);
        }
        body { font-family: 'Segoe UI', Roboto, Arial, sans-serif; background: var(--asb-bg); margin: 0; padding: 0; color: #333; }
        
        #login-screen { display: flex; justify-content: center; align-items: center; height: 100vh; background: var(--asb-dark); }
        .login-box { background: white; padding: 40px; border-radius: 12px; box-shadow: 0 15px 35px rgba(0,0,0,0.5); width: 320px; text-align: center; }
        
        #main-app { display: none; padding: 20px; }
        .container { max-width: 1200px; margin: auto; background: white; padding: 30px; border-radius: 10px; box-shadow: var(--shadow-soft); border-top: 10px solid var(--asb-dark); }
        
        header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #eee; padding-bottom: 20px; margin-bottom: 25px; }
        header h1 { margin: 0; font-size: 26px; letter-spacing: -1px; }
        header h1 span { color: var(--asb-blue); font-weight: 300; }
        
        .sync-badge { font-size: 10px; padding: 4px 8px; border-radius: 10px; background: #eee; color: #666; margin-left: 10px; vertical-align: middle; transition: 0.5s; }
        .sync-online { background: #d4edda !important; color: #155724 !important; border: 1px solid #c3e6cb !important; }
        .sync-offline { background: #f8d7da !important; color: #721c24 !important; }

        nav { display: flex; gap: 8px; background: #2d2d2d; padding: 8px; border-radius: 8px; margin-bottom: 25px; flex-wrap: wrap; box-shadow: inset 0 2px 5px rgba(0,0,0,0.2); }
        .nav-btn { padding: 12px 18px; cursor: pointer; border: none; background: transparent; color: #bbb; font-weight: 600; border-radius: 6px; transition: 0.3s; font-size: 13px; }
        .nav-btn:hover { color: white; background: rgba(255,255,255,0.1); }
        .nav-btn.active { background: var(--asb-blue-grad); color: white; box-shadow: 0 4px 12px rgba(0,86,179,0.3); }

        .section-panel { display: none; animation: fadeIn 0.3s ease; }
        .section-panel.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
        
        .search-hero { background: #f8f9fa; padding: 25px; border-radius: 10px; border: 1px solid #e0e4e8; margin-bottom: 20px; display: flex; gap: 12px; align-items: center; box-shadow: inset 0 2px 4px rgba(0,0,0,0.02); }
        .search-hero input { flex: 1; font-size: 16px; border: 2px solid #ddd; padding: 12px 20px; border-radius: 8px; transition: 0.3s; }

        .form-grid { display: none; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 15px; background: #fff; padding: 20px; border: 1px solid #eee; border-radius: 8px; margin-bottom: 20px; align-items: end; }
        .field-group { display: flex; flex-direction: column; gap: 6px; }
        .field-group label { font-size: 10px; font-weight: 800; color: #666; text-transform: uppercase; letter-spacing: 0.5px; }

        input, select { padding: 12px; border: 1px solid #ccc; border-radius: 6px; width: 100%; box-sizing: border-box; font-size: 14px; }
        
        table { width: 100%; border-collapse: collapse; margin-top: 15px; border: 1px solid #eee; border-radius: 8px; overflow: hidden; }
        th { background: #fdfdfd; padding: 8px 15px; text-align: left; border-bottom: 2px solid #eee; font-size: 11px; color: #888; text-transform: uppercase; }
        
        /* MUDANÇA 1: Espaçamento reduzido para compactar as linhas em todas as telas */
        td { padding: 2px 15px !important; border-bottom: 1px solid #f6f6f6; font-size: 13px; line-height: 1.1 !important; height: 24px; }
        
        .btn { padding: 10px 20px; border: none; border-radius: 6px; color: white; cursor: pointer; font-weight: bold; text-transform: uppercase; transition: 0.3s; height: 38px; display: inline-flex; align-items: center; justify-content: center; gap: 8px; font-size: 10px; }
        .btn-add { background: var(--asb-success-grad); }
        .btn-new { background: #6c757d; }
        .btn-edit { background: #ffc107; color: #000; padding: 5px 12px; height: 30px; }
        .btn-del { background: #e74c3c; padding: 5px 12px; height: 30px; }
        .btn-print { background: var(--asb-blue-grad); width: 100%; margin-top: 15px; height: 50px; font-size: 14px; }

        .edit-only { display: none; }
        .edit-highlight { background: #fff9c4 !important; border: 2px solid #fbc02d !important; }

        .summary-box { background: #f8f9fa; padding: 15px; margin-top: 15px; border-radius: 10px; border: 1px solid #eee; }
        .summary-row { display: flex; justify-content: space-between; padding: 3px 0; border-bottom: 1px solid #eee; font-size: 13px; }

        .temp-refresh-btn { background: var(--asb-success); color: white; border: none; border-radius: 4px; padding: 5px 10px; cursor: pointer; font-size: 10px; margin-left: 10px; }

        @media print {
            body { background: white !important; }
            nav, .no-print, .search-hero, header, .form-grid, .btn { display: none !important; }
            .container { box-shadow: none; border: none; width: 100% !important; margin: 0; padding: 0; }
            .section-panel { display: none !important; }
            #tab-orcamento.active { display: block !important; }
            table th:last-child, table td:last-child { display: none !important; }
            .summary-box { border: 1px solid #333; }
            td { padding: 1px 10px !important; font-size: 11px !important; height: 18px !important; }
        }
    </style>
</head>
<body>

<div id="login-screen">
    <div class="login-box">
        <h2 style="margin-bottom:30px;">ASB <span style="color:var(--asb-blue); font-weight:300;">SISTEMA</span></h2>
        <input type="text" id="user-input" placeholder="Usuário" style="margin-bottom:15px;">
        <input type="password" id="pass-input" placeholder="Senha" style="margin-bottom:25px;">
        <button class="btn btn-add" id="btn-login" style="width: 100%; height:50px;" onclick="handleLogin()">ACESSAR GESTÃO</button>
        <div id="login-status" style="font-size: 10px; color: #999; margin-top: 10px;">Verificando Nuvem...</div>
    </div>
</div>

<div id="main-app">
    <div class="container">
        <header>
            <div class="logo">
                <h1>ASB <span>AUTOMAÇÃO</span> <span id="sync-indicator" class="sync-badge">Conectando...</span></h1>
                <div style="font-size: 12px; color: #666; margin-top: 5px;">
                    Temperatura: <span id="temp-val">--°C</span>
                    <button class="temp-refresh-btn no-print" onclick="refreshTemperature()">ATUALIZAR ⚙️</button>
                </div>
            </div>
            <div class="no-print">
                <small id="welcome-msg" style="font-weight:bold; color:#666;"></small> 
                <button class="btn" style="background:#eee; color:#333; font-size:10px; height:30px; margin-left:15px;" onclick="location.reload()">SAIR</button>
            </div>
        </header>

        <nav class="no-print">
            <button class="nav-btn active" onclick="openTab('tab-estoque', this)">ESTOQUE</button>
            <button class="nav-btn" onclick="openTab('tab-clientes', this)">CLIENTES</button>
            <button class="nav-btn" onclick="openTab('tab-orcamento', this)">ORÇAMENTO</button>
            <button class="nav-btn" onclick="openTab('tab-relatorios', this)">HISTÓRICO</button>
            <button class="nav-btn" onclick="openTab('tab-movimentacao', this)">MOVIMENTAÇÃO</button>
            <button id="nav-master" class="nav-btn" style="display:none" onclick="openTab('tab-master', this)">USUÁRIOS</button>
            <button class="nav-btn" onclick="openTab('tab-config', this)">CONFIG</button>
        </nav>

        <section id="tab-estoque" class="section-panel active">
            <div class="search-hero">
                <button class="btn btn-new" onclick="mostrarFormEstoque()">NOVO PRODUTO</button>
                <input type="text" id="search-est-input" placeholder="🔍 Buscar produto..." onkeyup="render()">
                <button class="btn" style="background:var(--asb-dark)" onclick="render('all')">LISTAR TUDO</button>
                <button class="btn" style="background:var(--asb-blue)" onclick="window.print()">🖨️ IMPRIMIR LISTA</button>
            </div>
            <div class="form-grid" id="form-estoque">
                <div class="field-group" style="grid-column: span 2;"><label>Produto / Descrição</label><input type="text" id="est-desc"></div>
                <div class="field-group"><label>Unid.</label><input type="text" id="est-unid" placeholder="Ex: UN, PC, KG"></div>
                <div class="field-group"><label>NCM</label><input type="text" id="est-ncm"></div>
                <div class="field-group"><label>CFOP</label><input type="text" id="est-cfop"></div>
                <div class="field-group"><label>CST</label><input type="text" id="est-cst"></div>
                <div class="field-group"><label>IPI (%)</label><input type="number" id="est-ipi"></div>
                <div class="field-group"><label>ICMS (%)</label><input type="number" id="est-icms"></div>
                <div class="field-group edit-only"><label>Preço Compra (R$)</label><input type="number" id="est-compra" class="edit-highlight" oninput="calcularPrecoVenda()"></div>
                <div class="field-group edit-only"><label>Aumento (%)</label><input type="number" id="est-markup" class="edit-highlight" oninput="calcularPrecoVenda()"></div>
                <div class="field-group"><label>Preço Venda (R$)</label><input type="number" id="est-val"></div>
                <div class="field-group"><label>Qtd em Estoque</label><input type="number" id="est-qtd"></div>
                <div class="form-actions-column">
                    <button class="btn btn-add" id="btn-save-est" onclick="addEstoque()">SALVAR</button>
                    <button class="btn btn-new" onclick="cancelarEdicao()">CANCELAR</button>
                </div>
            </div>
            <table>
                <thead>
                    <tr><th>Descrição</th><th>Unid.</th><th>NCM</th><th>CFOP</th><th>Vlr. Venda</th><th>Estoque</th><th class="no-print">Ações</th></tr>
                </thead>
                <tbody id="tbl-estoque-corpo"></tbody>
            </table>
        </section>

        <section id="tab-clientes" class="section-panel">
            <div class="search-hero">
                <button class="btn btn-new" onclick="mostrarFormCliente()">NOVO CLIENTE</button>
                <input type="text" id="search-cli-input" placeholder="🔍 Buscar cliente..." onkeyup="render()">
                <button class="btn" style="background:var(--asb-dark)" onclick="render('all')">VER TODOS</button>
            </div>
            <div class="form-grid" id="form-cliente">
                <div class="field-group"><label>Nome Fantasia</label><input type="text" id="cli-nome"></div>
                <div class="field-group"><label>Razão Social</label><input type="text" id="cli-razao"></div>
                <div class="field-group"><label>Tipo</label><select id="cli-tipo"><option>Física</option><option>Jurídica</option></select></div>
                <div class="field-group"><label>CPF / CNPJ</label><input type="text" id="cli-doc"></div>
                <div class="field-group"><label>WhatsApp</label><input type="text" id="cli-tel"></div>
                <div class="field-group"><label>E-mail</label><input type="email" id="cli-email"></div>
                <div class="field-group"><label>Endereço</label><input type="text" id="cli-end"></div>
                <div class="field-group"><label>Cidade</label><input type="text" id="cli-cidade"></div>
                <div class="field-group"><label>Estado</label><select id="cli-uf"><option>GO</option><option>DF</option><option>SP</option><option>MG</option><option>RJ</option><option>PR</option></select></div>
                <div class="form-actions-column">
                    <button class="btn btn-add" id="btn-save-cli" onclick="addCliente()">SALVAR</button>
                    <button class="btn btn-new" onclick="cancelarEdicaoCliente()">CANCELAR</button>
                </div>
            </div>
            <table>
                <thead><tr><th>Cliente / Doc</th><th>Contato</th><th>Cidade/UF</th><th>Última Compra</th><th class="no-print">Ações</th></tr></thead>
                <tbody id="tbl-clientes-corpo"></tbody>
            </table>
        </section>

        <section id="tab-orcamento" class="section-panel">
            <div class="form-grid no-print" style="display:grid; grid-template-columns: 2fr 2fr 1fr 1fr auto auto;">
                <select id="orc-cli-sel" onchange="calculateTotal()"><option value="">Selecionar Cliente</option></select>
                <input type="text" id="orc-search" placeholder="🔍 Buscar Produto..." onkeyup="filterItems()">
                <select id="orc-item-sel"><option value="">Pesquise...</option></select>
                <input type="number" id="orc-qtd-sel" value="1">
                <button class="btn btn-add" onclick="addItemOrc()">INSERIR</button>
                <button class="btn btn-new" onclick="novoOrcamento()">NOVO ORÇAMENTO</button>
            </div>
            <div class="form-grid no-print" style="display:grid; background:#f0f7ff; border-color: #bcd9f5;">
                <div class="field-group"><label>Acessórios (%)</label><input type="number" id="orc-perc-acess" oninput="calculateTotal()"></div>
                <div class="field-group"><label>Mão de Obra (R$)</label><input type="number" id="orc-mo-fixo" oninput="calculateTotal()"></div>
                <div class="field-group"><label>Mão de Obra (%)</label><input type="number" id="orc-perc-mo" oninput="calculateTotal()"></div>
            </div>
            <div id="orc-area">
                <h2 style="text-align:center; border-bottom: 2px solid #000; padding-bottom:10px;">ORÇAMENTO - ASB AUTOMAÇÃO</h2>
                <p id="orc-info-cliente" style="font-weight:bold;"></p>
                <table>
                    <thead><tr><th>QTD</th><th>DESCRIÇÃO TÉCNICA</th><th>UNITÁRIO</th><th>SUBTOTAL</th><th class="no-print">AÇÃO</th></tr></thead>
                    <tbody id="orc-lista-corpo"></tbody>
                </table>
                <div class="summary-box">
                    <div class="summary-row"><span>Materiais:</span><strong id="res-materiais">R$ 0.00</strong></div>
                    <div class="summary-row"><span>Acessórios:</span><strong id="res-acessorios">R$ 0.00</strong></div>
                    <div class="summary-row"><span>Mão de Obra:</span><strong id="res-mo">R$ 0.00</strong></div>
                    <div class="summary-row" style="font-size:20px; border-top:2px solid #333; padding-top:15px; color:var(--asb-blue);"><span>TOTAL FINAL:</span><span id="res-total-final" style="font-weight:900;">R$ 0.00</span></div>
                </div>
                <div class="no-print">
                    <button class="btn btn-print" onclick="window.print()">🖨️ GERAR PDF / IMPRIMIR</button>
                    <button class="btn btn-add" style="width:100%; margin-top:10px; height:50px;" id="btn-finalizar-orc" onclick="finalizarERegistrar()">✅ FINALIZAR E SALVAR</button>
                </div>
            </div>
        </section>

        <section id="tab-relatorios" class="section-panel">
            <div class="search-hero"><input type="text" id="search-hist" placeholder="🔍 Filtrar histórico por cliente..." onkeyup="render()"></div>
            <table>
                <thead><tr><th>Data</th><th>Cliente</th><th>Valor</th><th>Status</th><th class="no-print">Ações</th></tr></thead>
                <tbody id="tbl-historico-corpo"></tbody>
            </table>
        </section>

        <section id="tab-movimentacao" class="section-panel">
            <h3>Log de Saídas de Estoque</h3>
            <table>
                <thead><tr><th>Data</th><th>Produto</th><th>Qtd</th><th>Destino (Cliente)</th></tr></thead>
                <tbody id="tbl-log-corpo"></tbody>
            </table>
        </section>

        <section id="tab-master" class="section-panel">
            <div class="form-grid" style="display:grid; grid-template-columns: 1fr 1fr 1fr;">
                <input type="text" id="new-user" placeholder="Usuário">
                <input type="password" id="new-pass" placeholder="Senha">
                <button class="btn btn-add" onclick="addNewUser()">CADASTRAR</button>
            </div>
            <table>
                <thead><tr><th>Usuário</th><th>Nível</th><th>Ações</th></tr></thead>
                <tbody id="tbl-users-corpo"></tbody>
            </table>
        </section>

        <section id="tab-config" class="section-panel">
            <h3 style="margin-top:0;">Gestão de Backup e Nuvem</h3>
            <div style="display:flex; flex-direction:column; gap:15px; max-width:400px;">
                <button class="btn" style="background:var(--asb-success); height:55px;" onclick="forceRealtimeSync()">🔄 RECONECTAR CANAL DE DADOS</button>
                <button class="btn" style="background:var(--asb-dark); height:45px;" onclick="exportDB()">💾 BAIXAR BACKUP LOCAL (.JSON)</button>
                <div style="border: 2px dashed #ccc; padding: 20px; border-radius: 8px; text-align: center;">
                    <label>IMPORTAR PARA NUVEM:</label>
                    <input type="file" id="import-file" accept=".json" onchange="importDB(this)">
                </div>
                <button class="btn btn-del" style="height:45px;" onclick="limparHistoricoGeral()">⚠️ LIMPAR HISTÓRICO E LOGS</button>
            </div>
        </section>
    </div>
</div>

<script>
    // CONFIGURAÇÃO FIREBASE
    const firebaseConfig = {
        apiKey: "AIzaSyA8rHSh4HW_bSVzccYPb49aQJ5QlvakAKo",
        authDomain: "asb-sistema.firebaseapp.com",
        databaseURL: "https://asb-sistema-default-rtdb.firebaseio.com",
        projectId: "asb-sistema",
        storageBucket: "asb-sistema.firebasestorage.app",
        messagingSenderId: "330568541447",
        appId: "1:330568541447:web:93017cef6d072bbc2951a2"
    };

    firebase.initializeApp(firebaseConfig);
    const database = firebase.database();
    const DB_PATH = 'asb_cloud_master_v81';
    let db = { users:[{user:'admin', pass:'asb123', level:'master'}], estoque:[], clientes:[], orc_temp:[], historico:[], movimentacoes:[] };
    
    let editingEstoqueIndex = null;
    let editingClienteIndex = null;
    let currentEditingOrcId = null;

    function initCloud() {
        database.ref(DB_PATH).on('value', (snapshot) => {
            const val = snapshot.val();
            if(val) {
                db = val;
                if(!db.users) db.users = [{user:'admin', pass:'asb123', level:'master'}];
                document.getElementById('sync-indicator').innerText = "Nuvem Ativa";
                document.getElementById('sync-indicator').className = 'sync-badge sync-online';
                render();
            } else { save(); }
        });
    }
    initCloud();

    function save() { database.ref(DB_PATH).set(db).catch(e => console.error("Erro ao salvar", e)); }

    function handleLogin() {
        const u = document.getElementById('user-input').value.toLowerCase().trim();
        const p = document.getElementById('pass-input').value.trim();
        const found = db.users.find(x => x.user.toLowerCase() === u && x.pass === p);
        if((u === 'admin' && p === 'asb123') || found) {
            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('main-app').style.display = 'block';
            document.getElementById('welcome-msg').innerText = `Logado: ${u.toUpperCase()}`;
            if((found && found.level === 'master') || u === 'admin') document.getElementById('nav-master').style.display = 'block';
            refreshTemperature(); render();
        } else { alert("Erro de Login"); }
    }

    function openTab(id, btn) {
        document.querySelectorAll('.section-panel').forEach(s => s.classList.remove('active'));
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        btn.classList.add('active'); render();
    }

    function mostrarFormEstoque() { document.getElementById('form-estoque').style.display = 'grid'; }
    function addEstoque() {
        const item = {
            desc: document.getElementById('est-desc').value.trim(),
            unid: document.getElementById('est-unid').value.trim(),
            ncm: document.getElementById('est-ncm').value.trim(),
            cfop: document.getElementById('est-cfop').value.trim(),
            cst: document.getElementById('est-cst').value.trim(),
            ipi: parseFloat(document.getElementById('est-ipi').value) || 0,
            icms: parseFloat(document.getElementById('est-icms').value) || 0,
            compra: parseFloat(document.getElementById('est-compra').value) || 0,
            markup: parseFloat(document.getElementById('est-markup').value) || 0,
            val: parseFloat(document.getElementById('est-val').value) || 0,
            qtd: parseInt(document.getElementById('est-qtd').value) || 0
        };
        if(!item.desc) return;
        if(editingEstoqueIndex !== null) db.estoque[editingEstoqueIndex] = item;
        else db.estoque.push(item);
        cancelarEdicao(); save();
    }

    function editEstItem(idx) {
        const i = db.estoque[idx];
        document.getElementById('est-desc').value = i.desc;
        document.getElementById('est-unid').value = i.unid;
        document.getElementById('est-ncm').value = i.ncm;
        document.getElementById('est-cfop').value = i.cfop;
        document.getElementById('est-cst').value = i.cst;
        document.getElementById('est-ipi').value = i.ipi;
        document.getElementById('est-icms').value = i.icms;
        document.getElementById('est-compra').value = i.compra;
        document.getElementById('est-markup').value = i.markup;
        document.getElementById('est-val').value = i.val;
        document.getElementById('est-qtd').value = i.qtd;
        editingEstoqueIndex = idx;
        mostrarFormEstoque();
        document.getElementById('btn-save-est').innerText = "ATUALIZAR";
    }

    function cancelarEdicao() {
        editingEstoqueIndex = null;
        document.getElementById('form-estoque').style.display = 'none';
        ['est-desc','est-unid','est-ncm','est-cfop','est-cst','est-ipi','est-icms','est-compra','est-markup','est-val','est-qtd'].forEach(id => document.getElementById(id).value = '');
    }

    function calcularPrecoVenda() {
        const comp = parseFloat(document.getElementById('est-compra').value) || 0;
        const mark = parseFloat(document.getElementById('est-markup').value) || 0;
        document.getElementById('est-val').value = (comp + (comp * (mark/100))).toFixed(2);
    }

    function mostrarFormCliente() { document.getElementById('form-cliente').style.display = 'grid'; }
    function addCliente() {
        const c = {
            nome: document.getElementById('cli-nome').value.trim(),
            razao: document.getElementById('cli-razao').value.trim(),
            tipo: document.getElementById('cli-tipo').value,
            doc: document.getElementById('cli-doc').value.trim(),
            tel: document.getElementById('cli-tel').value.trim(),
            email: document.getElementById('cli-email').value.trim(),
            end: document.getElementById('cli-end').value.trim(),
            cidade: document.getElementById('cli-cidade').value.trim(),
            uf: document.getElementById('cli-uf').value,
            ultima_compra: editingClienteIndex !== null ? db.clientes[editingClienteIndex].ultima_compra : '---'
        };
        if(!c.nome) return;
        if(editingClienteIndex !== null) db.clientes[editingClienteIndex] = c;
        else db.clientes.push(c);
        cancelarEdicaoCliente(); save();
    }

    function editCliente(idx) {
        const c = db.clientes[idx];
        document.getElementById('cli-nome').value = c.nome;
        document.getElementById('cli-razao').value = c.razao;
        document.getElementById('cli-tipo').value = c.tipo;
        document.getElementById('cli-doc').value = c.doc;
        document.getElementById('cli-tel').value = c.tel;
        document.getElementById('cli-email').value = c.email;
        document.getElementById('cli-end').value = c.end;
        document.getElementById('cli-cidade').value = c.cidade;
        document.getElementById('cli-uf').value = c.uf;
        editingClienteIndex = idx;
        mostrarFormCliente();
        document.getElementById('btn-save-cli').innerText = "ATUALIZAR";
    }

    function cancelarEdicaoCliente() {
        editingClienteIndex = null;
        document.getElementById('form-cliente').style.display = 'none';
        ['cli-nome','cli-razao','cli-doc','cli-tel','cli-email','cli-end','cli-cidade'].forEach(id => document.getElementById(id).value = '');
    }

    function filterItems() {
        const term = document.getElementById('orc-search').value.toUpperCase();
        const sel = document.getElementById('orc-item-sel');
        sel.innerHTML = '<option value="">Pesquise...</option>';
        if(db.estoque) {
            db.estoque.filter(i => i.desc.toUpperCase().includes(term)).forEach(i => {
                sel.innerHTML += `<option value="${i.desc}">${i.desc} (R$ ${i.val.toFixed(2)})</option>`;
            });
        }
    }

    function addItemOrc() {
        const valSel = document.getElementById('orc-item-sel').value;
        const qtd = parseInt(document.getElementById('orc-qtd-sel').value) || 1;
        const prod = db.estoque.find(p => p.desc === valSel);
        if(prod) { 
            if(!db.orc_temp) db.orc_temp = [];
            db.orc_temp.push({id: Date.now(), desc: prod.desc, val: prod.val, qtd: qtd}); 
            save(); 
        }
    }

    function novoOrcamento() {
        if(confirm("Limpar orçamento atual?")) {
            db.orc_temp = []; currentEditingOrcId = null;
            document.getElementById('orc-cli-sel').value = "";
            ['orc-perc-acess','orc-mo-fixo','orc-perc-mo'].forEach(id => document.getElementById(id).value = '');
            document.getElementById('btn-finalizar-orc').innerText = "✅ FINALIZAR E SALVAR";
            save();
        }
    }

    function calculateTotal() {
        let mat = 0; if(db.orc_temp) db.orc_temp.forEach(o => mat += (o.val * o.qtd));
        const pA = parseFloat(document.getElementById('orc-perc-acess').value) || 0;
        const mF = parseFloat(document.getElementById('orc-mo-fixo').value) || 0;
        const pM = parseFloat(document.getElementById('orc-perc-mo').value) || 0;
        const acess = mat * (pA/100);
        const mo = mF + (mat * (pM/100));
        const total = mat + acess + mo;
        document.getElementById('res-materiais').innerText = `R$ ${mat.toFixed(2)}`;
        document.getElementById('res-acessorios').innerText = `R$ ${acess.toFixed(2)}`;
        document.getElementById('res-mo').innerText = `R$ ${mo.toFixed(2)}`;
        document.getElementById('res-total-final').innerText = `R$ ${total.toFixed(2)}`;
        document.getElementById('orc-info-cliente').innerText = document.getElementById('orc-cli-sel').value ? "CLIENTE: " + document.getElementById('orc-cli-sel').value : "";
        return total;
    }

    // MUDANÇA 2: Registro e Edição do Histórico
    function finalizarERegistrar() {
        const cli = document.getElementById('orc-cli-sel').value;
        if(!cli || !db.orc_temp || db.orc_temp.length === 0) return alert("Selecione cliente e adicione itens!");
        if(!db.historico) db.historico = [];
        
        const orcData = { 
            id: currentEditingOrcId || Date.now(), 
            data: new Date().toLocaleString(), 
            cliente: cli, 
            total: calculateTotal(), 
            status: 'Pendente', 
            itens: [...db.orc_temp],
            perc_acess: document.getElementById('orc-perc-acess').value,
            mo_fixo: document.getElementById('orc-mo-fixo').value,
            perc_mo: document.getElementById('orc-perc-mo').value
        };

        if(currentEditingOrcId) {
            const idx = db.historico.findIndex(x => x.id === currentEditingOrcId);
            db.historico[idx] = orcData;
        } else { db.historico.push(orcData); }

        const cObj = db.clientes.find(c => c.nome === cli);
        if(cObj) cObj.ultima_compra = new Date().toLocaleString();
        
        db.orc_temp = [];
        currentEditingOrcId = null;
        document.getElementById('btn-finalizar-orc').innerText = "✅ FINALIZAR E SALVAR";
        save(); alert("Orçamento registrado!");
    }

    // MUDANÇA 3: Movimentação automática ao Aprovar
    function updateStatus(id, newStatus) {
        const h = db.historico.find(x => x.id === id);
        if(h && h.status !== newStatus) {
            if(newStatus === 'Aprovado') {
                if(confirm("Confirmar aprovação? Isso reduzirá o estoque e registrará a saída.")) {
                    h.itens.forEach(it => {
                        const p = db.estoque.find(e => e.desc === it.desc);
                        if(p) { 
                            p.qtd -= it.qtd; 
                            if(!db.movimentacoes) db.movimentacoes = [];
                            db.movimentacoes.push({data: new Date().toLocaleString(), produto: it.desc, qtd: it.qtd, destino: h.cliente}); 
                        }
                    });
                    h.status = 'Aprovado';
                } else { render(); return; }
            } else { h.status = newStatus; }
            save();
        }
    }

    function editOrcHist(id) {
        const h = db.historico.find(x => x.id === id);
        if(h) {
            db.orc_temp = [...h.itens];
            currentEditingOrcId = h.id;
            openTab('tab-orcamento', document.querySelectorAll('.nav-btn')[2]);
            document.getElementById('orc-cli-sel').value = h.cliente;
            document.getElementById('orc-perc-acess').value = h.perc_acess || 0;
            document.getElementById('orc-mo-fixo').value = h.mo_fixo || 0;
            document.getElementById('orc-perc-mo').value = h.perc_mo || 0;
            document.getElementById('btn-finalizar-orc').innerText = "💾 ATUALIZAR";
            render();
        }
    }

    function addNewUser() {
        const u = document.getElementById('new-user').value.trim();
        const p = document.getElementById('new-pass').value.trim();
        if(u && p) {
            if(!db.users) db.users = [];
            db.users.push({user: u, pass: p, level: 'comum'});
            save();
        }
    }

    function refreshTemperature() {
        const t = (Math.random() * (26 - 19) + 19).toFixed(1);
        document.getElementById('temp-val').innerText = t + "°C";
    }

    function exportDB() {
        const blob = new Blob([JSON.stringify(db)], {type: 'application/json'});
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a'); a.href = url; a.download = `ASB_BACKUP.json`; a.click();
    }

    function importDB(input) {
        const reader = new FileReader();
        reader.onload = function() {
            db = JSON.parse(reader.result); save(); alert("Sucesso!");
        };
        reader.readAsText(input.files[0]);
    }

    function limparHistoricoGeral() {
        if(confirm("Apagar histórico e logs?")) { db.historico = []; db.movimentacoes = []; save(); }
    }

    function forceRealtimeSync() { initCloud(); alert("Reiniciado!"); }

    function render(filter) {
        if(!db) return;
        const estTbody = document.getElementById('tbl-estoque-corpo');
        if(estTbody) {
            estTbody.innerHTML = '';
            const s = document.getElementById('search-est-input').value.toUpperCase();
            if(db.estoque) db.estoque.forEach((item, idx) => {
                if(filter === 'all' || item.desc.toUpperCase().includes(s)) {
                    estTbody.innerHTML += `<tr><td>${item.desc}</td><td>${item.unid}</td><td>${item.ncm}</td><td>${item.cfop}</td><td>R$ ${parseFloat(item.val).toFixed(2)}</td><td>${item.qtd}</td><td class="no-print"><button class="btn btn-edit" onclick="editEstItem(${idx})">EDIT</button><button class="btn btn-del" onclick="if(confirm('Excluir?')){db.estoque.splice(${idx},1);save();}">DEL</button></td></tr>`;
                }
            });
        }
        const cliTbody = document.getElementById('tbl-clientes-corpo');
        if(cliTbody) {
            cliTbody.innerHTML = '';
            const s = document.getElementById('search-cli-input').value.toUpperCase();
            if(db.clientes) db.clientes.forEach((c, idx) => {
                if(filter === 'all' || c.nome.toUpperCase().includes(s)) {
                    cliTbody.innerHTML += `<tr><td><strong>${c.nome}</strong><br><small>${c.doc}</small></td><td>${c.tel}</td><td>${c.cidade}/${c.uf}</td><td>${c.ultima_compra || '---'}</td><td class="no-print"><button class="btn btn-edit" onclick="editCliente(${idx})">EDIT</button></td></tr>`;
                }
            });
        }
        const orcCliSel = document.getElementById('orc-cli-sel');
        if(orcCliSel && db.clientes) {
            const current = orcCliSel.value;
            orcCliSel.innerHTML = '<option value="">Selecionar Cliente</option>';
            db.clientes.forEach(c => { orcCliSel.innerHTML += `<option value="${c.nome}" ${current === c.nome ? 'selected' : ''}>${c.nome}</option>`; });
        }
        const orcTbody = document.getElementById('orc-lista-corpo');
        if(orcTbody) {
            orcTbody.innerHTML = '';
            if(db.orc_temp) db.orc_temp.forEach((it, idx) => {
                orcTbody.innerHTML += `<tr><td>${it.qtd}</td><td>${it.desc}</td><td>R$ ${parseFloat(it.val).toFixed(2)}</td><td>R$ ${(it.val * it.qtd).toFixed(2)}</td><td class="no-print"><button class="btn btn-del" onclick="db.orc_temp.splice(${idx},1);save();">X</button></td></tr>`;
            });
            calculateTotal();
        }
        const histTbody = document.getElementById('tbl-historico-corpo');
        if(histTbody) {
            histTbody.innerHTML = '';
            const s = document.getElementById('search-hist').value.toUpperCase();
            if(db.historico) db.historico.slice().reverse().forEach((h) => {
                if(h.cliente.toUpperCase().includes(s)) {
                    histTbody.innerHTML += `<tr><td>${h.data}</td><td>${h.cliente}</td><td>R$ ${parseFloat(h.total).toFixed(2)}</td><td><select onchange="updateStatus(${h.id}, this.value)"><option ${h.status==='Pendente'?'selected':''}>Pendente</option><option ${h.status==='Aprovado'?'selected':''}>Aprovado</option><option ${h.status==='Cancelado'?'selected':''}>Cancelado</option></select></td><td class="no-print"><button class="btn btn-edit" onclick="editOrcHist(${h.id})">EDITAR</button><button class="btn btn-del" onclick="if(confirm('Excluir?')){db.historico.splice(db.historico.findIndex(x=>x.id===${h.id}),1);save();}">DEL</button></td></tr>`;
                }
            });
        }
        const logTbody = document.getElementById('tbl-log-corpo');
        if(logTbody) {
            logTbody.innerHTML = '';
            if(db.movimentacoes) db.movimentacoes.slice().reverse().forEach(m => {
                logTbody.innerHTML += `<tr><td>${m.data}</td><td>${m.produto}</td><td>${m.qtd}</td><td>${m.destino}</td></tr>`;
            });
        }
        const userTbody = document.getElementById('tbl-users-corpo');
        if(userTbody) {
            userTbody.innerHTML = '';
            if(db.users) db.users.forEach((u, idx) => {
                userTbody.innerHTML += `<tr><td>${u.user}</td><td>${u.level}</td><td><button class="btn btn-del" onclick="if(confirm('Excluir?')){db.users.splice(${idx},1);save();}">DEL</button></td></tr>`;
            });
        }
    }
</script>
</body>
</html>
