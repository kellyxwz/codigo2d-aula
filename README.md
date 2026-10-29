<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jogo 2D no Navegador</title>
    <style>
        body {
            margin: 0;
            background-color: #222;
            color: white;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        h1 {
            margin-bottom: 5px;
        }
        p {
            margin-top: 0;
            color: #aaa;
        }
        canvas {
            border: 4px solid #fff;
            background-color: #111;
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.2);
        }
    </style>
</head>
<body>

    <h1>Caça ao Ponto!</h1>
    <p>Use as setas <b>Esquerda</b> e <b>Direita</b> para mover a barra e pegar os blocos amarelos.</p>
    
    <canvas id="gameCanvas" width="600" height="400"></canvas>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        // Configuração do Jogador
        const player = {
            x: canvas.width / 2 - 50,
            y: canvas.height - 30,
            width: 100,
            height: 15,
            speed: 8,
            dx: 0
        };

        // Configuração do Item (Fruta/Ponto)
        const item = {
            x: Math.random() * (canvas.width - 20),
            y: 0,
            width: 20,
            height: 20,
            speed: 4
        };

        let score = 0;
        let lives = 3;
        let isGameOver = false;

        // Controle do Teclado
        document.addEventListener("keydown", (e) => {
            if (e.key === "ArrowLeft") player.dx = -player.speed;
            if (e.key === "ArrowRight") player.dx = player.speed;
        });

        document.addEventListener("keyup", (e) => {
            if (e.key === "ArrowLeft" && player.dx < 0) player.dx = 0;
            if (e.key === "ArrowRight" && player.dx > 0) player.dx = 0;
        });

        // Atualizar Posições
        function update() {
            if (isGameOver) return;

            // Movimento do jogador
            player.x += player.dx;

            // Limites da tela para o jogador
            if (player.x < 0) player.x = 0;
            if (player.x + player.width > canvas.width) {
                player.x = canvas.width - player.width;
            }

            // Movimento do item
            item.y += item.speed;

            // Colisão com o jogador
            if (
                item.x < player.x + player.width &&
                item.x + item.width > player.x &&
                item.y < player.y + player.height &&
                item.y + item.height > player.y
            ) {
                score++;
                resetItem();
                // Aumenta levemente a dificuldade a cada ponto
                item.speed += 0.2;
            }

            // Se o item cair no chão
            if (item.y > canvas.height) {
                lives--;
                resetItem();
                if (lives <= 0) {
                    isGameOver = true;
                }
            }
        }

        // Reposicionar item no topo
        function resetItem() {
            item.y = 0;
            item.x = Math.random() * (canvas.width - item.width);
        }

        // Desenhar Elementos na Tela
        function draw() {
            // Limpa a tela
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (!isGameOver) {
                // Desenhar Jogador (Barra)
                ctx.fillStyle = "#00ffcc";
                ctx.fillRect(player.x, player.y, player.width, player.height);

                // Desenhar Item
                ctx.fillStyle = "#ffcc00";
                ctx.fillRect(item.x, item.y, item.width, item.height);

                // Desenhar Placar e Vidas
                ctx.fillStyle = "#ffffff";
                ctx.font = "16px Arial";
                ctx.fillText(`Pontos: ${score}`, 20, 30);
                ctx.fillText(`Vidas: ${lives}`, canvas.width - 90, 30);
            } else {
                // Tela de Game Over
                ctx.fillStyle = "#ff3333";
                ctx.font = "30px Arial";
                ctx.textAlign = "center";
                ctx.fillText("FIM DE JOGO", canvas.width / 2, canvas.height / 2 - 20);

                ctx.fillStyle = "#ffffff";
                ctx.font = "18px Arial";
                ctx.fillText(`Pontuação Final: ${score}`, canvas.width / 2, canvas.height / 2 + 20);
                ctx.fillText("Atualize a página para jogar novamente", canvas.width / 2, canvas.height / 2 + 60);
                ctx.textAlign = "left"; // Reseta o alinhamento
            }
        }

        // Loop Principal do Jogo
        function loop() {
            update();
            draw();
            requestAnimationFrame(loop);
        }

        // Inicia o loop
        loop();
    </script>

</body>
</html>
