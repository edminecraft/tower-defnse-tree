<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Agrinho: Agro Forte e Sustentavel</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #1b3d22;
            color: #ffffff;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
        }

        h1 {
            margin: 5px 0;
            font-size: 1.8rem;
            text-shadow: 2px 2px #000;
            text-align: center;
        }

        p {
            margin: 5px 0 15px 0;
            font-size: 1rem;
            color: #a3e2ab;
            text-align: center;
        }

        #game-container {
            position: relative;
        }

        canvas {
            background-color: #7ece65;
            border: 6px solid #5c3a21;
            border-radius: 10px;
            box-shadow: 0 10px 20px rgba(0,0,0,0.5);
            cursor: crosshair;
        }

        #ui-panel {
            margin-top: 15px;
            display: flex;
            gap: 15px;
            background: rgba(0,0,0,0.5);
            padding: 15px;
            border-radius: 10px;
            align-items: center;
        }

        button {
            background-color: #4CAF50;
            border: none;
            color: white;
            padding: 12px 20px;
            font-size: 14px;
            border-radius: 8px;
            cursor: pointer;
            transition: 0.2s;
            font-weight: bold;
        }

        button:hover {
            background-color: #45a049;
            transform: scale(1.05);
        }

        button.active {
            background-color: #ff9800;
            box-shadow: 0 0 10px #fff;
        }

        #startBtn {
            background-color: #008CBA;
        }

        #startBtn:hover {
            background-color: #007B9A;
        }

        #stats {
            font-size: 1.2rem;
            margin-top: 10px;
            background: rgba(0,0,0,0.4);
            padding: 8px 20px;
            border-radius: 5px;
            display: flex;
            gap: 20px;
        }
    </style>
