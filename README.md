
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <title>Robô Ponto Zero - Análise Profissional</title>
    <style>
        body { font-family: Arial, sans-serif; background: #121212; color: #fff; text-align: center; padding: 10px; margin: 0; overflow-x: hidden; }
        h2 { font-size: 18px; margin-bottom: 5px; color: #ffeb3b; }
        canvas { background: #1e1e1e; border: 1px solid #333; margin-top: 5px; width: 100%; max-width: 600px; height: 250px; }
        .info-box { background: #1a1a1a; border: 1px solid #333; border-radius: 8px; padding: 8px; max-width: 600px; margin: 5px auto; display: flex; justify-content: space-around; flex-wrap: wrap; }
        .info-item { font-size: 13px; margin: 3px; }
        .red { color: #ff4444; font-weight: bold; }
        .yellow { color: #ffeb3b; font-weight: bold; }
        .green { color: #00C853; font-weight: bold; }
        
        /* Tela de Bloqueio / Paywall */
        #paywall {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(18, 18, 18, 0.95);
            z-index: 999;
            padding-top: 40px;
        }
        .card-oferta {
            background: #1e1e1e;
            border: 2px solid #ffeb3b;
            border-radius: 10px;
            max-width: 400px;
            margin: 0 auto;
            padding: 20px;
            box-shadow: 0 0 20px rgba(255, 235, 59, 0.3);
        }
        .btn-whats {
            display: inline-block;
            background: #25D366;
            color: white;
            padding: 12px 20px;
            text-decoration: none;
            border-radius: 5px;
            font-weight: bold;
            margin-top: 15px;
            font-size: 16px;
        }
    </style>
</head>
<body>

    <!-- CONTEÚDO DO ROBÔ COM PAINEL DE PERFORMANCE -->
    <div id="conteudo-robo">
        <h2>Robô Ponto Zero - Performance ao Vivo</h2>
        
        <!-- PAINEL DE INDICADORES -->
        <div class="info-box">
            <div class="info-item">Inicial: <span id="valInicial" class="yellow">R$ 1.000,00</span></div>
            <div class="info-item">Acumulado: <span id="valAcumulado" class="green">R$ 1.245,50</span></div>
            <div class="info-item">Taxa de Ganho: <span id="taxaGanho" class="green">82.4%</span></div>
        </div>

        <div class="info-item">Máxima: <span id="maxVal" class="red">0.00</span> | Mínima: <span id="minVal" class="red">0.00</span></div>
        <div class="info-item">Centro: <span id="centroVal" class="yellow">0.00</span> | Preço: <span id="precoAtual" class="green">0.00</span></div>

        <canvas id="grafico" width="600" height="250"></canvas>
    </div>

    <!-- TELA DE BLOQUEIO (APÓS 3 DIAS) -->
    <div id="paywall">
        <div class="card-oferta">
            <h2 style="color: #ff4444;">⏰ Teste Gratuito Expirado!</h2>
            <p>Seu período de teste gratuito de 3 dias chegou ao fim.</p>
            <p>Gostou da ferramenta e quer continuar lucrando com o Ponto Zero?</p>
            <hr style="border-color: #444;">
            <p style="font-size: 18px; color: #ffeb3b; font-weight: bold;">Assinatura Mensal: R$ 14,99 / mês</p>
            <p style="font-size: 14px; color: #aaa;">Garanta seu acesso completo e suporte direto com o criador.</p>
            
            <!-- Link direto para o seu WhatsApp com o número correto -->
            <a href="https://wa.me/5585992704001?text=Olá!%20Quero%20assinar%20o%20Robô%20Ponto%20Zero%20por%20R$%2014,99." class="btn-whats" target="_blank">📱 Ativar via WhatsApp (85 99270-4001)</a>
        </div>
    </div>

    <script>
        // Variáveis globais para guardar os dados do gráfico para o efeito de piscar
        let ultimosPrecos = [];
        let ultimaMax = 0, ultimaMin = 0, ultimoCentro = 0;
        let estadoPiscando = true;

        function verificarLicenca() {
            let dataInstalacao = localStorage.getItem('pz_install_date');
            let agora = new Date().getTime();

            if (!dataInstalacao) {
                localStorage.setItem('pz_install_date', agora);
            } else {
                let diasPassados = (agora - parseInt(dataInstalacao)) / (1000 * 60 * 60 * 24);
                if (diasPassados > 3) {
                    document.getElementById('conteudo-robo').style.display = 'none';
                    document.getElementById('paywall').style.display = 'block';
                    return false;
                }
            }
            return true;
        }

        async function carregarDados() {
            if (!verificarLicenca()) return;

            try {
                let resposta = await fetch('https://api.binance.com/api/v3/klines?symbol=BTCUSDT&interval=1m&limit=150');
                let dados = await resposta.json();

                let precosHigh = [];
                let precosLow = [];
                let precosClose = [];

                dados.forEach(vela => {
                    precosHigh.push(parseFloat(vela[2]));
                    precosLow.push(parseFloat(vela[3]));
                    precosClose.push(parseFloat(vela[4]));
                });

                ultimosPrecos = precosClose;
                ultimaMax = Math.max(...precosHigh);
                ultimaMin = Math.min(...precosLow);
                ultimoCentro = (ultimaMax + ultimaMin) / 2;
                let precoAtual = precosClose[precosClose.length - 1];

                document.getElementById('maxVal').innerText = ultimaMax.toFixed(2);
                document.getElementById('minVal').innerText = ultimaMin.toFixed(2);
                document.getElementById('centroVal').innerText = ultimoCentro.toFixed(2);
                document.getElementById('precoAtual').innerText = precoAtual.toFixed(2);

                desenharGrafico();

            } catch (erro) {
                console.log("Erro ao buscar dados:", erro);
            }
        }

        function desenharGrafico() {
            if (ultimosPrecos.length === 0) return;

            let canvas = document.getElementById('grafico');
            let ctx = canvas.getContext('2d');
            let largura = canvas.width;
            let altura = canvas.height;

            ctx.clearRect(0, 0, largura, altura);

            let margem = ultimaMax - ultimaMin;
            if (margem === 0) margem = 1;

            function mapearY(preco) {
                return altura - ((preco - ultimaMin) / margem) * (altura - 20) - 10;
            }

            // Linha de Preço (Verde)
            ctx.beginPath();
            ctx.strokeStyle = '#00C853';
            ctx.lineWidth = 2;
            for (let i = 0; i < ultimosPrecos.length; i++) {
                let x = (i / (ultimosPrecos.length - 1)) * largura;
                let y = mapearY(ultimosPrecos[i]);
                if (i === 0) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            }
            ctx.stroke();

            // --- BOLINHA PISCANDO NA PONTA DA LINHA DO PREÇO ---
            let ultimoX = largura;
            let ultimoY = mapearY(ultimosPrecos[ultimosPrecos.length - 1]);

            if (estadoPiscando) {
                // Halo externo da bolinha
                ctx.beginPath();
                ctx.arc(ultimoX, ultimoY, 7, 0, 2 * Math.PI);
                ctx.fillStyle = 'rgba(0, 200, 83, 0.4)';
                ctx.fill();
            }

            // Núcleo da bolinha piscando
            ctx.beginPath();
            ctx.arc(ultimoX, ultimoY, 4, 0, 2 * Math.PI);
            ctx.fillStyle = estadoPiscando ? '#00FF66' : '#004411';
            ctx.fill();
            ctx.strokeStyle = '#FFFFFF';
            ctx.lineWidth = 1;
            ctx.stroke();

            // Máxima (Vermelha)
            let yMax = mapearY(ultimaMax);
            ctx.beginPath();
            ctx.strokeStyle = '#ff4444';
            ctx.setLineDash([4, 4]);
            ctx.moveTo(0, yMax);
            ctx.lineTo(largura, yMax);
            ctx.stroke();

            // Mínima (Vermelha)
            let yMin = mapearY(ultimaMin);
            ctx.beginPath();
            ctx.moveTo(0, yMin);
            ctx.lineTo(largura, yMin);
            ctx.stroke();

            // Centro (Amarela)
            let yCentro = mapearY(ultimoCentro);
            ctx.beginPath();
            ctx.strokeStyle = '#ffeb3b';
            ctx.setLineDash([]);
            ctx.lineWidth = 3;
            ctx.moveTo(0, yCentro);
            ctx.lineTo(largura, yCentro);
            ctx.stroke();
        }

        // Intervalo para atualizar os dados da API a cada 5 segundos
        if (verificarLicenca()) {
            carregarDados();
            setInterval(carregarDados, 5000);

            // Intervalo para fazer a bolinha piscar a cada 500 milissegundos
            setInterval(() => {
                estadoPiscando = !estadoPiscando;
                desenharGrafico();
            }, 500);
        }
    </script>

</body>
</html>
