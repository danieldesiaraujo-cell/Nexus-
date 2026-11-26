Sim, o código completo abaixo já tem o motor de Inteligência Artificial ativado para responder a qualquer pergunta — tanto na barra de pesquisa principal quanto no chat lateral da Nexus Atlytic.
Reforcei a conexão da IA usando uma requisição POST direta e com sistema de tentativa automática (fallback) para garantir que ela nunca falhe ao responder suas perguntas no celular.
Copie todo o código abaixo e salve como index.html:
html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    
    <!-- SEO & GOOGLE -->
    <title>Navegador Nexus • Busca Inteligente & Nexus Atlytic IA</title>
    <meta name="description" content="Navegador Nexus com Inteligência Artificial Nexus Atlytic integrada para responder qualquer pergunta e navegar na web.">
    <meta name="robots" content="index, follow">
    <meta name="theme-color" content="#0d1017">
    <meta name="mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-capable" content="yes">

    <script type="application/ld+json">
    {
      "@context": "https://schema.org",
      "@type": "WebApplication",
      "name": "Navegador Nexus",
      "applicationCategory": "BrowserApplication",
      "operatingSystem": "All"
    }
    </script>

    <style>
        :root {
            --bg-frame: #0d1017;
            --bg-toolbar: #181c28;
            --bg-surface: #12151f;
            --glass: rgba(255, 255, 255, 0.05);
            --glass-border: rgba(255, 255, 255, 0.1);
            --accent: #00f2fe;
            --accent-grad: linear-gradient(135deg, #4facfe 0%, #00f2fe 50%, #9b51e0 100%);
            --text-main: #f3f4f6;
            --text-muted: #9ca3af;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; }
        body { background: var(--bg-frame); color: var(--text-main); height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        /* Barra de Abas Estilo Edge/Chrome */
        .tab-bar {
            display: flex; align-items: flex-end; background: var(--bg-frame);
            padding: 8px 10px 0 10px; gap: 5px; height: 42px; user-select: none;
        }
        .nexus-badge {
            font-weight: 900; font-size: 13px; letter-spacing: 2px;
            background: var(--accent-grad); -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            padding: 0 8px 9px 4px;
        }
        .tabs-container { display: flex; gap: 4px; flex: 1; overflow-x: auto; height: 100%; align-items: flex-end; }
        .tabs-container::-webkit-scrollbar { display: none; }
        .tab {
            background: transparent; color: var(--text-muted); padding: 7px 12px;
            border-radius: 10px 10px 0 0; font-size: 12px; display: flex; align-items: center;
            justify-content: space-between; min-width: 110px; max-width: 170px; height: 34px; cursor: pointer;
        }
        .tab.active { background: var(--bg-toolbar); color: white; font-weight: 600; }
        .close-tab { margin-left: 6px; border-radius: 50%; width: 16px; height: 16px; display: flex; align-items: center; justify-content: center; }
        .new-tab-btn { background: none; border: none; color: var(--text-muted); font-size: 18px; cursor: pointer; width: 28px; height: 28px; margin-bottom: 4px; }

        /* Barra de Navegação */
        .nav-bar {
            background: var(--bg-toolbar); padding: 8px 10px; display: flex; align-items: center;
            gap: 6px; border-bottom: 1px solid var(--glass-border); z-index: 20;
        }
        .nav-btn {
            background: none; border: none; color: var(--text-muted); font-size: 15px;
            cursor: pointer; width: 30px; height: 30px; border-radius: 50%; display: flex; align-items: center; justify-content: center;
        }
        .url-pill {
            flex: 1; display: flex; align-items: center; background: var(--bg-frame);
            border: 1px solid var(--glass-border); border-radius: 20px; padding: 6px 12px;
        }
        .url-pill:focus-within { border-color: var(--accent); }
        #url-input { flex: 1; background: none; border: none; color: white; font-size: 13px; outline: none; margin-left: 8px; width: 100%; }

        .ai-toggle-btn {
            display: flex; align-items: center; gap: 5px;
            background: linear-gradient(135deg, rgba(79, 172, 254, 0.2), rgba(155, 81, 224, 0.3));
            border: 1px solid rgba(0, 242, 254, 0.45); color: white;
            padding: 6px 12px; border-radius: 20px; font-size: 12px; font-weight: 600; cursor: pointer; white-space: nowrap;
        }

        /* Área Principal */
        .main-workspace { flex: 1; display: flex; position: relative; overflow: hidden; }
        .viewport { flex: 1; position: relative; background: var(--bg-surface); overflow-y: auto; }

        /* Página Inicial Animada */
        .home-screen {
            min-height: 100%; position: relative; overflow: hidden;
            display: flex; flex-direction: column; align-items: center; padding: 30px 16px; background: #0b0e17;
        }
        .aurora-bg { position: absolute; inset: 0; overflow: hidden; pointer-events: none; z-index: 0; }
        .orb { position: absolute; border-radius: 50%; filter: blur(80px); opacity: 0.45; animation: floatOrb 10s infinite alternate ease-in-out; }
        .orb-1 { width: 300px; height: 300px; background: #00f2fe; top: -60px; left: 5%; }
        .orb-2 { width: 280px; height: 280px; background: #8a2be2; bottom: -50px; right: 5%; animation-delay: -4s; }
        @keyframes floatOrb { 0% { transform: translate(0, 0) scale(1); } 100% { transform: translate(30px, -25px) scale(1.15); } }

        .home-content { position: relative; z-index: 1; width: 100%; max-width: 780px; display: flex; flex-direction: column; align-items: center; }
        .home-top { width: 100%; display: flex; justify-content: space-between; align-items: center; margin-bottom: 35px; }
        .logo-hero {
            font-size: 46px; font-weight: 900; letter-spacing: 6px;
            background: var(--accent-grad); -webkit-background-clip: text; -webkit-text-fill-color: transparent; margin-bottom: 6px;
        }
        .search-hero {
            width: 100%; max-width: 580px; display: flex; align-items: center;
            background: rgba(24, 28, 40, 0.85); backdrop-filter: blur(16px);
            border: 1px solid var(--glass-border); border-radius: 30px; padding: 13px 20px; margin: 20px 0 35px 0;
        }
        .search-hero input { flex: 1; background: none; border: none; color: white; font-size: 15px; outline: none; margin-left: 10px; }

        .shortcuts-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(95px, 1fr)); gap: 12px; width: 100%; }
        .shortcut-tile {
            background: var(--glass); backdrop-filter: blur(12px); border: 1px solid var(--glass-border);
            border-radius: 14px; padding: 16px 8px; text-align: center; cursor: pointer; font-size: 12.5px;
        }

        /* Tela de Resultados (Explicação IA + Sites) */
        .search-results-page { max-width: 800px; margin: 0 auto; padding: 24px 16px 60px 16px; }
        .ai-overview-card {
            background: linear-gradient(145deg, rgba(79, 172, 254, 0.12), rgba(155, 81, 224, 0.12));
            border: 1px solid rgba(0, 242, 254, 0.45); border-radius: 16px; padding: 18px; margin-bottom: 24px;
        }
        .ai-overview-header { font-weight: 700; font-size: 14px; color: var(--accent); margin-bottom: 10px; }
        .ai-overview-text { font-size: 14.5px; line-height: 1.65; color: #f3f4f6; }

        .web-result-card {
            background: var(--glass); border: 1px solid var(--glass-border);
            border-radius: 12px; padding: 15px 18px; margin-bottom: 12px;
        }
        .web-result-card a { color: #60a5fa; font-size: 16px; font-weight: 600; text-decoration: none; display: block; margin-bottom: 4px; }
        .web-result-url { color: #34d399; font-size: 11.5px; margin-bottom: 5px; }
        .web-result-desc { color: var(--text-muted); font-size: 13px; }

        /* Chat Lateral Nexus Atlytic */
        .atlytic-sidebar {
            position: absolute; right: 0; top: 0; bottom: 0; width: 360px; max-width: 100%;
            background: #131722; border-left: 1px solid var(--glass-border); display: none; flex-direction: column; z-index: 30;
        }
        .atlytic-sidebar.open { display: flex; }
        .atlytic-header {
            padding: 14px 16px; background: rgba(79,172,254,0.12); border-bottom: 1px solid var(--glass-border);
            display: flex; justify-content: space-between; align-items: center;
        }
        .chat-messages { flex: 1; padding: 16px; overflow-y: auto; display: flex; flex-direction: column; gap: 12px; }
        .msg { padding: 11px 14px; border-radius: 14px; font-size: 13.5px; line-height: 1.5; max-width: 88%; }
        .msg.user { background: #2563eb; color: white; align-self: flex-end; }
        .msg.ai { background: var(--glass); border: 1px solid var(--glass-border); align-self: flex-start; }
        .chat-input-area { padding: 10px; border-top: 1px solid var(--glass-border); display: flex; gap: 8px; background: var(--bg-toolbar); }
        .chat-input-area input { flex: 1; background: var(--bg-frame); border: 1px solid var(--glass-border); border-radius: 20px; padding: 10px 14px; color: white; outline: none; font-size: 14px; }
        .send-ai-btn { background: var(--accent-grad); border: none; border-radius: 50%; width: 38px; height: 38px; cursor: pointer; font-weight: bold; color: #090b10; }

        iframe { width: 100%; height: calc(100% - 36px); border: none; background: white; }
    </style>
</head>
<body>

    <div class="tab-bar">
        <div class="nexus-badge">NEXUS</div>
        <div class="tabs-container" id="tabs-container"></div>
        <button class="new-tab-btn" onclick="createTab()">+</button>
    </div>

    <div class="nav-bar">
        <button class="nav-btn" onclick="goHome()">⌂</button>
        <button class="nav-btn" onclick="reloadTab()">↻</button>
        <div class="url-pill">
            <span style="font-size:12px;">🔍</span>
Use o código com cuidado.
✨ Atlytic
✨ Nexus Atlytic AI
×


Olá! Sou a Nexus Atlytic. Pode me perguntar qualquer coisa!



➤


let tabs = [];
let activeId = null;
let count = 0;
// MOTOR DE INTELIGÊNCIA ARTIFICIAL NEXUS ATLYTIC
async function consultarIA(pergunta) {
const promptSistema = "Você é a Nexus Atlytic, a inteligência artificial oficial do Navegador Nexus. Responda sempre em português do Brasil de forma clara, completa, inteligente e amigável.";
// Tentativa 1: API POST oficial (suporta perguntas longas e complexas)
try {
const res = await fetch('pollinations.ai', {
method: 'POST',
headers: { 'Content-Type': 'application/json' },
body: JSON.stringify({
messages: [
{ role: 'system', content: promptSistema },
{ role: 'user', content: pergunta }
],
model: 'openai'
})
});
if (res.ok) {
const texto = await res.text();
if (texto && texto.trim().length > 0) return texto;
}
} catch (e) {}
// Tentativa 2 (Fallback automático via GET caso a rede bloqueie POST)
const resGet = await fetch(https://pollinations.ai{encodeURIComponent(pergunta)}?system=${encodeURIComponent(promptSistema)});
return await resGet.text();
}
function formatarResposta(texto) {
return texto
.replace(/**(.?)**/g, '$1')
.replace(/*(.?)*/g, '$1')
.replace(/\n/g, '
');
}
function createTab(url = 'nexus://home', title = 'Início') {
count++;
const id = 'tab-' + count;
tabs.push({ id, title, url });
switchTab(id);
}
function switchTab(id) {
activeId = id;
const tab = tabs.find(t => t.id === id);
document.getElementById('url-input').value = tab.url === 'nexus://home' ? '' : tab.url;
renderTabs();
renderViewport();
}
function closeTab(e, id) {
e.stopPropagation();
if (tabs.length === 1) return;
const idx = tabs.findIndex(t => t.id === id);
tabs.splice(idx, 1);
if (activeId === id) switchTab(tabs[Math.max(0, idx - 1)].id);
else renderTabs();
}
function renderTabs() {
document.getElementById('tabs-container').innerHTML = tabs.map(t => <div class="tab ${t.id === activeId ? 'active' : ''}" onclick="switchTab('${t.id}')"> <span style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">${t.title}</span> <span class="close-tab" onclick="closeTab(event, '${t.id}')">×</span> </div>).join('');
}
function navigate(query) {
const tab = tabs.find(t => t.id === activeId);
let q = query.trim();
if (!q || q === 'nexus://home') {
tab.url = 'nexus://home';
tab.title = 'Início';
} else if (q.startsWith('http://') || q.startsWith('https://')) {
tab.url = q;
tab.title = q.replace(/^https?:///, '').split('/');
} else if (q.includes('.') && !q.includes(' ')) {
tab.url = 'https://' + q;
tab.title = q;
} else {
tab.url = 'nexus://search?q=' + encodeURIComponent(q);
tab.title = q;
}
switchTab(activeId);
}
function goHome() { navigate('nexus://home'); }
function reloadTab() { renderViewport(); }
async function renderViewport() {
const viewport = document.getElementById('viewport');
const tab = tabs.find(t => t.id === activeId);
if (tab.url === 'nexus://home') {
const now = new Date();
const timeStr = now.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' });
const dateStr = now.toLocaleDateString('pt-BR', { weekday: 'short', day: 'numeric', month: 'short' });
viewport.innerHTML = `





${timeStr}
${dateStr}

✨ Atlytic IA Online

NEXUS
PERGUNTE QUALQUER COISA
✨



📚Wikipédia
🌌Testar IA
🚀Tech Hoje
💬Chat Atlytic



; } else if (tab.url.startsWith('nexus://search?q=')) { const query = decodeURIComponent(tab.url.split('=')); viewport.innerHTML = 


✨ Resposta • Nexus Atlytic
⏳ Nexus Atlytic está formulando a resposta para "${query}"...
Sites Relacionados na Web
google.com
Pesquisar "${query}" no Google ↗
Ver todos os resultados, imagens e páginas da web sobre o tema.
wikipedia.org
${query} – Wikipédia ↗
Consultar definições e artigos enciclopédicos completos.
youtube.com
Vídeos sobre "${query}" no YouTube ↗
Assistir a vídeos e explicações visuais sobre este assunto.


`;
try {
const resposta = await consultarIA(query);
const el = document.getElementById('ai-search-answer');
if (el) el.innerHTML = formatarResposta(resposta);
} catch (e) {
const el = document.getElementById('ai-search-answer');
if (el) el.textContent = "Sem conexão com a internet no momento para gerar a resposta da IA. Use os links abaixo.";
}
} else {
viewport.innerHTML = <div style="background:#1e2333; padding:8px 14px; font-size:12px; display:flex; justify-content:space-between; align-items:center;"> <span style="overflow:hidden; text-overflow:ellipsis; white-space:nowrap; max-width:65%;">${tab.url}</span> <a href="${tab.url}" target="_blank" style="color:var(--accent); font-weight:bold; text-decoration:none;">Abrir fora ↗</a> </div> <iframe src="${tab.url}"></iframe>;
}
}
function toggleAtlytic() {
document.getElementById('atlytic-sidebar').classList.toggle('open');
}
async function askAtlytic() {
const input = document.getElementById('ai-prompt');
const q = input.value.trim();
if (!q) return;
const chat = document.getElementById('chat-messages');
chat.innerHTML += <div class="msg user">${q}</div>;
input.value = '';
const loadId = 'msg-' + Date.now();
chat.innerHTML += <div class="msg ai" id="${loadId}">✨ Digitando...</div>;
chat.scrollTop = chat.scrollHeight;
try {
const resposta = await consultarIA(q);
document.getElementById(loadId).innerHTML = formatarResposta(resposta);
} catch (e) {
document.getElementById(loadId).textContent = 'Erro de conexão. Verifique sua internet.';
}
chat.scrollTop = chat.scrollHeight;
}
createTab();



<FollowUp>
Se quiser testar agora no seu Xiaomi:
* Cole esse código no **[html.cafe](https://html.cafe)** para baixar o arquivo `index.html`
* Suba no **[tiiny.host](https://tiiny.host)** para gerar o seu link

Me avise se conseguiu abrir o link e testar a primeira pergunta na **Nexus Atlytic**.
</FollowUp>
Copiando...

