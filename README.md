<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plataforma 3D Estilo Mario</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            background-color: #5c94fc; /* Céu clássico do Mario */
            font-family: 'Courier New', Courier, monospace;
        }
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            font-size: 24px;
            font-weight: bold;
            text-shadow: 2px 2px #000;
            pointer-events: none;
        }
        #instructions {
            position: absolute;
            bottom: 20px;
            width: 100%;
            text-align: center;
            color: white;
            font-size: 16px;
            text-shadow: 1px 1px #000;
            pointer-events: none;
        }
    </style>
</head>
<body>

    <div id="ui">Moedas: <span id="score">0</span></div>
    <div id="instructions">Use <b>W, A, S, D</b> para mover e <b>ESPAÇO</b> para pular!</div>

    <!-- Carrega a biblioteca Three.js direto via CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

    <script>
        // --- 1. CONFIGURAÇÃO DA CENA 3D ---
        const scene = new THREE.Scene();
        scene.background = new THREE.Color(0x5c94fc);
        scene.fog = new THREE.Fog(0x5c94fc, 20, 70);

        const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.shadowMap.enabled = true;
        document.body.appendChild(renderer.domElement);

        // Luzes
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);

        const dirLight = new THREE.DirectionalLight(0xffffff, 0.8);
        dirLight.position.set(10, 20, 10);
        dirLight.castShadow = true;
        scene.add(dirLight);

        // --- 2. CENÁRIO E PLATAFORMAS ---
        const platforms = [];

        function createPlatform(x, y, z, width, height, depth, color) {
            const geometry = new THREE.BoxGeometry(width, height, depth);
            const material = new THREE.MeshStandardMaterial({ color: color });
            const mesh = new THREE.Mesh(geometry, material);
            mesh.position.set(x, y, z);
            mesh.receiveShadow = true;
            scene.add(mesh);
            platforms.push(mesh);
            return mesh;
        }

        // Chão Principal (Estilo grama/terra do Mario)
        createPlatform(0, -1, 0, 30, 2, 30, 0x33cc33); // Grama em cima
        createPlatform(0, -2.5, 0, 30, 2, 30, 0x8b4513); // Terra embaixo

        // Plataformas flutuantes para pular
        createPlatform(-8, 2, -5, 5, 1, 5, 0xd95763);
        createPlatform(0, 4, -10, 6, 1, 5, 0xd95763);
        createPlatform(8, 6, -15, 5, 1, 5, 0xd95763);

        // --- 3. JOGADOR (MARIO 3D SIMULADO) ---
        const playerGeometry = new THREE.BoxGeometry(1.2, 1.8, 1.2);
        const playerMaterial = new THREE.MeshStandardMaterial({ color: 0xe52521 }); // Vermelho Mario
        const player = new THREE.Mesh(playerGeometry, playerMaterial);
        player.position.set(0, 1, 8);
        player.castShadow = true;
        scene.add(player);

        // Variáveis de Física do Jogador
        const playerPhysics = {
            vy: 0,
            speed: 0.15,
            gravity: 0.012,
            jumpForce: 0.28,
            isGrounded: false
        };

        // --- 4. MOEDAS COLECIONÁVEIS ---
        const coins = [];
        const coinGeometry = new THREE.CylinderGeometry(0.6, 0.6, 0.2, 16);
        const coinMaterial = new THREE.MeshStandardMaterial({ color: 0xf8d000, metalness: 0.5, roughness: 0.2 });

        function spawnCoin(x, y, z) {
            const coin = new THREE.Mesh(coinGeometry, coinMaterial);
            coin.rotation.x = Math.PI / 2;
            coin.position.set(x, y, z);
            scene.add(coin);
            coins.push(coin);
        }

        // Distribuir moedas pelas plataformas
        spawnCoin(0, 1, 0);
        spawnCoin(-8, 3.5, -5);
        spawnCoin(0, 5.5, -10);
        spawnCoin(8, 7.5, -15);

        let score = 0;

        // --- 5. CONTROLES DO TECLADO ---
        const keys = {};
        window.addEventListener('keydown', (e) => { keys[e.code] = true; });
        window.addEventListener('keyup', (e) => { keys[e.code] = false; });

        // --- 6. LOOP PRINCIPAL DO JOGO ---
        function animate() {
            requestAnimationFrame(animate);

            // Movimentação nas setas / W, A, S, D
            let moveX = 0;
            let moveZ = 0;

            if (keys['KeyW'] || keys['ArrowUp']) moveZ -= 1;
            if (keys['KeyS'] || keys['ArrowDown']) moveZ += 1;
            if (keys['KeyA'] || keys['ArrowLeft']) moveX -= 1;
            if (keys['KeyD'] || keys['ArrowRight']) moveX += 1;

            // Normalizar movimento diagonal
            if (moveX !== 0 && moveZ !== 0) {
                moveX *= 0.7071;
                moveZ *= 0.7071;
            }

            player.position.x += moveX * playerPhysics.speed;
            player.position.z += moveZ * playerPhysics.speed;

            // Pulo e Gravidade
            if (keys['Space'] && playerPhysics.isGrounded) {
                playerPhysics.vy = playerPhysics.jumpForce;
                playerPhysics.isGrounded = false;
            }

            playerPhysics.vy -= playerPhysics.gravity;
            player.position.y += playerPhysics.vy;

            // --- COLISÕES SIMPLES COM AS PLATAFORMAS ---
            playerPhysics.isGrounded = false;

            platforms.forEach(plat => {
                // Pega os limites da plataforma (caixa delimitadora aproximada)
                const pBox = new THREE.Box3().setFromObject(plat);
                const playerBox = new THREE.Box3().setFromObject(player);

                // Se houver interseção no eixo X e Z e o jogador estiver descendo em cima dela
                if (
                    player.position.x + 0.6 > pBox.min.x && player.position.x - 0.6 < pBox.max.x &&
                    player.position.z + 0.6 > pBox.min.z && player.position.z - 0.6 < pBox.max.z
                ) {
                    // Checa se bateu no topo da plataforma
                    if (player.position.y - 0.9 <= pBox.max.y && player.position.y - 0.9 >= pBox.max.y - 0.8 && playerPhysics.vy <= 0) {
                        player.position.y = pBox.max.y + 0.9;
                        playerPhysics.vy = 0;
                        playerPhysics.isGrounded = true;
                    }
                }
            });

            // Se cair muito no buraco (fora das plataformas), renasce no início
            if (player.position.y < -15) {
                player.position.set(0, 2, 8);
                playerPhysics.vy = 0;
            }

            // --- COLETA DE MOEDAS ---
            coins.forEach((coin, index) => {
                coin.rotation.z += 0.03; // Girar a moeda

                // Distância simples entre jogador e moeda
                const dist = player.position.distanceTo(coin.position);
                if (dist < 1.5) {
                    scene.remove(coin);
                    coins.splice(index, 1);
                    score += 10;
                    document.getElementById('score').innerText = score;
                }
            });

            // --- CÂMERA SEGUIDORA (Estilo Mario 3D) ---
            camera.position.x = player.position.x;
            camera.position.y = player.position.y + 5;
            camera.position.z = player.position.z + 9;
            camera.lookAt(player.position.x, player.position.y + 1, player.position.z);

            renderer.render(scene, camera);
        }

        // Ajustar redimensionamento da janela
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        animate();
    </script>
</body>
</html>