</head>
<body>

    <h1>🌱 Guardiao Agrinho: Agro Forte, Futuro Sustentavel 🌳</h1>
    <p>Clique em uma armadilha abaixo e depois clique na ESTRADA DE TERRA amarela para planta-la!</p>
    
    <div id="game-container">
        <canvas id="gameCanvas" width="800" height="400"></canvas>
    </div>

    <div id="stats">
        <div>Nivel: <strong id="levelDisplay" style="color: #ff9800;">1</strong></div>
        <div>Vida da Arvore: <strong id="treeHpDisplay" style="color: #ff4d4d;">100</strong>%</div>
        <div>Vida do Lenhador: <strong id="jackHpDisplay" style="color: #a3e2ab;">100</strong>%</div>
    </div>

    <div id="ui-panel">
        <button id="trap1" onclick="selectTrap('Cerca')">🪤 Cerca Viva</button>
        <button id="trap2" onclick="selectTrap('Raiz')">🌱 Raizes Fortes</button>
        <button id="trap3" onclick="selectTrap('Abelha')">🐝 Abelhas</button>
        <button id="startBtn" onclick="startRound()">Iniciar Invasao</button>
    </div>

    <script>
        let canvas = null;
        let ctx = null;
        let isRoundActive = false;
        let level = 1;
        let treeHp = 100;
        
        const tree = { x: 60, y: 140, width: 80, height: 120 };
        let lumberjack = { x: 700, y: 175, width: 45, height: 70 };
        
        let jackMaxHp = 100;
        let jackHp = 100;
        let jackSpeed = 1.2;

        let traps = [];
        let selectedTrap = null;

        document.addEventListener("DOMContentLoaded", function() {
            canvas = document.getElementById('gameCanvas');
            if (canvas) {
                ctx = canvas.getContext('2d');
                
                canvas.addEventListener('click', function(event) {
                    if (!selectedTrap) return;

                    const rect = canvas.getBoundingClientRect();
                    const x = event.clientX - rect.left - 15;
                    const y = event.clientY - rect.top - 15;

                    // Permite posicionar apenas dentro da faixa central da estrada de terra
                    if (x > 140 && x < 720 && y > 160 && y < 240) {
                        traps.push({
                            type: selectedTrap,
                            x: x,
                            y: y,
                            width: 30,
                            height: 30,
                            active: true
                        });

                        selectedTrap = null;
                        document.getElementById('trap1').classList.remove('active');
                        document.getElementById('trap2').classList.remove('active');
                        document.getElementById('trap3').classList.remove('active');
                    }
                });

                resetRoundData();
                gameLoop();
            }
        });

        function selectTrap(type) {
            selectedTrap = type;
            document.getElementById('trap1').classList.remove('active');
            document.getElementById('trap2').classList.remove('active');
            document.getElementById('trap3').classList.remove('active');
            
            if (type === 'Cerca') document.getElementById('trap1').classList.add('active');
            if (type === 'Raiz') document.getElementById('trap2').classList.add('active');
            if (type === 'Abelha') document.getElementById('trap3').classList.add('active');
        }

        function startRound() {
            if (!isRoundActive && treeHp > 0) {
                isRoundActive = true;
                document.getElementById('startBtn').disabled = true;
            }
        }

        function resetRoundData() {
            lumberjack.x = 700;
            lumberjack.y = 175;
            jackMaxHp = 100 + (level - 1) * 50; 
            jackHp = jackMaxHp;
            jackSpeed = 1.2 + (level - 1) * 0.3; 
            
            updateUI();
        }

        function updateUI() {
            const levelDisplay = document.getElementById('levelDisplay');
            const treeHpDisplay = document.getElementById('treeHpDisplay');
            const jackHpDisplay = document.getElementById('jackHpDisplay');

            if(levelDisplay) levelDisplay.innerText = level;
            if(treeHpDisplay) treeHpDisplay.innerText = Math.max(0, Math.ceil(treeHp));
            if(jackHpDisplay) jackHpDisplay.innerText = Math.max(0, Math.ceil((jackHp / jackMaxHp) * 100));
        }

        function drawTree() {
            ctx.fillStyle = '#5c3a21';
            ctx.fillRect(tree.x + 30, tree.y + 40, 20, 80);
            
            ctx.fillStyle = '#1b5e20';
            ctx.beginPath();
            ctx.arc(tree.x + 40, tree.y + 20, 40, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#2e7d32';
            ctx.beginPath();
            ctx.arc(tree.x + 20, tree.y + 35, 30, 0, Math.PI * 2);
            ctx.fill();

            ctx.fillStyle = '#4caf50';
            ctx.beginPath();
            ctx.arc(tree.x + 60, tree.y + 35, 30, 0, Math.PI * 2);
            ctx.fill();
        }

        function drawLumberjack() {
            if (jackHp <= 0) return;

            // Corpo/Camisa xadrez (Quadrado Vermelho)
            ctx.fillStyle = '#d32f2f';
            ctx.fillRect(lumberjack.x, lumberjack.y + 20, lumberjack.width, 35);
            
            // Calça (Retângulo Azul)
            ctx.fillStyle = '#1976d2';
            ctx.fillRect(lumberjack.x, lumberjack.y + 55, lumberjack.width, 15);

            // Cabeça (Círculo Bege)
            ctx.fillStyle = '#ffcc80';
            ctx.beginPath();
            ctx.arc(lumberjack.x + 22, lumberjack.y + 10, 12, 0, Math.PI * 2);
            ctx.fill();

            // Machado do Lenhador
            ctx.fillStyle = '#757575';
            ctx.fillRect(lumberjack.x - 10, lumberjack.y + 15, 15, 8);
        } // <-- ESSA CHAVE ESTAVA FALTANDO E QUEBRAVA O CÓDIGO

        function drawTraps() {
            traps.forEach(trap => {
                if (!trap.active) return;
                
                if (trap.type === 'Cerca') {
                    ctx.fillStyle = '#2e7d32';
                    ctx.fillRect(trap.x, trap.y, trap.width, trap.height);
                } else if (trap.type === 'Raiz') {
                    ctx.fillStyle = '#8d6e63';
                    ctx.fillRect(trap.x + 10, trap.y, 10, trap.height);
                    ctx.fillRect(trap.x, trap.y + 10, trap.width, 10);
                } else if (trap.type === 'Abelha') {
                    ctx.fillStyle = '#fbc02d';
                    ctx.beginPath();
                    ctx.arc(trap.x + 15, trap.y + 15, 10, 0, Math.PI * 2);
                    ctx.fill();
                }
            });
        }

        function updateGame() {
            if (!isRoundActive || treeHp <= 0) return;

            // Avanço do lenhador
            lumberjack.x -= jackSpeed;

            // Detecção de colisão com armadilhas
            traps.forEach(trap => {
                if (trap.active && 
                    lumberjack.x < trap.x + trap.width &&
                    lumberjack.x + lumberjack.width > trap.x) {
                    
                    trap.active = false; 

                    if (trap.type === 'Cerca') jackHp -= 30;
                    if (trap.type === 'Raiz') jackHp -= 20;
                    if (trap.type === 'Abelha') jackHp -= 50;
                }
            });

            // Derrota do lenhador (Avança de nível)
            if (jackHp <= 0) {
                isRoundActive = false;
                level++;
                document.getElementById('startBtn').disabled = false;
                resetRoundData();}//Ataque à árvore principalif (lumberjack.x <= tree.x + tree.width) {treeHp -= 25;isRoundActive = false;if (treeHp <= 0) {treeHp = 0;} else {document.getElementById('startBtn').disabled = false;resetRoundData();}}updateUI();}function gameLoop() {// Fundo do cenário (Grama)ctx.fillStyle = '#7ece65';ctx.fillRect(0, 0, canvas.width, canvas.height);// Estrada de Terra amarelactx.fillStyle = '#f4d068';ctx.fillRect(140, 160, 580, 80);updateGame();drawTree();drawTraps();drawLumberjack();// Mensagem de Fim de Jogoif (treeHp <= 0) {ctx.fillStyle = 'rgba(0, 0, 0, 0.8)';ctx.fillRect(0, 0, canvas.width, canvas.height);ctx.fillStyle = '#ff4d4d';ctx.font = 'bold 36px Segoe UI';ctx.textAlign = 'center';ctx.fillText('FIM DE JOGO', canvas.width / 2, canvas.height / 2 - 10);ctx.fillStyle = '#ffffff';ctx.font = '20px Segoe UI';ctx.fillText(Defesa Sustentável até o Nível ${level}, canvas.width / 2, canvas.height / 2 + 30);return;}requestAnimationFrame(gameLoop);}
              }
