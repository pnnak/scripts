# SCRIPT



##  LeiaPR Bypass


```// ==UserScript==
// @name         Leia-me Auto Gemini Cheat (Automático)
// @namespace    http://tampermonkey.net/
// @version      4.7
// @description  Responde perguntas e avança automaticamente no Leia-me/Odilo com Gemini AI 😎
// @author       MZ
// @match        *://*odilo*/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    const API_KEY = 'AIzaSyDzHvHcoBgfeNJf0iwM2AfjQM3mQ9sW-W8'; // Sua API Key aqui
    let autoMode = false;
    let processando = false;

    if (window.top !== window.self) return;

    // Estilo visual da UI
    const style = document.createElement("style");
    style.textContent = `
        .gemini-box {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: #1e1e2f;
            color: #fff;
            font-family: 'Segoe UI', sans-serif;
            padding: 15px 20px;
            border-radius: 16px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
            z-index: 99999;
            display: flex;
            flex-direction: column;
            gap: 8px;
            width: 240px;
        }
        .gemini-box h1 { font-size: 16px; margin: 0; font-weight: 600; text-align: center; }
        .gemini-box h2 { font-size: 13px; margin: 0; font-weight: 400; text-align: center; opacity: 0.8; }
        .gemini-box button {
            background: #3b82f6;
            color: white;
            border: none;
            padding: 10px;
            font-size: 14px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }
        .gemini-box .auto-on { background: #10b981 !important; }
        .gemini-box .auto-off { background: #ef4444 !important; }
    `;
    document.head.appendChild(style);

    const ui = document.createElement("div");
    ui.className = "gemini-box";
    ui.innerHTML = `
        <h1>📘 Leia-me Cheat</h1>
        <h2>🦇 by @mzzvxm</h2>
        <button id="toggleAuto" class="auto-off">⚙️ Auto: OFF</button>
        <div id="status" style="font-size:12px; color:#ccc; text-align:center; margin-top:4px;">Aguardando</div>
    `;
    document.body.appendChild(ui);

    const btnToggle = document.getElementById("toggleAuto");
    const statusDiv = document.getElementById("status");

    btnToggle.onclick = function () {
        autoMode = !autoMode;
        this.textContent = `⚙️ Auto: ${autoMode ? 'ON' : 'OFF'}`;
        this.classList.toggle("auto-on", autoMode);
        this.classList.toggle("auto-off", !autoMode);
        if (autoMode) {
            statusDiv.textContent = "Modo automático ativado";
            iniciarLeituraAutomatica();
        } else {
            statusDiv.textContent = "Modo automático desativado";
        }
    };

    function temPerguntaAtiva() {
        return !!document.querySelector('.question-quiz-text.ng-binding');
    }

    function selecionarResposta(letra) {
        const mapa = { A: 0, B: 1, C: 2, D: 3, E: 4 };
        const index = mapa[letra.toUpperCase()];
        if (index === undefined) return false;

        const radios = document.querySelectorAll('md-radio-button.choice-radio-button');
        const opcoesTexto = document.querySelectorAll('.choice-student.choice-new-styles__answer');

        if (radios[index]) radios[index].click();
        if (opcoesTexto[index]) opcoesTexto[index].click();

        return !!(radios[index] || opcoesTexto[index]);
    }

    function clicarBotaoQuiz() {
        return new Promise((resolve, reject) => {
            const tentarClique = () => {
                const container = document.querySelector('md-dialog-actions.quiz-dialog-buttons');
                if (!container) return false;

                const btnTerminar = container.querySelector('button[ng-click="finish()"]');
                if (btnTerminar && btnTerminar.offsetParent !== null && !btnTerminar.disabled) {
                    console.log("✅ Botão Terminar encontrado. Clicando...");
                    btnTerminar.click();

                    const obsEnviar = new MutationObserver((mutations, obs) => {
                        const btnEnviar = container.querySelector('button[ng-click="sendAnswer()"]');
                        if (btnEnviar && btnEnviar.offsetParent !== null && !btnEnviar.disabled) {
                            console.log("✅ Botão Enviar apareceu. Clicando...");
                            btnEnviar.click();
                            obs.disconnect();

                            setTimeout(() => {
                                esperarEClicarFechar();
                                resolve(true);
                            }, 500);
                        }
                    });

                    obsEnviar.observe(container, { childList: true, subtree: true });

                    setTimeout(() => {
                        obsEnviar.disconnect();
                        reject("❌ Botão Enviar não apareceu a tempo.");
                    }, 5000);

                    return true;
                }

                const btnProximo = container.querySelector('button[ng-click="next()"]');
                if (btnProximo && btnProximo.offsetParent !== null && !btnProximo.disabled) {
                    console.log("✅ Botão Próximo encontrado. Clicando...");
                    btnProximo.click();
                    resolve(true);
                    return true;
                }

                return false;
            };

            if (tentarClique()) return;

            const obsInit = new MutationObserver((mutations, obs) => {
                const container = document.querySelector('md-dialog-actions.quiz-dialog-buttons');
                if (container) {
                    obs.disconnect();
                    clicarBotaoQuiz().then(resolve).catch(reject);
                }
            });
            obsInit.observe(document.body, { childList: true, subtree: true });

            setTimeout(() => {
                obsInit.disconnect();
                reject("❌ Container de botões não apareceu.");
            }, 5000);
        });
    }

    async function avancarPergunta() {
        try {
            return await clicarBotaoQuiz();
        } catch (e) {
            console.warn("Erro ao tentar clicar no botão do Quiz:", e);
            return false;
        }
    }

    // Dispara a tecla 'Seta para a Direita' como fallback
    function simularSetaDireita() {
        const target = document.body || window;
        const keyEvent = new KeyboardEvent('keydown', {
            key: 'ArrowRight',
            code: 'ArrowRight',
            keyCode: 39,
            which: 39,
            bubbles: true,
            cancelable: true
        });
        target.dispatchEvent(keyEvent);
    }

    async function avancarPaginaLivro() {
        const seletoresBotaoNext = [
            'button#right-page-btn',
            '#right-page-btn',
            'button[aria-label*="Próxim"]',
            'button[aria-label*="Siguiente"]',
            'button[aria-label*="Next"]',
            'button[aria-label*="Página seguinte"]',
            '.icon-chevron-right',
            '.arrow-right',
            '.paged-right',
            '#next-page-btn',
            '.next-page',
            'button.page-right'
        ];

        let btn = null;

        // Search in standard document
        for (const sel of seletoresBotaoNext) {
            btn = document.querySelector(sel);
            if (btn && btn.offsetParent !== null) break;
        }

        // Search inside iframes if not found in root document
        if (!btn) {
            const iframes = document.querySelectorAll('iframe');
            for (const iframe of iframes) {
                try {
                    const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
                    for (const sel of seletoresBotaoNext) {
                        btn = iframeDoc.querySelector(sel);
                        if (btn && btn.offsetParent !== null) break;
                    }
                    if (btn) break;
                } catch (e) {
                    // Ignora se for restrição CORS de iframe externo
                }
            }
        }

        if (btn && !btn.disabled) {
            console.log("✅ Botão de próxima página encontrado. Clicando...");
            btn.click();
            btn.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true }));
            statusDiv.textContent = "Avançou página (Botão)";
            return true;
        } else {
            console.log("⚠️ Botão físico não detectado. Enviando tecla Seta Direita (ArrowRight)...");
            simularSetaDireita();
            statusDiv.textContent = "Avançou página (Teclado ➔)";
            return true;
        }
    }

    async function chamarGemini(pergunta, alternativas) {
        const prompt = `
Responda a seguinte pergunta do tipo múltipla escolha. Retorne apenas a letra correta.

Pergunta: ${pergunta}

Alternativas:
${alternativas.map((alt, i) => `${String.fromCharCode(65 + i)}) ${alt}`).join("\n")}
        `.trim();

        try {
            const response = await fetch(
                `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`,
                {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }]
                    })
                }
            );

            const data = await response.json();
            const texto = data?.candidates?.[0]?.content?.parts?.[0]?.text || "";
            const letra = texto.match(/[A-E]/i)?.[0]?.toUpperCase();

            console.log("💬 Resposta Gemini (raw):", texto);
            console.log("✅ Letra extraída:", letra);

            return letra || null;

        } catch (error) {
            console.error("Erro na chamada Gemini:", error);
            statusDiv.textContent = "Erro na API Gemini";
            return null;
        }
    }

    async function processarPergunta() {
        if (processando) return;
        processando = true;

        try {
            const perguntaEl = document.querySelector('.question-quiz-text.ng-binding');
            if (!perguntaEl) {
                statusDiv.textContent = "Nenhuma pergunta ativa";
                processando = false;
                return;
            }

            const pergunta = perguntaEl.innerText.trim();
            const opcoes = [...document.querySelectorAll('.choice-student.choice-new-styles__answer')]
                .map(el => el.innerText.trim())
                .filter(text => text.length > 0);

            if (!pergunta || opcoes.length === 0) {
                statusDiv.textContent = "Pergunta/opções não encontradas";
                processando = false;
                return;
            }

            statusDiv.textContent = "Respondendo pergunta com IA...";
            console.log("📘 Pergunta:", pergunta);

            const letra = await chamarGemini(pergunta, opcoes);

            if (letra) {
                statusDiv.textContent = `Gemini respondeu: ${letra}`;

                if (selecionarResposta(letra)) {
                    await new Promise(r => setTimeout(r, 1500));
                    await avancarPergunta();
                    await new Promise(r => setTimeout(r, 3000));
                } else {
                    statusDiv.textContent = "Erro ao selecionar opção";
                }
            } else {
                statusDiv.textContent = "Sem resposta do Gemini";
            }
        } finally {
            processando = false;
        }
    }

    async function iniciarLeituraAutomatica() {
        statusDiv.textContent = "Iniciando modo automático...";
        while (autoMode) {
            if (temPerguntaAtiva()) {
                await processarPergunta();
            } else {
                await avancarPaginaLivro();
                // Ajuste o tempo de espera por página aqui (atualmente entre 35s e 45s)
                const tempoLeitura = Math.floor(Math.random() * (45000 - 35000 + 1)) + 35000;
                
                // Atualiza contagem regressiva no status
                let restante = Math.floor(tempoLeitura / 1000);
                while (restante > 0 && autoMode && !temPerguntaAtiva()) {
                    statusDiv.textContent = `Próxima página em ${restante}s...`;
                    await new Promise(r => setTimeout(r, 1000));
                    restante--;
                }
            }
            await new Promise(r => setTimeout(r, 1000));
        }
        statusDiv.textContent = "Modo automático parado";
    }

    function esperarEClicarFechar() {
        const tentarCliqueFechar = () => {
            const btnFechar = document.querySelector('md-toolbar button.md-icon-button[ng-click="close()"]');
            if (btnFechar && btnFechar.offsetParent !== null && !btnFechar.disabled) {
                console.log("✅ Botão Fechar encontrado. Clicando...");
                btnFechar.click();
                return true;
            }
            return false;
        };

        if (tentarCliqueFechar()) return;

        const obsFechar = new MutationObserver(() => {
            if (tentarCliqueFechar()) {
                obsFechar.disconnect();
            }
        });

        obsFechar.observe(document.body, { childList: true, subtree: true });
        setTimeout(() => obsFechar.disconnect(), 5000);
    }

})()// ==UserScript==
// @name         Leia-me Auto Gemini Cheat (Automático)
// @namespace    http://tampermonkey.net/
// @version      4.7
// @description  Responde perguntas e avança automaticamente no Leia-me/Odilo com Gemini AI 😎
// @author       MZ
// @match        *://*odilo*/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    const API_KEY = 'AIzaSyDzHvHcoBgfeNJf0iwM2AfjQM3mQ9sW-W8'; // Sua API Key aqui
    let autoMode = false;
    let processando = false;

    if (window.top !== window.self) return;

    // Estilo visual da UI
    const style = document.createElement("style");
    style.textContent = `
        .gemini-box {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: #1e1e2f;
            color: #fff;
            font-family: 'Segoe UI', sans-serif;
            padding: 15px 20px;
            border-radius: 16px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
            z-index: 99999;
            display: flex;
            flex-direction: column;
            gap: 8px;
            width: 240px;
        }
        .gemini-box h1 { font-size: 16px; margin: 0; font-weight: 600; text-align: center; }
        .gemini-box h2 { font-size: 13px; margin: 0; font-weight: 400; text-align: center; opacity: 0.8; }
        .gemini-box button {
            background: #3b82f6;
            color: white;
            border: none;
            padding: 10px;
            font-size: 14px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }
        .gemini-box .auto-on { background: #10b981 !important; }
        .gemini-box .auto-off { background: #ef4444 !important; }
    `;
    document.head.appendChild(style);

    const ui = document.createElement("div");
    ui.className = "gemini-box";
    ui.innerHTML = `
        <h1>📘 Leia-me Cheat</h1>
        <h2>🦇 by @mzzvxm</h2>
        <button id="toggleAuto" class="auto-off">⚙️ Auto: OFF</button>
        <div id="status" style="font-size:12px; color:#ccc; text-align:center; margin-top:4px;">Aguardando</div>
    `;
    document.body.appendChild(ui);

    const btnToggle = document.getElementById("toggleAuto");
    const statusDiv = document.getElementById("status");

    btnToggle.onclick = function () {
        autoMode = !autoMode;
        this.textContent = `⚙️ Auto: ${autoMode ? 'ON' : 'OFF'}`;
        this.classList.toggle("auto-on", autoMode);
        this.classList.toggle("auto-off", !autoMode);
        if (autoMode) {
            statusDiv.textContent = "Modo automático ativado";
            iniciarLeituraAutomatica();
        } else {
            statusDiv.textContent = "Modo automático desativado";
        }
    };

    function temPerguntaAtiva() {
        return !!document.querySelector('.question-quiz-text.ng-binding');
    }

    function selecionarResposta(letra) {
        const mapa = { A: 0, B: 1, C: 2, D: 3, E: 4 };
        const index = mapa[letra.toUpperCase()];
        if (index === undefined) return false;

        const radios = document.querySelectorAll('md-radio-button.choice-radio-button');
        const opcoesTexto = document.querySelectorAll('.choice-student.choice-new-styles__answer');

        if (radios[index]) radios[index].click();
        if (opcoesTexto[index]) opcoesTexto[index].click();

        return !!(radios[index] || opcoesTexto[index]);
    }

    function clicarBotaoQuiz() {
        return new Promise((resolve, reject) => {
            const tentarClique = () => {
                const container = document.querySelector('md-dialog-actions.quiz-dialog-buttons');
                if (!container) return false;

                const btnTerminar = container.querySelector('button[ng-click="finish()"]');
                if (btnTerminar && btnTerminar.offsetParent !== null && !btnTerminar.disabled) {
                    console.log("✅ Botão Terminar encontrado. Clicando...");
                    btnTerminar.click();

                    const obsEnviar = new MutationObserver((mutations, obs) => {
                        const btnEnviar = container.querySelector('button[ng-click="sendAnswer()"]');
                        if (btnEnviar && btnEnviar.offsetParent !== null && !btnEnviar.disabled) {
                            console.log("✅ Botão Enviar apareceu. Clicando...");
                            btnEnviar.click();
                            obs.disconnect();

                            setTimeout(() => {
                                esperarEClicarFechar();
                                resolve(true);
                            }, 500);
                        }
                    });

                    obsEnviar.observe(container, { childList: true, subtree: true });

                    setTimeout(() => {
                        obsEnviar.disconnect();
                        reject("❌ Botão Enviar não apareceu a tempo.");
                    }, 5000);

                    return true;
                }

                const btnProximo = container.querySelector('button[ng-click="next()"]');
                if (btnProximo && btnProximo.offsetParent !== null && !btnProximo.disabled) {
                    console.log("✅ Botão Próximo encontrado. Clicando...");
                    btnProximo.click();
                    resolve(true);
                    return true;
                }

                return false;
            };

            if (tentarClique()) return;

            const obsInit = new MutationObserver((mutations, obs) => {
                const container = document.querySelector('md-dialog-actions.quiz-dialog-buttons');
                if (container) {
                    obs.disconnect();
                    clicarBotaoQuiz().then(resolve).catch(reject);
                }
            });
            obsInit.observe(document.body, { childList: true, subtree: true });

            setTimeout(() => {
                obsInit.disconnect();
                reject("❌ Container de botões não apareceu.");
            }, 5000);
        });
    }

    async function avancarPergunta() {
        try {
            return await clicarBotaoQuiz();
        } catch (e) {
            console.warn("Erro ao tentar clicar no botão do Quiz:", e);
            return false;
        }
    }

    // Dispara a tecla 'Seta para a Direita' como fallback
    function simularSetaDireita() {
        const target = document.body || window;
        const keyEvent = new KeyboardEvent('keydown', {
            key: 'ArrowRight',
            code: 'ArrowRight',
            keyCode: 39,
            which: 39,
            bubbles: true,
            cancelable: true
        });
        target.dispatchEvent(keyEvent);
    }

    async function avancarPaginaLivro() {
        const seletoresBotaoNext = [
            'button#right-page-btn',
            '#right-page-btn',
            'button[aria-label*="Próxim"]',
            'button[aria-label*="Siguiente"]',
            'button[aria-label*="Next"]',
            'button[aria-label*="Página seguinte"]',
            '.icon-chevron-right',
            '.arrow-right',
            '.paged-right',
            '#next-page-btn',
            '.next-page',
            'button.page-right'
        ];

        let btn = null;

        // Search in standard document
        for (const sel of seletoresBotaoNext) {
            btn = document.querySelector(sel);
            if (btn && btn.offsetParent !== null) break;
        }

        // Search inside iframes if not found in root document
        if (!btn) {
            const iframes = document.querySelectorAll('iframe');
            for (const iframe of iframes) {
                try {
                    const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
                    for (const sel of seletoresBotaoNext) {
                        btn = iframeDoc.querySelector(sel);
                        if (btn && btn.offsetParent !== null) break;
                    }
                    if (btn) break;
                } catch (e) {
                    // Ignora se for restrição CORS de iframe externo
                }
            }
        }

        if (btn && !btn.disabled) {
            console.log("✅ Botão de próxima página encontrado. Clicando...");
            btn.click();
            btn.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true }));
            statusDiv.textContent = "Avançou página (Botão)";
            return true;
        } else {
            console.log("⚠️ Botão físico não detectado. Enviando tecla Seta Direita (ArrowRight)...");
            simularSetaDireita();
            statusDiv.textContent = "Avançou página (Teclado ➔)";
            return true;
        }
    }

    async function chamarGemini(pergunta, alternativas) {
        const prompt = `
Responda a seguinte pergunta do tipo múltipla escolha. Retorne apenas a letra correta.

Pergunta: ${pergunta}

Alternativas:
${alternativas.map((alt, i) => `${String.fromCharCode(65 + i)}) ${alt}`).join("\n")}
        `.trim();

        try {
            const response = await fetch(
                `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`,
                {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }]
                    })
                }
            );

            const data = await response.json();
            const texto = data?.candidates?.[0]?.content?.parts?.[0]?.text || "";
            const letra = texto.match(/[A-E]/i)?.[0]?.toUpperCase();

            console.log("💬 Resposta Gemini (raw):", texto);
            console.log("✅ Letra extraída:", letra);

            return letra || null;

        } catch (error) {
            console.error("Erro na chamada Gemini:", error);
            statusDiv.textContent = "Erro na API Gemini";
            return null;
        }
    }

    async function processarPergunta() {
        if (processando) return;
        processando = true;

        try {
            const perguntaEl = document.querySelector('.question-quiz-text.ng-binding');
            if (!perguntaEl) {
                statusDiv.textContent = "Nenhuma pergunta ativa";
                processando = false;
                return;
            }

            const pergunta = perguntaEl.innerText.trim();
            const opcoes = [...document.querySelectorAll('.choice-student.choice-new-styles__answer')]
                .map(el => el.innerText.trim())
                .filter(text => text.length > 0);

            if (!pergunta || opcoes.length === 0) {
                statusDiv.textContent = "Pergunta/opções não encontradas";
                processando = false;
                return;
            }

            statusDiv.textContent = "Respondendo pergunta com IA...";
            console.log("📘 Pergunta:", pergunta);

            const letra = await chamarGemini(pergunta, opcoes);

            if (letra) {
                statusDiv.textContent = `Gemini respondeu: ${letra}`;

                if (selecionarResposta(letra)) {
                    await new Promise(r => setTimeout(r, 1500));
                    await avancarPergunta();
                    await new Promise(r => setTimeout(r, 3000));
                } else {
                    statusDiv.textContent = "Erro ao selecionar opção";
                }
            } else {
                statusDiv.textContent = "Sem resposta do Gemini";
            }
        } finally {
            processando = false;
        }
    }

    async function iniciarLeituraAutomatica() {
        statusDiv.textContent = "Iniciando modo automático...";
        while (autoMode) {
            if (temPerguntaAtiva()) {
                await processarPergunta();
            } else {
                await avancarPaginaLivro();
                // Ajuste o tempo de espera por página aqui (atualmente entre 35s e 45s)
                const tempoLeitura = Math.floor(Math.random() * (45000 - 35000 + 1)) + 35000;
                
                // Atualiza contagem regressiva no status
                let restante = Math.floor(tempoLeitura / 1000);
                while (restante > 0 && autoMode && !temPerguntaAtiva()) {
                    statusDiv.textContent = `Próxima página em ${restante}s...`;
                    await new Promise(r => setTimeout(r, 1000));
                    restante--;
                }
            }
            await new Promise(r => setTimeout(r, 1000));
        }
        statusDiv.textContent = "Modo automático parado";
    }

    function esperarEClicarFechar() {
        const tentarCliqueFechar = () => {
            const btnFechar = document.querySelector('md-toolbar button.md-icon-button[ng-click="close()"]');
            if (btnFechar && btnFechar.offsetParent !== null && !btnFechar.disabled) {
                console.log("✅ Botão Fechar encontrado. Clicando...");
                btnFechar.click();
                return true;
            }
            return false;
        };

        if (tentarCliqueFechar()) return;

        const obsFechar = new MutationObserver(() => {
            if (tentarCliqueFechar()) {
                obsFechar.disconnect();
            }
        });

        obsFechar.observe(document.body, { childList: true, subtree: true });
        setTimeout(() => obsFechar.disconnect(), 5000);
    }

})();// ==UserScript==
// @name         Leia-me Auto Gemini Cheat (Automático)
// @namespace    http://tampermonkey.net/
// @version      4.7
// @description  Responde perguntas e avança automaticamente no Leia-me/Odilo com Gemini AI 😎
// @author       MZ
// @match        *://*odilo*/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==

(function () {
    'use strict';

    const API_KEY = 'AIzaSyDzHvHcoBgfeNJf0iwM2AfjQM3mQ9sW-W8'; // Sua API Key aqui
    let autoMode = false;
    let processando = false;

    if (window.top !== window.self) return;

    // Estilo visual da UI
    const style = document.createElement("style");
    style.textContent = `
        .gemini-box {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: #1e1e2f;
            color: #fff;
            font-family: 'Segoe UI', sans-serif;
            padding: 15px 20px;
            border-radius: 16px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
            z-index: 99999;
            display: flex;
            flex-direction: column;
            gap: 8px;
            width: 240px;
        }
        .gemini-box h1 { font-size: 16px; margin: 0; font-weight: 600; text-align: center; }
        .gemini-box h2 { font-size: 13px; margin: 0; font-weight: 400; text-align: center; opacity: 0.8; }
        .gemini-box button {
            background: #3b82f6;
            color: white;
            border: none;
            padding: 10px;
            font-size: 14px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }
        .gemini-box .auto-on { background: #10b981 !important; }
        .gemini-box .auto-off { background: #ef4444 !important; }
    `;
    document.head.appendChild(style);

    const ui = document.createElement("div");
    ui.className = "gemini-box";
    ui.innerHTML = `
        <h1>📘 Leia-me Cheat</h1>
        <h2>🦇 by @mzzvxm</h2>
        <button id="toggleAuto" class="auto-off">⚙️ Auto: OFF</button>
        <div id="status" style="font-size:12px; color:#ccc; text-align:center; margin-top:4px;">Aguardando</div>
    `;
    document.body.appendChild(ui);

    const btnToggle = document.getElementById("toggleAuto");
    const statusDiv = document.getElementById("status");

    btnToggle.onclick = function () {
        autoMode = !autoMode;
        this.textContent = `⚙️ Auto: ${autoMode ? 'ON' : 'OFF'}`;
        this.classList.toggle("auto-on", autoMode);
        this.classList.toggle("auto-off", !autoMode);
        if (autoMode) {
            statusDiv.textContent = "Modo automático ativado";
            iniciarLeituraAutomatica();
        } else {
            statusDiv.textContent = "Modo automático desativado";
        }
    };

    function temPerguntaAtiva() {
        return !!document.querySelector('.question-quiz-text.ng-binding');
    }

    function selecionarResposta(letra) {
        const mapa = { A: 0, B: 1, C: 2, D: 3, E: 4 };
        const index = mapa[letra.toUpperCase()];
        if (index === undefined) return false;

        const radios = document.querySelectorAll('md-radio-button.choice-radio-button');
        const opcoesTexto = document.querySelectorAll('.choice-student.choice-new-styles__answer');

        if (radios[index]) radios[index].click();
        if (opcoesTexto[index]) opcoesTexto[index].click();

        return !!(radios[index] || opcoesTexto[index]);
    }

    function clicarBotaoQuiz() {
        return new Promise((resolve, reject) => {
            const tentarClique = () => {
                const container = document.querySelector('md-dialog-actions.quiz-dialog-buttons');
                if (!container) return false;

                const btnTerminar = container.querySelector('button[ng-click="finish()"]');
                if (btnTerminar && btnTerminar.offsetParent !== null && !btnTerminar.disabled) {
                    console.log("✅ Botão Terminar encontrado. Clicando...");
                    btnTerminar.click();

                    const obsEnviar = new MutationObserver((mutations, obs) => {
                        const btnEnviar = container.querySelector('button[ng-click="sendAnswer()"]');
                        if (btnEnviar && btnEnviar.offsetParent !== null && !btnEnviar.disabled) {
                            console.log("✅ Botão Enviar apareceu. Clicando...");
                            btnEnviar.click();
                            obs.disconnect();

                            setTimeout(() => {
                                esperarEClicarFechar();
                                resolve(true);
                            }, 500);
                        }
                    });

                    obsEnviar.observe(container, { childList: true, subtree: true });

                    setTimeout(() => {
                        obsEnviar.disconnect();
                        reject("❌ Botão Enviar não apareceu a tempo.");
                    }, 5000);

                    return true;
                }

                const btnProximo = container.querySelector('button[ng-click="next()"]');
                if (btnProximo && btnProximo.offsetParent !== null && !btnProximo.disabled) {
                    console.log("✅ Botão Próximo encontrado. Clicando...");
                    btnProximo.click();
                    resolve(true);
                    return true;
                }

                return false;
            };

            if (tentarClique()) return;

            const obsInit = new MutationObserver((mutations, obs) => {
                const container = document.querySelector('md-dialog-actions.quiz-dialog-buttons');
                if (container) {
                    obs.disconnect();
                    clicarBotaoQuiz().then(resolve).catch(reject);
                }
            });
            obsInit.observe(document.body, { childList: true, subtree: true });

            setTimeout(() => {
                obsInit.disconnect();
                reject("❌ Container de botões não apareceu.");
            }, 5000);
        });
    }

    async function avancarPergunta() {
        try {
            return await clicarBotaoQuiz();
        } catch (e) {
            console.warn("Erro ao tentar clicar no botão do Quiz:", e);
            return false;
        }
    }

    // Dispara a tecla 'Seta para a Direita' como fallback
    function simularSetaDireita() {
        const target = document.body || window;
        const keyEvent = new KeyboardEvent('keydown', {
            key: 'ArrowRight',
            code: 'ArrowRight',
            keyCode: 39,
            which: 39,
            bubbles: true,
            cancelable: true
        });
        target.dispatchEvent(keyEvent);
    }

    async function avancarPaginaLivro() {
        const seletoresBotaoNext = [
            'button#right-page-btn',
            '#right-page-btn',
            'button[aria-label*="Próxim"]',
            'button[aria-label*="Siguiente"]',
            'button[aria-label*="Next"]',
            'button[aria-label*="Página seguinte"]',
            '.icon-chevron-right',
            '.arrow-right',
            '.paged-right',
            '#next-page-btn',
            '.next-page',
            'button.page-right'
        ];

        let btn = null;

        // Search in standard document
        for (const sel of seletoresBotaoNext) {
            btn = document.querySelector(sel);
            if (btn && btn.offsetParent !== null) break;
        }

        // Search inside iframes if not found in root document
        if (!btn) {
            const iframes = document.querySelectorAll('iframe');
            for (const iframe of iframes) {
                try {
                    const iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
                    for (const sel of seletoresBotaoNext) {
                        btn = iframeDoc.querySelector(sel);
                        if (btn && btn.offsetParent !== null) break;
                    }
                    if (btn) break;
                } catch (e) {
                    // Ignora se for restrição CORS de iframe externo
                }
            }
        }

        if (btn && !btn.disabled) {
            console.log("✅ Botão de próxima página encontrado. Clicando...");
            btn.click();
            btn.dispatchEvent(new MouseEvent('click', { bubbles: true, cancelable: true }));
            statusDiv.textContent = "Avançou página (Botão)";
            return true;
        } else {
            console.log("⚠️ Botão físico não detectado. Enviando tecla Seta Direita (ArrowRight)...");
            simularSetaDireita();
            statusDiv.textContent = "Avançou página (Teclado ➔)";
            return true;
        }
    }

    async function chamarGemini(pergunta, alternativas) {
        const prompt = `
Responda a seguinte pergunta do tipo múltipla escolha. Retorne apenas a letra correta.

Pergunta: ${pergunta}

Alternativas:
${alternativas.map((alt, i) => `${String.fromCharCode(65 + i)}) ${alt}`).join("\n")}
        `.trim();

        try {
            const response = await fetch(
                `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${API_KEY}`,
                {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: prompt }] }]
                    })
                }
            );

            const data = await response.json();
            const texto = data?.candidates?.[0]?.content?.parts?.[0]?.text || "";
            const letra = texto.match(/[A-E]/i)?.[0]?.toUpperCase();

            console.log("💬 Resposta Gemini (raw):", texto);
            console.log("✅ Letra extraída:", letra);

            return letra || null;

        } catch (error) {
            console.error("Erro na chamada Gemini:", error);
            statusDiv.textContent = "Erro na API Gemini";
            return null;
        }
    }

    async function processarPergunta() {
        if (processando) return;
        processando = true;

        try {
            const perguntaEl = document.querySelector('.question-quiz-text.ng-binding');
            if (!perguntaEl) {
                statusDiv.textContent = "Nenhuma pergunta ativa";
                processando = false;
                return;
            }

            const pergunta = perguntaEl.innerText.trim();
            const opcoes = [...document.querySelectorAll('.choice-student.choice-new-styles__answer')]
                .map(el => el.innerText.trim())
                .filter(text => text.length > 0);

            if (!pergunta || opcoes.length === 0) {
                statusDiv.textContent = "Pergunta/opções não encontradas";
                processando = false;
                return;
            }

            statusDiv.textContent = "Respondendo pergunta com IA...";
            console.log("📘 Pergunta:", pergunta);

            const letra = await chamarGemini(pergunta, opcoes);

            if (letra) {
                statusDiv.textContent = `Gemini respondeu: ${letra}`;

                if (selecionarResposta(letra)) {
                    await new Promise(r => setTimeout(r, 1500));
                    await avancarPergunta();
                    await new Promise(r => setTimeout(r, 3000));
                } else {
                    statusDiv.textContent = "Erro ao selecionar opção";
                }
            } else {
                statusDiv.textContent = "Sem resposta do Gemini";
            }
        } finally {
            processando = false;
        }
    }

    async function iniciarLeituraAutomatica() {
        statusDiv.textContent = "Iniciando modo automático...";
        while (autoMode) {
            if (temPerguntaAtiva()) {
                await processarPergunta();
            } else {
                await avancarPaginaLivro();
                // Ajuste o tempo de espera por página aqui (atualmente entre 35s e 45s)
                const tempoLeitura = Math.floor(Math.random() * (45000 - 35000 + 1)) + 35000;
                
                // Atualiza contagem regressiva no status
                let restante = Math.floor(tempoLeitura / 1000);
                while (restante > 0 && autoMode && !temPerguntaAtiva()) {
                    statusDiv.textContent = `Próxima página em ${restante}s...`;
                    await new Promise(r => setTimeout(r, 1000));
                    restante--;
                }
            }
            await new Promise(r => setTimeout(r, 1000));
        }
        statusDiv.textContent = "Modo automático parado";
    }

    function esperarEClicarFechar() {
        const tentarCliqueFechar = () => {
            const btnFechar = document.querySelector('md-toolbar button.md-icon-button[ng-click="close()"]');
            if (btnFechar && btnFechar.offsetParent !== null && !btnFechar.disabled) {
                console.log("✅ Botão Fechar encontrado. Clicando...");
                btnFechar.click();
                return true;
            }
            return false;
        };

        if (tentarCliqueFechar()) return;

        const obsFechar = new MutationObserver(() => {
            if (tentarCliqueFechar()) {
                obsFechar.disconnect();
            }
        });

        obsFechar.observe(document.body, { childList: true, subtree: true });
        setTimeout(() => obsFechar.disconnect(), 5000);
    }

})();
```

---

##  Redação Bypass


```javascript
javascript:(function(){
var s=document.createElement('script');
s.src='https://cdn.jsdelivr.net/gh/mzzvxm/RedacaoBypass@main/script.js?t='+Date.now();
s.crossOrigin='anonymous';
s.onload=function(){
console.log('RedacaoBypass loaded (no-cache)')
};
document.head.appendChild(s)
})();
```

---

##  Links Khan e Quizziz

* https://waygroundx.vercel.app/
* https://khan.cupiditys.lol/pt-br/

## 
