<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="apple-mobile-web-app-title" content="Gestão ASB ENG">
    <meta name="application-name" content="Gestão ASB ENG">
    <meta name="theme-color" content="#1a1a1a">
    <meta name="mobile-web-app-capable" content="yes">
    
    <title>Gestão ASB ENG - v117.0</title>
    
    <script src="https://www.gstatic.com/firebasejs/9.6.10/firebase-app-compat.js">
    // ── Modais reutilizáveis ──────────────────────────────────────────
    function mostrarModalInfo(msg) {
        document.getElementById('modal-msg').innerHTML = msg;
        document.getElementById('modal-btn-sim').style.display = 'none';
        document.getElementById('modal-btn-nao').innerText = 'OK';
        document.getElementById('modal-btn-nao').style.background = 'var(--asb-blue)';
        document.getElementById('modal-confirmacao').style.display = 'flex';
        document.getElementById('modal-btn-nao').onclick = () => {
            document.getElementById('modal-confirmacao').style.display = 'none';
            document.getElementById('modal-btn-sim').style.display = '';
            document.getElementById('modal-btn-nao').innerText = 'CANCELAR';
            document.getElementById('modal-btn-nao').style.background = '';
        };
    }

    function mostrarModalConfirm(msg, onSim, onNao) {
        document.getElementById('modal-msg').innerHTML = msg;
        document.getElementById('modal-btn-sim').style.display = '';
        document.getElementById('modal-btn-sim').innerText = 'SIM, CONFIRMAR';
        document.getElementById('modal-btn-nao').innerText = 'CANCELAR';
        document.getElementById('modal-btn-nao').style.background = '';
        document.getElementById('modal-confirmacao').style.display = 'flex';
        document.getElementById('modal-btn-sim').onclick = () => {
            document.getElementById('modal-confirmacao').style.display = 'none';
            if(onSim) onSim();
        };
        document.getElementById('modal-btn-nao').onclick = () => {
            document.getElementById('modal-confirmacao').style.display = 'none';
            if(onNao) onNao();
        };
    }

    function excluirEstoque(idx) {
        const item = db.estoque[idx];
        mostrarModalConfirm(`Excluir produto <strong>${item ? item.desc : ''}</strong> do estoque?`, () => {
            db.estoque.splice(idx, 1); save();
        });
    }

    function excluirUsuario(idx) {
        const u = db.users[idx];
        if(u && u.level === 'master') return mostrarModalInfo("⚠️ Não é possível excluir um usuário master!");
        mostrarModalConfirm(`Excluir usuário <strong>${u ? u.user : ''}</strong>?`, () => {
            db.users.splice(idx, 1); save();
        });
    }

    function excluirOrcamento(id) {
        const h = db.historico.find(x => x.id === id);
        const num = h ? (h.numero || h.id) : id;
        mostrarModalConfirm(`Excluir orçamento <strong>Nº ${num}</strong>?<br>Esta ação não pode ser desfeita.`, () => {
            db.historico.splice(db.historico.findIndex(x => x.id === id), 1);
            save();
        });
    }

    // ── Gerar número ao entrar na aba orçamento sem orçamento ativo ──
    // (garante que sempre haja um número disponível)
    document.querySelector('[onclick*="tab-orcamento"]').addEventListener('click', function() {
        setTimeout(() => {
            if(!currentOrcNumero && (!db.orc_temp || db.orc_temp.length === 0)) {
                currentOrcNumero = gerarNumeroOrc();
                const badge = document.getElementById('orc-num-badge');
                if(badge) badge.innerText = `Nº ${currentOrcNumero}`;
                const imp = document.getElementById('orc-num-impresso');
                if(imp) imp.innerText = `Nº ${currentOrcNumero}  |  Data: ${new Date().toLocaleDateString('pt-BR')}`;
            }
        }, 100);
    });

