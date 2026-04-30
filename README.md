<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ESSENCIO - Gestão Ativa</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --dourado: #BA9D58; --fundo: #0a0e0a; --card: #151a15; --borda: #2a332a; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--fundo); color: white; margin: 0; padding-bottom: 120px; -webkit-font-smoothing: antialiased; }
        
        .input-container { background: var(--card); border-radius: 16px; padding: 12px 20px; margin-bottom: 16px; border: 1px solid var(--borda); transition: 0.3s; }
        .input-container:focus-within { border-color: var(--dourado); }
        .label-elite { font-size: 10px; font-weight: 800; color: var(--dourado); text-transform: uppercase; letter-spacing: 1px; margin-bottom: 4px; display: block; }
        .input-elite { background: transparent; border: none; width: 100%; color: white; font-weight: 900; font-size: 14px; text-transform: uppercase; outline: none; }
        .input-elite::placeholder { color: #444; }

        .btn-principal { height: 60px; border-radius: 16px; font-weight: 900; font-size: 14px; text-transform: uppercase; width: 100%; background: transparent; color: white; border: 2px solid var(--dourado); cursor: pointer; transition: 0.2s; }
        .btn-principal:active { background: var(--dourado); color: black; transform: scale(0.98); }
        
        .bottom-nav { position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%); background: var(--card); border: 1px solid var(--dourado); border-radius: 30px; display: flex; gap: 40px; padding: 12px 40px; z-index: 1000; box-shadow: 0 10px 30px rgba(0,0,0,0.9); }
        .nav-icon { color: #555; cursor: pointer; transition: 0.3s; }
        .nav-icon.active { color: var(--dourado); }
        .view-hidden { display: none !important; }

        /* Estilo dos Cards */
        .clash-card { position: relative; background: #0a0e0a; border-radius: 12px; border: 2px solid #333; overflow: hidden; cursor: pointer; transition: 0.2s; display: flex; flex-direction: column; }
        .clash-card.selected { border: 3px solid var(--dourado); box-shadow: 0 0 15px var(--dourado); z-index: 10; }
        .clash-card-header { height: 80px; position: relative; display: flex; align-items: center; justify-content: center; border-bottom: 2px solid var(--dourado); background: linear-gradient(to bottom, #2a2a2a, #0a0e0a); }
        .clash-img { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; opacity: 0.4; }
        .clash-elixir { position: absolute; top: -6px; left: -6px; background: var(--dourado); color: black; width: 28px; height: 28px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: 900; font-size: 12px; border: 3px solid #0a0e0a; z-index: 20; }
        .clash-delete { position: absolute; top: 4px; right: 4px; background: rgba(0,0,0,0.6); color: #ff4444; width: 24px; height: 24px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-weight: bold; border: 1px solid #ff4444; z-index: 30; }

        #action-bar { position: fixed; bottom: 90px; left: 50%; transform: translateX(-50%); background: var(--dourado); color: black; padding: 10px 20px; border-radius: 20px; display: none; gap: 20px; font-weight: 900; font-size: 12px; z-index: 2000; }
        #toast { visibility: hidden; background: var(--dourado); color: black; border-radius: 30px; padding: 15px 20px; position: fixed; left: 50%; top: 20px; transform: translateX(-50%); font-weight: 900; opacity: 0; transition: 0.3s; z-index: 5000; }
        #toast.show { visibility: visible; opacity: 1; top: 40px; }
        
        .historico-icon { color: var(--dourado); font-size: 9px; cursor: help; text-decoration: underline; margin-left: 4px; }
    </style>
</head>
<body>

    <div id="toast">PROCESSANDO...</div>

    <div id="action-bar">
        <button onclick="excluirSelecionados()">🗑️ EXCLUIR</button>
        <span class="opacity-30">|</span>
        <button onclick="movimentarSelecionados()">🚚 MOVER</button>
    </div>

    <header class="pt-12 pb-6 text-center flex flex-col items-center">
        <div class="w-20 h-20 bg-white border-[3px] border-[#BA9D58] rounded-[1.5rem] flex items-center justify-center mb-4 overflow-hidden shadow-lg">
            <img src="https://i.imgur.com/LNNpmQr.jpeg" alt="Logo" class="w-full h-full object-contain p-2">
        </div>
        <h1 class="font-black text-2xl tracking-[0.2em] text-[#BA9D58]">ESSENCIO</h1>
        <h2 id="status-nuvem" class="text-[9px] font-bold tracking-[0.3em] text-white uppercase mt-2">Gestão de Ativos | Nuvem</h2>
    </header>

    <main id="view-form" class="px-6 max-w-md mx-auto">
        <div class="relative h-32 mb-8 bg-[#151a15] rounded-2xl border border-[#2a332a] flex items-center justify-center overflow-hidden">
            <input type="file" id="input-foto" accept="image/*" capture="environment" class="absolute inset-0 opacity-0 z-20 cursor-pointer">
            <img id="preview" class="absolute inset-0 w-full h-full object-cover hidden z-10 opacity-70">
            <div id="instrucao" class="text-center z-0">
                <svg class="w-8 h-8 text-[#BA9D58] mx-auto mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"></path><path d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                <span class="text-white text-[10px] font-black uppercase">Scan Visual</span>
            </div>
        </div>

        <form id="form-registro" data-edit-mode="false">
            <div class="input-container"><label class="label-elite">Responsável pelo Registro</label><input type="text" id="f-user" placeholder="Seu nome" class="input-elite" required></div>
            <div class="input-container"><label class="label-elite">Modelo/NOME do Ativo</label><input type="text" id="f-nome" placeholder="Equipamento" class="input-elite" required></div>
            <div class="input-container"><label class="label-elite">Marca / Modelo Técnico</label><input type="text" id="f-modelo" placeholder="Ex: Bosch, Dell..." class="input-elite"></div>
            <div class="flex gap-4">
                <div class="input-container flex-1"><label class="label-elite">Patrimônio</label><input type="text" id="f-pt" placeholder="PT-000" class="input-elite"></div>
                <div class="input-container flex-1"><label class="label-elite">Série</label><input type="text" id="f-sn" placeholder="SN-000" class="input-elite"></div>
            </div>
            <div class="flex gap-4">
                <div class="input-container flex-1"><label class="label-elite">QTD</label><input type="number" id="f-qtd" value="1" class="input-elite"></div>
                <div class="input-container flex-1"><label class="label-elite">Nota Fiscal</label><input type="text" id="f-nf" placeholder="NF-000" class="input-elite"></div>
            </div>
            <div class="input-container"><label class="label-elite">Part Number</label><input type="text" id="f-pn" placeholder="PN-000" class="input-elite"></div>
            <div class="input-container"><label class="label-elite">Localização / Destino</label><input type="text" id="f-local" placeholder="Alocação atual" class="input-elite"></div>
            <button type="submit" id="btn-submit" class="btn-principal mt-4 mb-24">Confirmar Registro</button>
        </form>
    </main>

    <main id="view-collection" class="px-4 max-w-md mx-auto view-hidden">
        <div class="flex justify-between items-end mb-6 border-b border-[#2a332a] pb-2 px-2">
            <h3 class="text-white font-black text-sm uppercase">Estoque Ativo</h3>
            <span id="contador-ativos" class="text-[#BA9D58] font-black text-sm bg-[#111] px-2 py-1 rounded">0</span>
        </div>
        <div id="lista-ativos" class="grid grid-cols-2 gap-4 pb-28"></div>
    </main>

    <nav class="bottom-nav">
        <button id="nav-home" class="nav-icon active" onclick="switchTab('form')">
            <svg class="w-7 h-7" fill="currentColor" viewBox="0 0 24 24"><path d="M10 20v-6h4v6h5v-8h3L12 3 2 12h3v8z"/></svg>
        </button>
        <button id="nav-collection" class="nav-icon" onclick="switchTab('collection')">
            <svg class="w-7 h-7" fill="currentColor" viewBox="0 0 24 24"><path d="M4 6h16v2H4zm0 5h16v2H4zm0 5h16v2H4z"/></svg>
        </button>
    </nav>

    <script>
        // COLOQUE SEU LINK DO GOOGLE APPS SCRIPT AQUI
        const URL_GOOGLE_SHEETS = "https://script.google.com/macros/s/AKfycbzkQsRPcdEUxsNiyEmZ03KcbFwZCBLkHYVV5TmBEMfTtMwn38OPaXpR3735wl2aBVUX/exec"; 
        
        let fotoMiniatura = "";
        let selecionados = [];
        let dadosGlobais = [];

        function switchTab(tab) {
            document.getElementById('view-form').classList.toggle('view-hidden', tab !== 'form');
            document.getElementById('view-collection').classList.toggle('view-hidden', tab !== 'collection');
            document.getElementById('nav-home').classList.toggle('active', tab === 'form');
            document.getElementById('nav-collection').classList.toggle('active', tab === 'collection');
            if(tab === 'collection') carregarDados();
        }

        async function carregarDados() {
            try {
                const res = await fetch(URL_GOOGLE_SHEETS);
                dadosGlobais = await res.json();
                renderizar(dadosGlobais);
                document.getElementById('status-nuvem').innerText = "Gestão de Ativos | Nuvem";
            } catch (e) { document.getElementById('status-nuvem').innerText = "Modo Local"; }
        }

        function renderizar(dados) {
            const lista = document.getElementById('lista-ativos');
            lista.innerHTML = "";
            document.getElementById('contador-ativos').innerText = dados.length;
            
            dados.slice().reverse().forEach((item, reversoIndex) => {
                const originalIndex = dados.length - 1 - reversoIndex;
                const card = document.createElement('div');
                card.className = "clash-card";
                card.onclick = () => toggleSelect(item.pt, item.sn, card);
                
                const temHistorico = item.local.includes("(ex:");
                const localExibicao = item.local.split(" (ex:")[0];

                card.innerHTML = `
                    <div class="clash-elixir">${item.qtd}</div>
                    <button onclick="deletarAtivo('${item.pt}', '${item.sn}', event)" class="clash-delete">X</button>
                    <div class="clash-card-header">
                        <img src="${item.foto}" class="clash-img" onerror="this.style.display='none'">
                        <h4 class="text-white font-black text-center uppercase text-[10px] px-2 z-10 leading-tight">${item.nome}</h4>
                    </div>
                    <div class="clash-card-body p-2 flex-1 flex flex-col"> 
                        <span class="text-[#BA9D58] text-[8px] font-black uppercase text-center mb-1 truncate">${item.modelo}</span>
                        <div class="bg-[#151a15] rounded p-1 border border-[#2a332a] text-[7px] font-bold uppercase mb-1">
                            <div class="flex justify-between"><span>PT:</span><span>${item.pt}</span></div>
                            <div class="flex justify-between"><span>SN:</span><span>${item.sn}</span></div>
                            <div class="flex justify-between pt-0.5 border-t border-[#2a332a] mt-0.5"><span>NF:</span><span>${item.nf}</span></div>
                        </div>
                        <div class="text-[7px] text-[#BA9D58] font-black uppercase">👤 ${item.user}</div>
                        <div class="text-[7px] text-white/50 font-bold flex items-center">
                            📍 ${localExibicao} ${temHistorico ? `<span class="historico-icon" onclick="alert('Histórico: ${item.local}')">📜 Histórico</span>` : ''}
                        </div>
                        <div class="text-[7px] text-[#555] text-center font-bold mt-auto pt-2">${item.data} - ${item.hora}</div>
                        <button onclick="prepararEdicao(${originalIndex}, event)" class="w-full mt-2 border border-[#BA9D58] text-[#BA9D58] text-[7px] font-black py-1 rounded">✏️ EDITAR</button>
                    </div>
                `;
                lista.appendChild(card);
            });
        }

        // --- LÓGICA DE SELEÇÃO E AÇÕES ---
        function toggleSelect(pt, sn, element) {
            const index = selecionados.findIndex(i => i.pt === pt && i.sn === sn);
            if(index > -1) {
                selecionados.splice(index, 1);
                element.classList.remove('selected');
            } else {
                selecionados.push({pt, sn});
                element.classList.add('selected');
            }
            document.getElementById('action-bar').style.display = selecionados.length > 0 ? 'flex' : 'none';
        }

        async function excluirSelecionados() {
            if(!confirm("Excluir itens selecionados?")) return;
            showToast("EXCLUINDO...");
            for(let item of selecionados) {
                await fetch(URL_GOOGLE_SHEETS, { method: 'POST', mode: 'no-cors', body: JSON.stringify({ action: 'delete', pt: item.pt, sn: item.sn }) });
            }
            carregarDados();
            selecionados = [];
            document.getElementById('action-bar').style.display = 'none';
        }

        async function movimentarSelecionados() {
            const novoLocal = prompt("Novo destino:");
            if(!novoLocal) return;
            showToast("MOVIMENTANDO...");
            for(let item of selecionados) {
                await fetch(URL_GOOGLE_SHEETS, { 
                    method: 'POST', 
                    mode: 'no-cors', 
                    body: JSON.stringify({ action: 'move', origPt: item.pt, origSn: item.sn, novoLocal: novoLocal }) 
                });
            }
            carregarDados();
            selecionados = [];
            document.getElementById('action-bar').style.display = 'none';
        }

        // --- EDIÇÃO ---
        function prepararEdicao(idx, event) {
            event.stopPropagation();
            const item = dadosGlobais[idx];
            document.getElementById('f-user').value = item.user;
            document.getElementById('f-nome').value = item.nome;
            document.getElementById('f-modelo').value = item.modelo;
            document.getElementById('f-pt').value = item.pt;
            document.getElementById('f-sn').value = item.sn;
            document.getElementById('f-qtd').value = item.qtd;
            document.getElementById('f-nf').value = item.nf;
            document.getElementById('f-pn').value = item.pn;
            document.getElementById('f-local').value = item.local;
            
            fotoMiniatura = item.foto;
            document.getElementById('preview').src = item.foto;
            document.getElementById('preview').classList.remove('hidden');
            document.getElementById('instrucao').classList.add('hidden');

            const form = document.getElementById('form-registro');
            form.dataset.editMode = "true";
            form.dataset.origPt = item.pt;
            form.dataset.origSn = item.sn;
            document.getElementById('btn-submit').innerText = "SALVAR ALTERAÇÕES";
            switchTab('form');
        }

        // --- SUBMIT ---
        document.getElementById('form-registro').onsubmit = async (e) => {
            e.preventDefault();
            const form = e.target;
            const isEdit = form.dataset.editMode === "true";
            const btn = document.getElementById('btn-submit');
            btn.disabled = true;

            const dados = {
                action: isEdit ? 'edit' : 'insert',
                origPt: form.dataset.origPt || "",
                origSn: form.dataset.origSn || "",
                user: document.getElementById('f-user').value,
                nome: document.getElementById('f-nome').value,
                modelo: document.getElementById('f-modelo').value,
                qtd: document.getElementById('f-qtd').value,
                nf: document.getElementById('f-nf').value,
                pt: document.getElementById('f-pt').value,
                pn: document.getElementById('f-pn').value,
                sn: document.getElementById('f-sn').value,
                local: document.getElementById('f-local').value,
                foto: fotoMiniatura,
                data: new Date().toLocaleDateString('pt-BR'),
                hora: new Date().toLocaleTimeString('pt-BR', {hour:'2-digit', minute:'2-digit'})
            };

            await fetch(URL_GOOGLE_SHEETS, { method: 'POST', mode: 'no-cors', body: JSON.stringify(dados) });
            
            showToast(isEdit ? "ATUALIZADO!" : "CADASTRADO!");
            form.reset();
            form.dataset.editMode = "false";
            btn.innerText = "Confirmar Registro";
            btn.disabled = false;
            fotoMiniatura = "";
            document.getElementById('preview').classList.add('hidden');
            document.getElementById('instrucao').classList.remove('hidden');
            switchTab('collection');
        };

        // --- ÚTEIS ---
        function showToast(msg) {
            const t = document.getElementById('toast');
            t.innerText = msg; t.classList.add('show');
            setTimeout(() => t.classList.remove('show'), 2500);
        }

        document.getElementById('input-foto').onchange = function(e) {
            const reader = new FileReader();
            reader.onload = function(ev) {
                const img = new Image();
                img.onload = function() {
                    const canvas = document.createElement('canvas');
                    const ctx = canvas.getContext('2d');
                    canvas.width = 200; canvas.height = img.height * (200 / img.width);
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    fotoMiniatura = canvas.toDataURL('image/jpeg', 0.6);
                    document.getElementById('preview').src = fotoMiniatura;
                    document.getElementById('preview').classList.remove('hidden');
                    document.getElementById('instrucao').classList.add('hidden');
                };
                img.src = ev.target.result;
            };
            reader.readAsDataURL(e.target.files[0]);
        };
    </script>
</body>
</html>
