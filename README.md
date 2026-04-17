# index.html567
小恐龍遊戲
<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>小恐龍：進化挑戰 (無無敵版)</title>
    <style>
        body { margin: 0; display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #f0f0f0; overflow: hidden; font-family: 'Segoe UI', sans-serif; }
        #game-container { position: relative; width: 800px; height: 250px; background-color: white; border-bottom: 2px solid #535353; overflow: hidden; box-shadow: 0 10px 30px rgba(0,0,0,0.1); }
        
        /* 恐龍樣式 - 改用背景圖 */
        #dino { 
            position: absolute; bottom: 0; left: 50px; width: 50px; height: 50px; 
            background: url('./assets/dino.png') no-repeat center/contain;
            z-index: 10; transition: filter 0.3s;
        }
        
        /* 進化視覺效果 (僅濾鏡，不無敵) */
        .evo-gold { filter: drop-shadow(0 0 5px #FFD700) sepia(1) saturate(5) hue-rotate(10deg); }
        .evo-flame { filter: drop-shadow(0 0 8px #ff4757) sepia(1) saturate(8) hue-rotate(-20deg); }

        /* 障礙物樣式 */
        .obstacle { position: absolute; bottom: 0; background-size: contain; background-repeat: no-repeat; background-position: center; }
        .cactus-img { background-image: url('./assets/cactus.png'); }
        .bird-img { background-image: url('./assets/bird.png'); }

        /* UI */
        #ui { position: absolute; top: 10px; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; z-index: 15; color: #535353; font-weight: bold; }
        #message { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); text-align: center; display: none; z-index: 20; background: rgba(255,255,255,0.9); padding: 20px; border: 2px solid #535353; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="ui">
        <div id="evo-status">狀態: 幼年體</div>
        <div id="score">SCORE: 00000</div>
    </div>
    <div id="dino"></div>
    <div id="message">
        <h1>GAME OVER</h1>
        <p>按下 [空白鍵] 重新挑戰</p>
    </div>
</div>

<script>
    const container = document.getElementById('game-container');
    const dinoElement = document.getElementById('dino');
    const scoreElement = document.getElementById('score');
    const evoStatusElement = document.getElementById('evo-status');
    const messageElement = document.getElementById('message');

    let isRunning = true;
    let score = 0;
    let gameSpeed = 6;
    let currentEvo = 'normal';
    
    let dino = { x: 50, y: 0, vy: 0, w: 45, h: 45, jumping: false };
    let obstacles = [];
    let timer = 0;

    // 監聽跳躍
    document.addEventListener('keydown', (e) => {
        if (e.code === 'Space' || e.code === 'ArrowUp') {
            if (!isRunning) reset();
            else if (!dino.jumping) { dino.vy = -15; dino.jumping = true; }
        }
    });

    function loop() {
        if (!isRunning) return;

        // 1. 物理運動
        dino.vy += 0.8; // 重力
        dino.y -= dino.vy;
        if (dino.y <= 0) { dino.y = 0; dino.vy = 0; dino.jumping = false; }
        dinoElement.style.bottom = dino.y + 'px';

        // 2. 障礙物管理
        if (++timer > (100 - gameSpeed)) {
            spawnObstacle();
            timer = 0;
        }

        for (let i = obstacles.length - 1; i >= 0; i--) {
            let o = obstacles[i];
            o.x -= gameSpeed;
            o.el.style.left = o.x + 'px';
            
            // 碰撞偵測 (即使是火焰型態也會死)
            if (checkHit(dino, o)) {
                gameOver();
            }

            if (o.x < -50) {
                container.removeChild(o.el);
                obstacles.splice(i, 1);
            }
        }

        // 3. 進化與分數
        score += 0.15;
        let s = Math.floor(score);
        scoreElement.innerText = 'SCORE: ' + s.toString().padStart(5, '0');
        
        if (s >= 1000 && currentEvo !== 'flame') {
            currentEvo = 'flame';
            dinoElement.className = 'evo-flame';
            evoStatusElement.innerText = '狀態: 火焰進化 (極速挑戰)';
            gameSpeed += 2;
        } else if (s >= 500 && s < 1000 && currentEvo !== 'gold') {
            currentEvo = 'gold';
            dinoElement.className = 'evo-gold';
            evoStatusElement.innerText = '狀態: 金色覺醒';
            gameSpeed += 1;
        }

        requestAnimationFrame(loop);
    }

    function spawnObstacle() {
        let isBird = Math.random() > 0.7;
        let obj = {
            x: 800,
            y: isBird ? 70 : 0,
            w: isBird ? 40 : 30,
            h: isBird ? 30 : 50,
            el: document.createElement('div')
        };
        obj.el.className = 'obstacle ' + (isBird ? 'bird-img' : 'cactus-img');
        obj.el.style.width = obj.w + 'px';
        obj.el.style.height = obj.h + 'px';
        obj.el.style.bottom = obj.y + 'px';
        container.appendChild(obj.el);
        obstacles.push(obj);
    }

    function checkHit(d, o) {
        // 稍微縮減碰撞判定範圍，讓手感更好 (Padding)
        let p = 10; 
        return d.x + p < o.x + o.w - p &&
               d.x + d.w - p > o.x + p &&
               d.y + p < o.y + o.h - p &&
               d.y + d.h - p > o.y + p;
    }

    function gameOver() {
        isRunning = false;
        messageElement.style.display = 'block';
    }

    function reset() {
        location.reload(); // 最穩定的重開方式
    }

    loop();
</script>
</body>
</html>
