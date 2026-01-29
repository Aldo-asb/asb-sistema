<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="apple-mobile-web-app-title" content="Gestão ASB ENG">
    <meta name="application-name" content="Gestão ASB ENG">
    <meta name="theme-color" content="#1a1a1a">
    <meta name="mobile-web-app-capable" content="yes">
    
    <title>Gestão ASB ENG - v69.0</title>
    
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
        
        .sync-badge { font-size: 10px; padding: 4px 8px; border-radius: 10px; background: #eee; color: #666; margin-left: 10px; vertical-align: middle; }
        .sync-online { background: #d4edda; color: #155724; border: 1px solid #c3e6cb; }

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
        
        table { width: 100%; border-collapse: separate; border-spacing: 0; margin-top: 15px; border: 1px solid #eee; border-radius: 8px; overflow: hidden; }
        th { background: #fdfdfd; padding: 15px; text-align: left; border-bottom: 2px solid #eee; font-size: 11px; color: #888; text-transform: uppercase; }
        td { padding: 14px 15px; border-bottom: 1px solid #f6f6f6; font-size: 14px; }
        
        .btn { padding: 10px 20px; border: none; border-radius: 6px; color: white; cursor: pointer; font-weight: bold; text-transform: uppercase; transition: 0.3s; height: 42px; display: inline-flex; align-items: center; justify-content: center; gap: 8px; font-size: 11px; }
        .btn-add { background: var(--asb-success-grad); }
        .btn-new { background: #6c757d; }
        .btn-edit { background: #ffc107; color: #000; padding: 5px 12px; height: 30px; }
        .btn-del { background: #e74c3c; padding: 5px 12px; height: 30px; }
        .btn-print { background: var(--asb-blue-grad); width: 100%; margin-top: 15px; height: 50px; font-size: 14px; }

        .edit-only { display: none; }
        .edit-highlight { background: #fff9c4 !important; border: 2px solid #fbc02d !important; }

        .summary-box { background: #f8f9fa; padding: 25px; margin-top: 20px; border-radius: 10px; border: 1px solid #eee; }
        .summary-row { display: flex; justify-content: space-between; padding: 8px 0; border-bottom: 1px solid #eee; font-size: 14px; }

        .temp-refresh-btn { background: var(--asb-success); color: white; border: none; border-radius: 4px; padding: 5px 10px; cursor: pointer; font-size: 10px; margin-left: 10px; }

        @media print {
            body { background: white; }
            nav, .no-print, .search-hero, header, .form-grid, .btn { display: none !important; }
            .container { box-shadow: none; border: none; width: 100%; margin: 0; padding: 0; }
            .section-panel { display: none !important; }
            #tab-estoque.active, #tab-clientes.active, #tab-orcamento.active { display: block !important; }
            table th:last-child, table td:last-child { display: none !important; }
            #tab-orcamento.active #orc-area { display: block !important; }
            .summary-box { border: 1px solid #333; }
        }
    </style>
</head>
<body>

<div id="login-screen">
    <div class="login-box">
        <h2 style="margin-bottom:30px;">ASB <span style="color:var(--asb-blue); font-weight:300;">SISTEMA</span></h2>
        <input type="text" id="user-input" placeholder="Usuário" style="margin-bottom:15px;">
        <input type="password" id="pass-input" placeholder="Senha" style="margin-bottom:25px;">
        <button class="btn btn-add" style="width: 100%; height:50px;" onclick="handleLogin()">ACESSAR GESTÃO</button>
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
                    <tr>
                        <th>Descrição</th>
                        <th>Unid.</th>
                        <th>NCM</th>
                        <th>CFOP</th>
                        <th>Vlr. Venda</th>
                        <th>Estoque</th>
                        <th class="no-print">Ações</th>
                    </tr>
                </thead>
                <tbody id="tbl-estoque-corpo"></tbody>
            </table>
        </section>

        <section id="tab-clientes" class="section-panel">
            <div class="search-hero">
                <button class="btn btn-new" onclick="mostrarFormCliente()">NOVO CLIENTE</button>
                <input type="text" id="search-cli-input" placeholder="🔍 Buscar cliente..." onkeyup="render()">
            </div>
            <table>
                <thead><tr><th>Cliente / Doc</th><th>Contato</th><th>Cidade/UF</th><th>Última Compra</th><th class="no-print">Ações</th></tr></thead>
                <tbody id="tbl-clientes-corpo"></tbody>
            </table>
        </section>

        <section id="tab-orcamento" class="section-panel">
            <div class="form-grid no-print" style="display:grid; grid-template-columns: 2fr 2fr 1fr 1fr auto auto;">
                <select id="orc-cli-sel"><option value="">Selecionar Cliente</option></select>
                <input type="text" id="orc-search" placeholder="🔍 Buscar Produto..." onkeyup="filterItems()">
                <select id="orc-item-sel"><option value="">Pesquise...</option></select>
                <input type="number" id="orc-qtd-sel" value="1">
                <button class="btn btn-add" onclick="addItemOrc()">INSERIR</button>
                <button class="btn btn-new" onclick="novoOrcamento()">NOVO ORÇAMENTO</button>
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
                    <div class="summary-row" style="font-size:20px; border-top:2px solid #333; padding-top:15px; color:var(--asb-blue);"><span>TOTAL FINAL:</span><span id="res-total-final" style="font-weight:900;">R$ 0.00</span></div>
                </div>
                <button class="btn btn-print no-print" onclick="window.print()">🖨️ GERAR PDF / IMPRIMIR</button>
            </div>
        </section>

        <section id="tab-relatorios" class="section-panel">
            <table>
                <thead><tr><th>Data</th><th>Cliente</th><th>Valor</th><th>Status</th><th>Ações</th></tr></thead>
                <tbody id="tbl-historico-corpo"></tbody>
            </table>
        </section>

        <section id="tab-movimentacao" class="section-panel">
            <h3>Log de Saídas de Estoque</h3>
            <tbody id="tbl-log-corpo"></tbody>
        </section>

        <section id="tab-master" class="section-panel">
            <div class="form-grid" style="display:grid;">
                <input type="text" id="new-user" placeholder="Usuário">
                <input type="password" id="new-pass" placeholder="Senha">
                <button class="btn btn-add" onclick="addNewUser()">CADASTRAR</button>
            </div>
            <tbody id="tbl-users-corpo"></tbody>
        </section>

        <section id="tab-config" class="section-panel">
            <h3 style="margin-top:0;">Gestão de Backup e Nuvem</h3>
            <div style="display:flex; flex-direction:column; gap:15px; max-width:400px;">
                <button class="btn" style="background:var(--asb-success); height:55px;" onclick="forçarSincronia()">🔄 FORÇAR SINCRONIA (CELULAR)</button>
                <button class="btn" style="background:var(--asb-dark); height:45px;" onclick="exportDB()">💾 BAIXAR BACKUP LOCAL (.JSON)</button>
                <div style="border: 2px dashed #ccc; padding: 20px; border-radius: 8px; text-align: center;">
                    <label>IMPORTAR PARA NUVEM:</label>
                    <input type="file" accept=".json" onchange="importDB(this)">
                </div>
            </div>
        </section>
    </div>
</div>

<script>
    // CONFIGURAÇÃO FIREBASE - v69.0
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
    const DB_KEY = 'asb_erp_db_v37';
    let db = JSON.parse(localStorage.getItem(DB_KEY)) || {};

    function initDB() {
        if (!db.users) db.users = [{user: 'admin', pass: 'asb123', level: 'master'}];
        if (!db.estoque) db.estoque = [];
        if (!db.clientes) db.clientes = [];
        if (!db.historico) db.historico = [];
        if (!db.movimentacoes) db.movimentacoes = [];
        if (!db.orc_temp) db.orc_temp = [];
        localStorage.setItem(DB_KEY, JSON.stringify(db));
    }
    initDB();

    // ESCUTA EM TEMPO REAL - MELHORADA PARA CELULAR
    database.ref('sistema').on('value', (snapshot) => {
        const cloudData = snapshot.val();
        if (cloudData) {
            db = cloudData;
            localStorage.setItem(DB_KEY, JSON.stringify(db));
            const badge = document.getElementById('sync-indicator');
            badge.innerText = "Online (Nuvem)";
            badge.classList.add('sync-online');
            render();
        }
    }, (error) => {
        console.error("Erro na nuvem:", error);
        document.getElementById('sync-indicator').innerText = "Erro de Conexão";
    });

    function forçarSincronia() {
        database.ref('sistema').once('value').then((snapshot) => {
            const data = snapshot.val();
            if(data) {
                db = data;
                localStorage.setItem(DB_KEY, JSON.stringify(db));
                render();
                alert("Dados sincronizados com sucesso!");
            }
        });
    }

    function save() { 
        localStorage.setItem(DB_KEY, JSON.stringify(db)); 
        database.ref('sistema').set(db);
        render(); 
    }

    function handleLogin() {
        const u = document.getElementById('user-input').value;
        const p = document.getElementById('pass-input').value;
        const found = db.users.find(x => x.user === u && x.pass === p);
        if(found) {
            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('main-app').style.display = 'block';
            document.getElementById('welcome-msg').innerText = `Logado: ${u.toUpperCase()}`;
            if(found.level === 'master') document.getElementById('nav-master').style.display = 'block';
            refreshTemperature();
            render();
        } else { alert("Acesso Negado!"); }
    }

    function openTab(id, btn) {
        document.querySelectorAll('.section-panel').forEach(s => s.classList.remove('active'));
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        btn.classList.add('active');
        render();
    }

    // FUNÇÕES DE RENDERIZAÇÃO
    function render(filter = '') {
        renderEstoque(filter);
        renderClientes(filter);
        renderOrcamento();
        renderHistorico();
        renderUsers();
    }

    function renderEstoque(f) {
        const corpo = document.getElementById('tbl-estoque-corpo');
        if(!corpo) return;
        corpo.innerHTML = '';
        const search = document.getElementById('search-est-input').value.toUpperCase();
        db.estoque.forEach((item, idx) => {
            if(f !== 'all' && search && !item.desc.toUpperCase().includes(search)) return;
            corpo.innerHTML += `<tr>
                <td>${item.desc}</td>
                <td>${item.unid}</td>
                <td>${item.ncm}</td>
                <td>${item.cfop}</td>
                <td>R$ ${item.val.toFixed(2)}</td>
                <td style="font-weight:bold; color:${item.qtd < 5 ? 'red' : 'green'}">${item.qtd}</td>
                <td class="no-print">
                    <button class="btn btn-edit" onclick="editEstItem(${idx})">✏️</button>
                    <button class="btn btn-del" onclick="if(confirm('Excluir?')){db.estoque.splice(${idx},1);save();}">🗑️</button>
                </td>
            </tr>`;
        });
    }

    function renderClientes(f) {
        const corpo = document.getElementById('tbl-clientes-corpo');
        if(!corpo) return;
        corpo.innerHTML = '';
        const search = document.getElementById('search-cli-input').value.toUpperCase();
        db.clientes.forEach((c, idx) => {
            if(search && !c.nome.toUpperCase().includes(search)) return;
            corpo.innerHTML += `<tr>
                <td><strong>${c.nome}</strong><br><small>${c.doc}</small></td>
                <td>${c.tel}</td>
                <td>${c.cidade}/${c.uf}</td>
                <td>${c.ultima_compra || '---'}</td>
                <td class="no-print">
                    <button class="btn btn-edit" onclick="editCliente(${idx})">✏️</button>
                </td>
            </tr>`;
        });
        const sel = document.getElementById('orc-cli-sel');
        if(sel) {
            const valAnterior = sel.value;
            sel.innerHTML = '<option value="">Selecionar Cliente</option>';
            db.clientes.forEach(c => sel.innerHTML += `<option value="${c.nome}">${c.nome}</option>`);
            sel.value = valAnterior;
        }
    }

    function renderOrcamento() {
        const corpo = document.getElementById('orc-lista-corpo');
        if(!corpo) return;
        corpo.innerHTML = '';
        let total = 0;
        db.orc_temp.forEach((it, idx) => {
            const sub = it.val * it.qtd;
            total += sub;
            corpo.innerHTML += `<tr>
                <td>${it.qtd}</td>
                <td>${it.desc}</td>
                <td>R$ ${it.val.toFixed(2)}</td>
                <td>R$ ${sub.toFixed(2)}</td>
                <td class="no-print"><button class="btn btn-del" onclick="db.orc_temp.splice(${idx},1);save();">X</button></td>
            </tr>`;
        });
        document.getElementById('res-materiais').innerText = `R$ ${total.toFixed(2)}`;
        document.getElementById('res-total-final').innerText = `R$ ${total.toFixed(2)}`;
    }

    function renderHistorico() {
        const corpo = document.getElementById('tbl-historico-corpo');
        if(!corpo) return;
        corpo.innerHTML = '';
        db.historico.slice().reverse().forEach(h => {
            corpo.innerHTML += `<tr>
                <td>${h.data}</td>
                <td>${h.cliente}</td>
                <td>R$ ${h.total.toFixed(2)}</td>
                <td>${h.status}</td>
                <td><button class="btn btn-edit" onclick="alert('Funcionalidade em desenvolvimento')">VER</button></td>
            </tr>`;
        });
    }

    function renderUsers() {
        const corpo = document.getElementById('tbl-users-corpo');
        if(!corpo) return;
        corpo.innerHTML = '';
        db.users.forEach(u => {
            corpo.innerHTML += `<tr><td>${u.user}</td><td>${u.level}</td><td>-</td></tr>`;
        });
    }

    // CRUD ESTOQUE E CLIENTES (BASICO)
    function mostrarFormEstoque() { document.getElementById('form-estoque').style.display = 'grid'; }
    function mostrarFormCliente() { document.getElementById('form-cliente').style.display = 'grid'; }
    function cancelarEdicao() { document.getElementById('form-estoque').style.display = 'none'; }
    function cancelarEdicaoCliente() { document.getElementById('form-cliente').style.display = 'none'; }

    function addEstoque() {
        const item = {
            desc: document.getElementById('est-desc').value,
            unid: document.getElementById('est-unid').value,
            val: parseFloat(document.getElementById('est-val').value) || 0,
            qtd: parseInt(document.getElementById('est-qtd').value) || 0,
            ncm: document.getElementById('est-ncm').value,
            cfop: document.getElementById('est-cfop').value
        };
        db.estoque.push(item);
        save();
        cancelarEdicao();
    }

    function refreshTemperature() {
        const t = (Math.random() * (26 - 19) + 19).toFixed(1);
        document.getElementById('temp-val').innerText = t + "°C";
    }

    function exportDB() {
        const blob = new Blob([JSON.stringify(db)], {type: 'application/json'});
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `ASB_BACKUP.json`;
        a.click();
    }

    function importDB(input) {
        const reader = new FileReader();
        reader.onload = function() {
            db = JSON.parse(reader.result);
            save();
            alert("Sincronizado!");
        };
        reader.readAsText(input.files[0]);
    }

    // v69.0
</script>
</body>
</html>