</script>
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
        th { background: #fdfdfd; padding: 10px 15px; text-align: left; border-bottom: 2px solid #eee; font-size: 11px; color: #888; text-transform: uppercase; }
        td { padding: 1px 15px !important; border-bottom: 1px solid #f6f6f6; font-size: 13px; line-height: 1.0 !important; height: 22px; }
        
        .btn { padding: 10px 20px; border: none; border-radius: 6px; color: white; cursor: pointer; font-weight: bold; text-transform: uppercase; transition: 0.3s; height: 38px; display: inline-flex; align-items: center; justify-content: center; gap: 8px; font-size: 10px; }
        .btn-add { background: var(--asb-success-grad); }
        .btn-new { background: #6c757d; }
        .btn-edit { background: #e8f4fd; color: #0056b3; border: 1px solid #b8d4ee; padding: 4px 10px; height: 26px; font-size: 10px; }
        .btn-edit:hover { background: #cce3f5; }
        .btn-del  { background: #fdecea; color: #c0392b; border: 1px solid #f5c6c2; padding: 4px 10px; height: 26px; font-size: 10px; }
        .btn-del:hover  { background: #fbd5d1; }
        /* Ações na mesma linha */
        td .acoes-inline { display: flex; gap: 5px; align-items: center; flex-wrap: nowrap; }
        .btn-print { background: var(--asb-blue-grad); width: 100%; margin-top: 15px; height: 50px; font-size: 14px; }

        .hide-table { display: none; }

        @media print {
            body { background: white !important; }
            nav, .no-print, .search-hero, header, .form-grid, .btn { display: none !important; }
            .container { box-shadow: none; border: none; width: 100% !important; margin: 0; padding: 0; }
            .section-panel { display: none !important; }
            #tab-estoque.active, #tab-clientes.active, #tab-orcamento.active { display: block !important; }
            #table-estoque, #table-clientes, #table-historico, #orc-area { display: table !important; }
            #orc-area { display: block !important; }
            table th:last-child, table td:last-child { display: none !important; }
            .summary-box { border: 1px solid #333; }
            td { padding: 0px 10px !important; font-size: 11px !important; height: 16px !important; }
        }

        /* Summary box - totais do orçamento */
        .summary-box { background: #f8f9fa; border: 1px solid #dee2e6; border-radius: 8px; padding: 20px; margin-top: 20px; }
        .summary-row { display: flex; justify-content: space-between; align-items: center; padding: 8px 0; font-size: 15px; border-bottom: 1px solid #eee; }
        .summary-row:last-child { border-bottom: none; }

        /* Coluna de ações nos formulários */
        .form-actions-column { display: flex; flex-direction: column; gap: 8px; justify-content: flex-end; }

        /* Botão de temperatura */
        .temp-refresh-btn { background: none; border: none; cursor: pointer; font-size: 11px; color: #888; padding: 2px 6px; }

        /* Busca de produto no orçamento - estilo melhorado */
        .orc-search-wrapper { position: relative; flex: 1; }
        .orc-search-wrapper input { width: 100%; box-sizing: border-box; }
        #orc-resultados { position: absolute; top: 100%; left: 0; right: 0; background: white; border: 2px solid var(--asb-blue); border-top: none; border-radius: 0 0 8px 8px; max-height: 220px; overflow-y: auto; z-index: 1000; display: none; box-shadow: 0 8px 20px rgba(0,0,0,0.15); }
        #orc-resultados .orc-res-item { padding: 10px 15px; cursor: pointer; font-size: 13px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #f0f0f0; transition: background 0.15s; }
        #orc-resultados .orc-res-item:hover, #orc-resultados .orc-res-item.selected { background: #e8f0fe; }
        #orc-resultados .orc-res-item .orc-res-preco { font-weight: bold; color: var(--asb-blue); white-space: nowrap; margin-left: 10px; }
        #orc-resultados .orc-sem-resultado { padding: 12px 15px; color: #999; font-size: 13px; text-align: center; }

        /* Feedback de item adicionado */
        .flash-success { animation: flashGreen 0.6s ease; }
        @keyframes flashGreen { 0%,100%{ background: transparent; } 50%{ background: #d4edda; } }

        /* Modal de confirmação customizado */
        #modal-confirmacao { display:none; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(0,0,0,0.55); z-index:9999; justify-content:center; align-items:center; }
        .modal-box { background:white; border-radius:12px; padding:30px 35px; max-width:350px; width:90%; text-align:center; box-shadow: 0 20px 50px rgba(0,0,0,0.3); }
        .modal-box p { font-size:16px; margin-bottom:25px; color:#333; }
        .modal-btns { display:flex; gap:12px; justify-content:center; }
        .modal-btns button { flex:1; padding:12px; border-radius:8px; border:none; font-weight:bold; font-size:14px; cursor:pointer; }
        #modal-btn-sim { background:#dc3545; color:white; }
        #modal-btn-nao { background:#6c757d; color:white; }


        /* Cabeçalho do orçamento impresso */
        .orc-cabecalho { text-align: center; border-bottom: 3px solid #000; padding-bottom: 12px; margin-bottom: 14px; }
        .orc-cabecalho .empresa-nome { font-size: 22px; font-weight: 900; letter-spacing: 1px; margin: 0; }
        .orc-cabecalho .empresa-sub  { font-size: 13px; color: #555; margin: 2px 0 8px; }
        .orc-cabecalho .orc-titulo   { font-size: 16px; font-weight: 700; border: 2px solid #000; display: inline-block; padding: 3px 30px; margin-top: 6px; letter-spacing: 2px; }
        .orc-cabecalho .orc-num-line { font-size: 12px; color: #333; margin-top: 6px; }

        /* Bloco de dados do cliente na impressão */
        .orc-dados-cliente { background: #f8f9fa; border: 1px solid #ccc; border-radius: 6px; padding: 12px 16px; margin-bottom: 14px; display: grid; grid-template-columns: 1fr 1fr; gap: 2px 20px; font-size: 12px; }
        .orc-dc-header { grid-column: 1/-1; display: flex; justify-content: space-between; align-items: baseline; border-bottom: 1px solid #ccc; padding-bottom: 6px; margin-bottom: 4px; }
        .orc-dc-header strong { font-size: 14px; }
        .orc-dc-numdata { font-size: 12px; font-weight: 700; color: #0056b3; white-space: nowrap; }
        .orc-dados-cliente span { color: #444; }
        .orc-dados-cliente span b { color: #111; }

        /* Campo número orçamento na barra de controle */
        .orc-num-badge { background: #fff3cd; border: 2px solid #ffc107; border-radius: 8px; padding: 6px 14px; font-size: 13px; font-weight: 700; color: #856404; display: flex; align-items: center; gap: 6px; white-space: nowrap; }

        @media print {
            .orc-dados-cliente { background: white !important; -webkit-print-color-adjust: exact; }
        }

    </style>
</head>
<body>

<div id="login-screen">
    <div class="login-box">
        <h2 style="margin-bottom:30px;">ASB <span style="color:var(--asb-blue); font-weight:300;">SISTEMA</span></h2>
        <input type="text" id="user-input" placeholder="Usuário" style="margin-bottom:15px;" onkeydown="if(event.key==='Enter') document.getElementById('pass-input').focus()">
        <input type="password" id="pass-input" placeholder="Senha" style="margin-bottom:25px;" onkeydown="if(event.key==='Enter') handleLogin()">
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
                <div class="field-group"><label>Unid.</label><input type="text" id="est-unid"></div>
                <div class="field-group"><label>NCM</label><input type="text" id="est-ncm"></div>
                <div class="field-group"><label>CFOP</label><input type="text" id="est-cfop"></div>
                <div class="field-group"><label>CST</label><input type="text" id="est-cst"></div>
                <div class="field-group"><label>Preço Compra (R$)</label><input type="number" id="est-compra" oninput="calcVenda()"></div>
                <div class="field-group"><label>Markup (%)</label><input type="number" id="est-markup" oninput="calcVenda()"></div>
                <div class="field-group"><label>IPI (%)</label><input type="number" id="est-ipi" oninput="calcVenda()"></div>
                <div class="field-group"><label>ICMS (%)</label><input type="number" id="est-icms" oninput="calcVenda()"></div>
                <div class="field-group"><label>Valor Final (Venda)</label><input type="number" id="est-val" step="0.01"></div>
                <div class="field-group"><label>Qtd em Estoque</label><input type="number" id="est-qtd"></div>
                <div class="form-actions-column">
                    <button class="btn btn-add" id="btn-save-est" onclick="addEstoque()">SALVAR</button>
                    <button class="btn btn-new" onclick="cancelarEdicao()">CANCELAR</button>
                </div>
            </div>
            <table id="table-estoque">
                <thead>
                    <tr><th>Descrição</th><th>Unid.</th><th>NCM</th><th>CFOP</th><th>CST</th><th>Vlr. Venda</th><th>Estoque</th><th class="no-print">Ações</th></tr>
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
            <table id="table-clientes">
                <thead><tr><th>Cliente / Doc</th><th>Contato</th><th>Cidade/UF</th><th>Última Compra</th><th class="no-print">Ações</th></tr></thead>
                <tbody id="tbl-clientes-corpo"></tbody>
            </table>
        </section>

        <section id="tab-orcamento" class="section-panel">
            <!-- Linha 1: Cliente | Busca produto | Qtd | Inserir | Novo -->
            <div class="form-grid no-print" style="display:grid; grid-template-columns: 2fr 3fr 70px auto auto;">
                <select id="orc-cli-sel" onchange="calculateTotal()"><option value="">Selecionar Cliente</option></select>
                <div class="orc-search-wrapper">
                    <input type="text" id="orc-search" placeholder="🔍 Buscar produto no estoque..."
                           oninput="filterItemsLive()" onkeyup="filterItemsLive()"
                           onkeydown="orcSearchKeydown(event)" autocomplete="off">
                    <div id="orc-resultados"></div>
                </div>
                <input type="number" id="orc-qtd-sel" value="1" min="1" placeholder="Qtd">
                <button class="btn btn-add" onclick="addItemOrc()">➕ INSERIR</button>
                <button class="btn btn-new" onclick="novoOrcamento()">🗒️ NOVO</button>
            </div>
            <!-- Linha 2: Nº Orçamento | Acessórios | MO fixo | MO % -->
            <div class="form-grid no-print" style="display:grid; grid-template-columns: 1fr 1fr 1fr 1fr; background:#f0f7ff; border-color: #bcd9f5; align-items:end; gap:15px;">
                <div class="field-group">
                    <label>Nº Orçamento</label>
                    <div class="orc-num-badge" id="orc-num-badge" style="height:42px; border-radius:6px; border:1px solid #ffc107; font-size:14px; justify-content:center;">Nº —</div>
                </div>
                <div class="field-group"><label>Acessórios (%)</label><input type="number" id="orc-perc-acess" oninput="calculateTotal()"></div>
                <div class="field-group"><label>Mão de Obra (R$)</label><input type="number" id="orc-mo-fixo" oninput="calculateTotal()"></div>
                <div class="field-group"><label>Mão de Obra (%)</label><input type="number" id="orc-perc-mo" oninput="calculateTotal()"></div>
            </div>
            <div id="orc-area" class="hide-table">
                <!-- Cabeçalho impresso -->
                <div class="orc-cabecalho">
                    <p class="empresa-nome">ASB AUTOMAÇÃO INDUSTRIAL</p>
                    <p class="empresa-sub">Automação, Controle e Engenharia Elétrica</p>
                </div>
                <!-- Dados do cliente -->
                <div class="orc-dados-cliente" id="orc-dados-cliente-bloco">
                    <div class="orc-dc-header">
                        <strong id="orc-dc-nome"></strong>
                        <span id="orc-num-impresso" class="orc-dc-numdata"></span>
                    </div>
                    <span><b>CPF/CNPJ:</b> <span id="orc-dc-doc"></span></span>
                    <span><b>E-mail:</b> <span id="orc-dc-email"></span></span>
                    <span><b>Endereço:</b> <span id="orc-dc-end"></span></span>
                    <span><b>Cidade/UF:</b> <span id="orc-dc-cidade"></span></span>
                    <span><b>WhatsApp:</b> <span id="orc-dc-tel"></span></span>
                </div>
                <!-- Tabela de itens -->
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
            <div class="search-hero">
                <input type="text" id="search-hist" placeholder="🔍 Filtrar por cliente ou Nº orçamento..." onkeyup="render()" style="flex:2">
                <select id="filter-hist-status" onchange="render()" style="width:150px; flex:none;">
                    <option value="">Todos os status</option>
                    <option value="Pendente">Pendente</option>
                    <option value="Aprovado">Aprovado</option>
                    <option value="Cancelado">Cancelado</option>
                </select>
            </div>
            <table id="table-historico">
                <thead><tr><th>Nº</th><th>Data</th><th>Cliente</th><th>Valor</th><th>Status</th><th class="no-print">Ações</th></tr></thead>
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
    let currentOrcNumero = null;   // número gerado para o orçamento ativo

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

    function save() { database.ref(DB_PATH).set(db).catch(e => console.error("Erro Nuvem", e)); }

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
        } else { alert("Acesso Negado."); }
    }

    function openTab(id, btn) {
        document.querySelectorAll('.section-panel').forEach(s => s.classList.remove('active'));
        document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        btn.classList.add('active');
        document.getElementById('search-est-input').value = "";
        document.getElementById('search-cli-input').value = "";
        document.getElementById('search-hist').value = "";
        render();
    }

    // Cálculo automático de preço de venda (Custo + Markup + IPI + ICMS)
    function calcVenda() {
        const compra = parseFloat(document.getElementById('est-compra').value) || 0;
        const mark = parseFloat(document.getElementById('est-markup').value) || 0;
        const ipi = parseFloat(document.getElementById('est-ipi').value) || 0;
        const icms = parseFloat(document.getElementById('est-icms').value) || 0;
        
        const precoComMarkup = compra + (compra * (mark / 100));
        const total = precoComMarkup + (precoComMarkup * (ipi / 100)) + (precoComMarkup * (icms / 100));
        
        document.getElementById('est-val').value = total.toFixed(2);
    }

    function mostrarFormEstoque() { document.getElementById('form-estoque').style.display = 'grid'; }

    function addEstoque() {
        const item = {
            desc: document.getElementById('est-desc').value.trim(),
            unid: document.getElementById('est-unid').value.trim(),
            ncm: document.getElementById('est-ncm').value.trim(),
            cfop: document.getElementById('est-cfop').value.trim(),
            cst: document.getElementById('est-cst').value.trim(),
            compra: parseFloat(document.getElementById('est-compra').value) || 0,
            markup: parseFloat(document.getElementById('est-markup').value) || 0,
            ipi: parseFloat(document.getElementById('est-ipi').value) || 0,
            icms: parseFloat(document.getElementById('est-icms').value) || 0,
            val: parseFloat(document.getElementById('est-val').value) || 0,
            qtd: parseInt(document.getElementById('est-qtd').value) || 0
        };
        if(!item.desc) return;
        if(editingEstoqueIndex !== null) db.estoque[editingEstoqueIndex] = item;
        else db.estoque.push(item);
        cancelarEdicao(); 
        document.getElementById('search-est-input').value = "";
        save();
    }

    function editEstItem(idx) {
        const i = db.estoque[idx];
        document.getElementById('est-desc').value = i.desc;
        document.getElementById('est-unid').value = i.unid;
        document.getElementById('est-ncm').value = i.ncm;
        document.getElementById('est-cfop').value = i.cfop || "";
        document.getElementById('est-cst').value = i.cst || "";
        document.getElementById('est-compra').value = i.compra || 0;
        document.getElementById('est-markup').value = i.markup || 0;
        document.getElementById('est-ipi').value = i.ipi || 0;
        document.getElementById('est-icms').value = i.icms || 0;
        document.getElementById('est-val').value = i.val;
        document.getElementById('est-qtd').value = i.qtd;
        editingEstoqueIndex = idx; mostrarFormEstoque();
    }

    function cancelarEdicao() {
        editingEstoqueIndex = null; document.getElementById('form-estoque').style.display = 'none';
        ['est-desc','est-unid','est-ncm','est-cfop','est-cst','est-compra','est-markup','est-ipi','est-icms','est-val','est-qtd'].forEach(id => document.getElementById(id).value = '');
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
        cancelarEdicaoCliente(); 
        document.getElementById('search-cli-input').value = "";
        save();
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
        editingClienteIndex = idx; mostrarFormCliente();
    }

    function cancelarEdicaoCliente() {
        editingClienteIndex = null; document.getElementById('form-cliente').style.display = 'none';
        ['cli-nome','cli-razao','cli-doc','cli-tel','cli-email','cli-end','cli-cidade'].forEach(id => document.getElementById(id).value = '');
    }

    // ── Gerar número único do orçamento: DDMMAA-SEQ ─────────────────
    function gerarNumeroOrc() {
        const now = new Date();
        const dd  = String(now.getDate()).padStart(2,'0');
        const mm  = String(now.getMonth()+1).padStart(2,'0');
        const aa  = String(now.getFullYear()).slice(-2);
        const base = `${dd}${mm}${aa}`;
        // Conta quantos orçamentos do mesmo dia já existem
        const hoje = (db.historico || []).filter(h => h.numero && h.numero.startsWith(base));
        const seq  = String(hoje.length + 1).padStart(2,'0');
        return `${base}-${seq}`;
    }

    // ── Preencher dados do cliente na área de impressão ──────────────
    function preencherDadosCliente(nomeCliente) {
        const c = (db.clientes || []).find(x => x.nome === nomeCliente);
        document.getElementById('orc-dc-nome').innerText   = c ? `Cliente: ${c.nome}${c.razao ? ' | ' + c.razao : ''}` : (nomeCliente || '');
        document.getElementById('orc-dc-doc').innerText    = c ? (c.doc  || '—') : '—';
        document.getElementById('orc-dc-email').innerText  = c ? (c.email|| '—') : '—';
        document.getElementById('orc-dc-end').innerText    = c ? (c.end  || '—') : '—';
        document.getElementById('orc-dc-cidade').innerText = c ? `${c.cidade||''}/${c.uf||''}` : '—';
        document.getElementById('orc-dc-tel').innerText    = c ? (c.tel  || '—') : '—';
    }

    // ── ORÇAMENTO: produto selecionado temporariamente ──────────────
    let orcProdutoSelecionado = null;
    let orcResultadoIndex = -1;

    function filterItemsLive() {
        const term = document.getElementById('orc-search').value.trim().toUpperCase();
        const box = document.getElementById('orc-resultados');
        orcProdutoSelecionado = null;
        orcResultadoIndex = -1;

        if(!term || !db.estoque) { box.style.display = 'none'; return; }

        const found = db.estoque.filter(i => i.desc.toUpperCase().includes(term));
        if(found.length === 0) {
            box.innerHTML = '<div class="orc-sem-resultado">Nenhum produto encontrado</div>';
            box.style.display = 'block';
            return;
        }

        box.innerHTML = found.map((i, idx) =>
            `<div class="orc-res-item" data-desc="${i.desc}" data-idx="${idx}"
                  onmousedown="selecionarProdutoOrc('${i.desc.replace(/'/g,"\'")}')">
                <span>${i.desc}</span>
                <span class="orc-res-preco">R$ ${parseFloat(i.val).toFixed(2)}</span>
             </div>`
        ).join('');
        box.style.display = 'block';
    }

    function selecionarProdutoOrc(desc) {
        orcProdutoSelecionado = db.estoque.find(p => p.desc === desc);
        if(orcProdutoSelecionado) {
            document.getElementById('orc-search').value = orcProdutoSelecionado.desc;
            document.getElementById('orc-resultados').style.display = 'none';
            document.getElementById('orc-qtd-sel').focus();
            document.getElementById('orc-qtd-sel').select();
        }
    }

    function orcSearchKeydown(e) {
        const box = document.getElementById('orc-resultados');
        const items = box.querySelectorAll('.orc-res-item');
        if(e.key === 'ArrowDown') {
            e.preventDefault();
            orcResultadoIndex = Math.min(orcResultadoIndex + 1, items.length - 1);
            items.forEach((el, i) => el.classList.toggle('selected', i === orcResultadoIndex));
        } else if(e.key === 'ArrowUp') {
            e.preventDefault();
            orcResultadoIndex = Math.max(orcResultadoIndex - 1, 0);
            items.forEach((el, i) => el.classList.toggle('selected', i === orcResultadoIndex));
        } else if(e.key === 'Enter') {
            e.preventDefault();
            if(orcResultadoIndex >= 0 && items[orcResultadoIndex]) {
                selecionarProdutoOrc(items[orcResultadoIndex].dataset.desc);
            } else if(orcProdutoSelecionado) {
                addItemOrc();
            }
        } else if(e.key === 'Escape') {
            box.style.display = 'none';
        }
    }

    // Fechar dropdown ao clicar fora
    document.addEventListener('click', function(e) {
        if(!e.target.closest('.orc-search-wrapper')) {
            const box = document.getElementById('orc-resultados');
            if(box) box.style.display = 'none';
        }
    });

    function addItemOrc() {
        // Tenta usar o produto selecionado pelo autocomplete, ou busca pelo texto do campo
        let prod = orcProdutoSelecionado;
        if(!prod) {
            const term = document.getElementById('orc-search').value.trim().toUpperCase();
            if(term && db.estoque) prod = db.estoque.find(p => p.desc.toUpperCase() === term);
        }
        const qtd = parseInt(document.getElementById('orc-qtd-sel').value) || 1;
        if(!prod) { alert("Selecione um produto da lista!"); return; }
        if(!db.orc_temp) db.orc_temp = [];
        db.orc_temp.push({id: Date.now(), desc: prod.desc, val: prod.val, qtd: qtd});
        // Limpar campos de busca para facilitar próxima inserção
        document.getElementById('orc-search').value = '';
        document.getElementById('orc-resultados').style.display = 'none';
        document.getElementById('orc-qtd-sel').value = 1;
        orcProdutoSelecionado = null;
        // Feedback visual na tabela
        save();
        document.getElementById('orc-search').focus();
    }

    function novoOrcamento() {
        const modal = document.getElementById('modal-confirmacao');
        if(modal) {
            document.getElementById('modal-msg').innerText = "Limpar orçamento atual e começar um novo?";
            modal.style.display = 'flex';
            document.getElementById('modal-btn-sim').onclick = function() {
                modal.style.display = 'none';
                _limparOrcamento();
            };
            document.getElementById('modal-btn-nao').onclick = function() {
                modal.style.display = 'none';
            };
        } else { _limparOrcamento(); }
    }

    function _limparOrcamento() {
        db.orc_temp = [];
        currentEditingOrcId = null;
        orcProdutoSelecionado = null;
        currentOrcNumero = gerarNumeroOrc();   // ← gera número ao abrir novo orçamento
        const cliSel = document.getElementById('orc-cli-sel');
        if(cliSel) cliSel.value = "";
        ['orc-perc-acess','orc-mo-fixo','orc-perc-mo'].forEach(id => {
            const el = document.getElementById(id); if(el) el.value = '';
        });
        document.getElementById('orc-search').value = '';
        document.getElementById('orc-resultados').style.display = 'none';
        document.getElementById('orc-qtd-sel').value = 1;
        const orcArea = document.getElementById('orc-area');
        if(orcArea) orcArea.classList.add('hide-table');
        ['res-materiais','res-acessorios','res-mo'].forEach(id => {
            const el = document.getElementById(id); if(el) el.innerText = 'R$ 0.00';
        });
        const totalEl = document.getElementById('res-total-final');
        if(totalEl) totalEl.innerText = 'R$ 0.00';
        // Limpa dados cliente impressos
        ['orc-dc-nome','orc-dc-doc','orc-dc-email','orc-dc-end','orc-dc-cidade','orc-dc-tel','orc-num-impresso'].forEach(id => {
            const el = document.getElementById(id); if(el) el.innerText = '';
        });
        document.getElementById('orc-num-badge').innerText = `Nº ${currentOrcNumero}`;
        document.getElementById('orc-num-impresso').innerText = `Nº ${currentOrcNumero}  |  Data: ${new Date().toLocaleDateString('pt-BR')}`;
        save();
    }

    function removerItemOrc(idx) {
        if(db.orc_temp) { db.orc_temp.splice(idx, 1); save(); }
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
        const cliVal = document.getElementById('orc-cli-sel').value;
        // Preenche dados do cliente na área impressa
        if(cliVal) preencherDadosCliente(cliVal);
        // Exibe número do orçamento
        if(currentOrcNumero) {
            document.getElementById('orc-num-badge').innerText  = `Nº ${currentOrcNumero}`;
            document.getElementById('orc-num-impresso').innerText = `Nº ${currentOrcNumero}  |  Data: ${new Date().toLocaleDateString('pt-BR')}`;
        }
        return total;
    }

    function finalizarERegistrar() {
        const cli = document.getElementById('orc-cli-sel').value;
        if(!cli) return mostrarModalInfo("⚠️ Selecione um cliente antes de finalizar!");
        if(!db.orc_temp || db.orc_temp.length === 0) return mostrarModalInfo("⚠️ Adicione ao menos um produto ao orçamento!");
        if(!db.historico) db.historico = [];
        // Preserva o status original se for edição
        let statusAtual = 'Pendente';
        if(currentEditingOrcId) {
            const existente = db.historico.find(x => x.id === currentEditingOrcId);
            if(existente) statusAtual = existente.status;
        }
        // Número: reutiliza se for edição, senão usa o currentOrcNumero
        const numero = currentEditingOrcId
            ? (db.historico.find(x => x.id === currentEditingOrcId)?.numero || currentOrcNumero)
            : (currentOrcNumero || gerarNumeroOrc());
        const orcData = {
            id:        currentEditingOrcId || Date.now(),
            numero:    numero,
            data:      new Date().toLocaleString(),
            cliente:   cli,
            total:     calculateTotal(),
            status:    statusAtual,
            itens:     [...db.orc_temp],
            perc_acess: document.getElementById('orc-perc-acess').value,
            mo_fixo:    document.getElementById('orc-mo-fixo').value,
            perc_mo:    document.getElementById('orc-perc-mo').value
        };
        if(currentEditingOrcId) {
            const idx = db.historico.findIndex(x => x.id === currentEditingOrcId);
            if(idx >= 0) db.historico[idx] = orcData;
        } else {
            db.historico.push(orcData);
        }
        db.orc_temp = []; currentEditingOrcId = null; currentOrcNumero = null;
        save();
        mostrarModalInfo(`✅ Orçamento <strong>Nº ${numero}</strong> registrado com sucesso!`);
    }

    function updateStatus(id, newStatus) {
        const h = db.historico.find(x => x.id === id);
        if(!h || h.status === newStatus) return;
        if(newStatus === 'Aprovado') {
            // Verifica estoque antes de aprovar
            const semEstoque = (h.itens || []).filter(it => {
                const p = db.estoque.find(e => e.desc === it.desc);
                return p && p.qtd < it.qtd;
            });
            const msgAlerta = semEstoque.length > 0
                ? `⚠️ Atenção: estoque insuficiente para:<br>${semEstoque.map(i=>i.desc).join('<br>')}<br><br>Deseja aprovar mesmo assim?`
                : `Confirmar aprovação do Nº ${h.numero||h.id}?<br>O estoque será reduzido automaticamente.`;
            mostrarModalConfirm(msgAlerta, () => {
                (h.itens || []).forEach(it => {
                    const p = db.estoque.find(e => e.desc === it.desc);
                    if(p) {
                        p.qtd -= it.qtd;
                        if(!db.movimentacoes) db.movimentacoes = [];
                        db.movimentacoes.push({data: new Date().toLocaleString(), produto: it.desc, qtd: it.qtd, destino: h.cliente});
                    }
                });
                h.status = 'Aprovado'; save();
            }, () => { render(); });
        } else {
            h.status = newStatus; save();
        }
    }

    function editOrcHist(id) {
        const h = db.historico.find(x => x.id === id);
        if(h) {
            db.orc_temp = [...h.itens];
            currentEditingOrcId = h.id;
            currentOrcNumero = h.numero || gerarNumeroOrc();
            // Navega para aba orçamento pelo id, sem depender de índice fixo
            const btnOrc = document.querySelector('[onclick*="tab-orcamento"]');
            openTab('tab-orcamento', btnOrc);
            document.getElementById('orc-cli-sel').value = h.cliente;
            document.getElementById('orc-perc-acess').value = h.perc_acess || 0;
            document.getElementById('orc-mo-fixo').value = h.mo_fixo || 0;
            document.getElementById('orc-perc-mo').value = h.perc_mo || 0;
            // Atualiza badge e dados do cliente
            document.getElementById('orc-num-badge').innerText = `Nº ${currentOrcNumero}`;
            preencherDadosCliente(h.cliente);
            render();
        }
    }

    function addNewUser() {
        const u = document.getElementById('new-user').value.trim();
        const p = document.getElementById('new-pass').value.trim();
        if(!u || !p) return mostrarModalInfo("⚠️ Preencha usuário e senha!");
        if(db.users && db.users.find(x => x.user.toLowerCase() === u.toLowerCase()))
            return mostrarModalInfo("⚠️ Usuário já existe!");
        if(!db.users) db.users = [];
        db.users.push({user: u, pass: p, level: 'comum'});
        document.getElementById('new-user').value = '';
        document.getElementById('new-pass').value = '';
        save();
        mostrarModalInfo(`✅ Usuário <strong>${u}</strong> cadastrado!`);
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
        mostrarModalConfirm("⚠️ Isso apagará TODO o histórico e logs de movimentação.<br>Esta ação não pode ser desfeita. Confirma?", () => {
            db.historico = []; db.movimentacoes = []; save();
            mostrarModalInfo("✅ Histórico e logs limpos.");
        });
    }

    function forceRealtimeSync() { initCloud(); alert("Reiniciado!"); }

    function render(filter) {
        if(!db) return;

        // ESTOQUE
        const sEst = document.getElementById('search-est-input').value.toUpperCase();
        const tableEst = document.getElementById('table-estoque');
        const estTbody = document.getElementById('tbl-estoque-corpo');
        if(estTbody) {
            estTbody.innerHTML = '';
            if(sEst !== "" || filter === 'all'){
                tableEst.classList.remove('hide-table');
                if(db.estoque) db.estoque.forEach((item, idx) => {
                    if(filter === 'all' || item.desc.toUpperCase().includes(sEst)) {
                        estTbody.innerHTML += `<tr><td>${item.desc}</td><td>${item.unid}</td><td>${item.ncm}</td><td>${item.cfop || ""}</td><td>${item.cst || ""}</td><td>R$ ${parseFloat(item.val).toFixed(2)}</td><td>${item.qtd}</td><td class="no-print"><div class="acoes-inline"><button class="btn btn-edit" onclick="editEstItem(${idx})">EDIT</button><button class="btn btn-del" onclick="excluirEstoque(${idx})">DEL</button></div></td></tr>`;
                    }
                });
            } else { tableEst.classList.add('hide-table'); }
        }

        // CLIENTES
        const sCli = document.getElementById('search-cli-input').value.toUpperCase();
        const tableCli = document.getElementById('table-clientes');
        const cliTbody = document.getElementById('tbl-clientes-corpo');
        if(cliTbody) {
            cliTbody.innerHTML = '';
            if(sCli !== "" || filter === 'all'){
                tableCli.classList.remove('hide-table');
                if(db.clientes) db.clientes.forEach((c, idx) => {
                    if(filter === 'all' || c.nome.toUpperCase().includes(sCli)) {
                        cliTbody.innerHTML += `<tr><td><strong>${c.nome}</strong><br><small>${c.doc}</small></td><td>${c.tel}</td><td>${c.cidade}/${c.uf}</td><td>${c.ultima_compra || '---'}</td><td class="no-print"><button class="btn btn-edit" onclick="editCliente(${idx})">EDIT</button></td></tr>`;
                    }
                });
            } else { tableCli.classList.add('hide-table'); }
        }

        // ORÇAMENTO - preserva seleção do cliente durante sync
        const orcCliSel = document.getElementById('orc-cli-sel');
        if(orcCliSel && db.clientes) {
            const current = orcCliSel.value;
            orcCliSel.innerHTML = '<option value="">Selecionar Cliente</option>';
            db.clientes.forEach(c => { orcCliSel.innerHTML += `<option value="${c.nome}" ${current === c.nome ? 'selected' : ''}>${c.nome} - ${c.cidade||''}</option>`; });
            if(current) orcCliSel.value = current; // garante que preserva mesmo após innerHTML
        }
        const orcTbody = document.getElementById('orc-lista-corpo');
        const orcArea = document.getElementById('orc-area');
        if(orcTbody) {
            orcTbody.innerHTML = '';
            if(db.orc_temp && db.orc_temp.length > 0) {
                orcArea.classList.remove('hide-table');
                db.orc_temp.forEach((it, idx) => {
                    orcTbody.innerHTML += `<tr><td>${it.qtd}</td><td>${it.desc}</td><td>R$ ${parseFloat(it.val).toFixed(2)}</td><td>R$ ${(it.val * it.qtd).toFixed(2)}</td><td class="no-print"><button class="btn btn-del" onclick="removerItemOrc(${idx})">✕</button></td></tr>`;
                });
            } else { orcArea.classList.add('hide-table'); }
            calculateTotal();
        }

        // HISTÓRICO
        const sHist = document.getElementById('search-hist').value.toUpperCase();
        const tableHist = document.getElementById('table-historico');
        const histTbody = document.getElementById('tbl-historico-corpo');
        if(histTbody) {
            histTbody.innerHTML = '';
            if(sHist !== ""){
                tableHist.classList.remove('hide-table');
                const filtroStatus = document.getElementById('filter-hist-status') ? document.getElementById('filter-hist-status').value : '';
                if(db.historico) db.historico.slice().reverse().forEach((h) => {
                    const matchTxt = h.cliente.toUpperCase().includes(sHist) || (h.numero && h.numero.toUpperCase().includes(sHist));
                    const matchSts = !filtroStatus || h.status === filtroStatus;
                    if(matchTxt && matchSts) {
                        const statusColor = h.status==='Aprovado' ? '#28a745' : h.status==='Cancelado' ? '#dc3545' : '#ff9800';
                        histTbody.innerHTML += `<tr>
                            <td><strong style="color:var(--asb-blue)">${h.numero||'—'}</strong></td>
                            <td>${h.data}</td>
                            <td>${h.cliente}</td>
                            <td>R$ ${parseFloat(h.total).toFixed(2)}</td>
                            <td><select onchange="updateStatus(${h.id}, this.value)" style="border-color:${statusColor}; color:${statusColor}; font-weight:600">
                                <option ${h.status==='Pendente'?'selected':''}>Pendente</option>
                                <option ${h.status==='Aprovado'?'selected':''}>Aprovado</option>
                                <option ${h.status==='Cancelado'?'selected':''}>Cancelado</option>
                            </select></td>
                            <td class="no-print">
                                <button class="btn btn-edit" onclick="editOrcHist(${h.id})">EDITAR</button>
                                <button class="btn btn-del" onclick="excluirOrcamento(${h.id})">DEL</button>
                            </td></tr>`;
                    }
                });
            } else { tableHist.classList.add('hide-table'); }
        }

        // MOVIMENTAÇÃO
        const logTbody = document.getElementById('tbl-log-corpo');
        if(logTbody) {
            logTbody.innerHTML = '';
            if(db.movimentacoes) db.movimentacoes.slice().reverse().forEach(m => {
                logTbody.innerHTML += `<tr><td>${m.data}</td><td>${m.produto}</td><td>${m.qtd}</td><td>${m.destino}</td></tr>`;
            });
        }
        
        // USUÁRIOS
        const userTbody = document.getElementById('tbl-users-corpo');
        if(userTbody) {
            userTbody.innerHTML = '';
            if(db.users) db.users.forEach((u, idx) => {
                userTbody.innerHTML += `<tr><td>${u.user}</td><td>${u.level}</td><td><button class="btn btn-del" onclick="excluirUsuario(${idx})">DEL</button></td></tr>`;
            });
        }
    }
</script>

<!-- Modal de Confirmacao -->
<div id="modal-confirmacao">
    <div class="modal-box">
        <p id="modal-msg">Tem certeza?</p>
        <div class="modal-btns">
            <button id="modal-btn-sim">SIM, LIMPAR</button>
            <button id="modal-btn-nao">CANCELAR</button>
        </div>
    </div>
</div>
</body>
</html>

