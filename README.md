<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Rocket Tilt Game</title>
  <link rel="stylesheet" href="style.css"/>
</head>
<body>
  <div id="game">
    <div id="rocket"></div>
    <div id="scoreboard">
      <span>Score: <span id="score">0</span></span>
      <span>High Score: <span id="highScore">0</span></span>
    </div>
    <div id="gameOver">Game Over<br/><button onclick="restartGame()">Replay</button></div>
  </div>
  <script src="script.js"></script>
</body>
</html>
<style>
body, html {
    margin: 0;
    padding: 0;
    overflow: hidden;
    font-family: Arial, sans-serif;
    background: black;
  }
  
  #game {
    width: 100vw;
    height: 100vh;
    position: relative;
    background: url('https://i.imgur.com/3xN9jFj.jpg') repeat;
    background-size: cover;
    overflow: hidden;
  }

    #rocket {
  width: 0;
  height: 0;
  border-left: 20px solid transparent;
  border-right: 20px solid transparent;
  border-bottom: 60px solid white;
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
}

  
  .obstacle, .alien, .laser {
    position: absolute;
  }
  
  .obstacle {
    width: 40px;
    height: 40px;
    background: gray;
    border-radius: 50%;
  }
  
  .alien {
    width: 50px;
    height: 50px;
    background: url('https://i.imgur.com/dZTPJYB.png') no-repeat center/contain;
  }
  
  .laser {
    width: 4px;
    height: 20px;
    background: red;
  }
  
  #scoreboard {
    position: absolute;
    top: 10px;
    left: 10px;
    color: white;
    font-size: 18px;
    z-index: 10;
  }
  
  #gameOver {
    position: absolute;
    top: 40%;
    width: 100%;
   
    color: white;
    display: none;
    z-index: 20;
  }
  
  #gameOver button {
    margin-top: 10px;
    padding: 10px 20px;
    font-size: 16px;
    cursor: pointer;
  }
  </style>
  <script>
  const rocket = document.getElementById('rocket');
  const game = document.getElementById('game');
  const scoreDisplay = document.getElementById('score');
  const highScoreDisplay = document.getElementById('highScore');
  const gameOverScreen = document.getElementById('gameOver');
  
  let score = 0;
  let highScore = localStorage.getItem('highScore') || 0;
  let rocketX = window.innerWidth / 2;
  let speed = 2;
  let gameInterval, spawnInterval, laserInterval;
  let gameRunning = true;
  
  highScoreDisplay.innerText = highScore;
  
  function createObstacle() {
    const obs = document.createElement('div');
    obs.classList.add('obstacle');
    obs.style.left = Math.random() * (window.innerWidth - 40) + 'px';
    obs.style.top = '-40px';
    game.appendChild(obs);
  }
  
  function createAlien() {
    const alien = document.createElement('div');
    alien.classList.add('alien');
    alien.style.left = Math.random() * (window.innerWidth - 50) + 'px';
    alien.style.top = '-50px';
    game.appendChild(alien);
  
    setTimeout(() => shootLaser(alien), 1500);
  }
  
  function shootLaser(alien) {
    if (!alien.parentElement) return;
    const laser = document.createElement('div');
    laser.classList.add('laser');
    laser.style.left = alien.offsetLeft + 23 + 'px';
    laser.style.top = alien.offsetTop + 40 + 'px';
    game.appendChild(laser);
  }
  
  function moveObjects() {
    document.querySelectorAll('.obstacle, .alien, .laser').forEach(el => {
      let top = parseFloat(el.style.top);
      el.style.top = (top + speed) + 'px';
  
      if (el.classList.contains('laser')) {
        if (checkCollision(el, rocket)) endGame();
      } else if (checkCollision(el, rocket)) {
        endGame();
      }
  
      if (top > window.innerHeight) {
        el.remove();
        if (!el.classList.contains('laser')) score++;
        updateScore();
      }
    });
  }
  
  function updateScore() {
    scoreDisplay.innerText = score;
    if (score > highScore) {
      highScore = score;
      localStorage.setItem('highScore', highScore);
      highScoreDisplay.innerText = highScore;
    }
    speed = 2 + score / 20; // Game gets faster
  }
  
  function checkCollision(a, b) {
    const rect1 = a.getBoundingClientRect();
    const rect2 = b.getBoundingClientRect();
    return !(
      rect1.bottom < rect2.top || 
      rect1.top > rect2.bottom || 
      rect1.right < rect2.left || 
      rect1.left > rect2.right
    );
  }
  
  function endGame() {
    clearInterval(gameInterval);
    clearInterval(spawnInterval);
    clearInterval(laserInterval);
    gameRunning = false;
    gameOverScreen.style.display = 'block';
  }
  
  function restartGame() {
    game.innerHTML = `<div id="rocket"></div><div id="scoreboard">
        <span>Score: <span id="score">0</span></span>
        <span>High Score: <span id="highScore">${highScore}</span></span>
      </div><div id="gameOver">Game Over<br/><button onclick="restartGame()">Replay</button></div>`;
    rocketX = window.innerWidth / 2;
    score = 0;
    speed = 2;
    scoreDisplay.innerText = score;
    gameOverScreen.style.display = 'none';
    gameRunning = true;
  
    init();
  }
  
  function init() {
    rocket.style.left = rocketX + 'px';
  
    // Device tilt controls
    window.addEventListener('deviceorientation', e => {
      if (!gameRunning) return;
      let tilt = e.gamma; // side to side tilt
      rocketX += tilt * 1.5;
      rocketX = Math.max(0, Math.min(window.innerWidth - 40, rocketX));
      rocket.style.left = rocketX + 'px';
    });
  
    gameInterval = setInterval(moveObjects, 20);
    spawnInterval = setInterval(() => {
      createObstacle();
      if (Math.random() < 0.5) createAlien();
    }, 1000);
  }
  
  init();
  </script>
  </html>
</body>
      
