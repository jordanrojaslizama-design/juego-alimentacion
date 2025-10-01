# juego-alimentacion<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Juego de Buena Alimentación</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: sans-serif;
      text-align: center;
      background-color: #f0fff0;
    }
    canvas {
      background: #dff0d8;
      display: block;
      margin: 20px auto;
      border: 2px solid #4CAF50;
    }
    #score {
      font-size: 24px;
      margin: 10px;
    }
    .question {
      margin: 20px;
    }
    .hidden {
      display: none;
    }
    button {
      padding: 10px 20px;
      margin: 10px;
      font-size: 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover {
      background-color: #45a049;
    }
  </style>
</head>
<body>

  <h1>🍎 Juego de Buena Alimentación</h1>
  <div id="score">Puntos: 0</div>
  <canvas id="gameCanvas" width="400" height="400"></canvas>

  <div id="questionnaire" class="hidden">
    <h2>Responde estas preguntas:</h2>
    <form id="quizForm"></form>
    <button onclick="submitQuiz()">Enviar respuestas</button>
  </div>

  <div id="finalMessage" class="hidden"></div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreDisplay = document.getElementById('score');
    const questionnaire = document.getElementById('questionnaire');
    const quizForm = document.getElementById('quizForm');
    const finalMessage = document.getElementById('finalMessage');

    const basket = { x: 180, width: 40, height: 20 };
    let items = [];
    let score = 0;
    let gameOver = false;

    const healthyFoods = ['🍎', '🥕', '🍌', '🥦'];
    const junkFoods = ['🍔', '🍟', '🍩'];

    function randomItem() {
      const isHealthy = Math.random() > 0.3;
      return {
        x: Math.random() * (canvas.width - 20),
        y: 0,
        emoji: isHealthy ? healthyFoods[Math.floor(Math.random() * healthyFoods.length)] : junkFoods[Math.floor(Math.random() * junkFoods.length)],
        isHealthy
      };
    }

    function drawBasket() {
      ctx.fillStyle = '#4CAF50';
      ctx.fillRect(basket.x, canvas.height - basket.height, basket.width, basket.height);
    }

    function drawItems() {
      ctx.font = '24px Arial';
      items.forEach(item => {
        ctx.fillText(item.emoji, item.x, item.y);
      });
    }

    function updateItems() {
      items.forEach(item => item.y += 2);
      items = items.filter(item => item.y < canvas.height);
    }

    function checkCatch() {
      items = items.filter(item => {
        if (
          item.y > canvas.height - basket.height &&
          item.x > basket.x &&
          item.x < basket.x + basket.width
        ) {
          if (item.isHealthy) {
            score += 1;
          } else {
            score -= 2;
          }
          scoreDisplay.textContent = `Puntos: ${score}`;
          if (score >= 20 || score < 0) {
            endGame();
          }
          return false;
        }
        return true;
      });
    }

    function endGame() {
      gameOver = true;
      clearInterval(spawnInterval);
      clearInterval(gameLoop);
      canvas.classList.add('hidden');
      questionnaire.classList.remove('hidden');
      loadQuestions();
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawBasket();
      drawItems();
    }

    function loadQuestions() {
      const questions = [
        { q: '¿Cuál de estos alimentos es rico en hierro?', a: 'Espinaca', o: ['Pan blanco', 'Espinaca', 'Gaseosa'] },
        { q: '¿Qué vitamina ayuda a absorber el hierro?', a: 'Vitamina C', o: ['Vitamina A', 'Vitamina C', 'Vitamina D'] },
        { q: '¿Cuál es una fruta saludable?', a: 'Manzana', o: ['Hamburguesa', 'Donut', 'Manzana'] },
        { q: '¿Qué debemos evitar para prevenir la anemia?', a: 'Comida chatarra', o: ['Comida chatarra', 'Verduras', 'Frutas'] },
        { q: '¿Qué órgano produce los glóbulos rojos?', a: 'Médula ósea', o: ['Hígado', 'Médula ósea', 'Estómago'] },
        { q: '¿Qué bebida ayuda a absorber mejor el hierro?', a: 'Jugo de naranja', o: ['Café', 'Té', 'Jugo de naranja'] },
        { q: '¿Qué grupo debe cuidar especialmente el hierro?', a: 'Mujeres embarazadas', o: ['Hombres adultos', 'Niños pequeños', 'Mujeres embarazadas'] }
      ];

      questions.forEach((question, index) => {
        const div = document.createElement('div');
        div.classList.add('question');
        div.innerHTML = `<p><strong>${index + 1}. ${question.q}</strong></p>`;
        question.o.forEach(option => {
          div.innerHTML += `<label><input type="radio" name="q${index}" value="${option}"> ${option}</label><br>`;
        });
        quizForm.appendChild(div);
      });

      quizForm.dataset.answers = JSON.stringify(questions.map(q => q.a));
    }

    function submitQuiz() {
      const correctAnswers = JSON.parse(quizForm.dataset.answers);
      let quizScore = 0;

      correctAnswers.forEach((answer, i) => {
        const selected = document.querySelector(`input[name="q${i}"]:checked`);
        if (selected && selected.value === answer) {
          quizScore += 3;
        }
      });

      const totalScore = score + quizScore;

      questionnaire.classList.add('hidden');
      finalMessage.classList.remove('hidden');

      if (totalScore >= 20) {
        finalMessage.innerHTML = `<h2>🎉 ¡Felicidades! Terminaste con ${totalScore} puntos. Has ganado.</h2>`;
      } else {
        finalMessage.innerHTML = `<h2>😢 Obtuviste ${totalScore} puntos. Intenta nuevamente.</h2>`;
      }
    }

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft' && basket.x > 0) basket.x -= 20;
      if (e.key === 'ArrowRight' && basket.x < canvas.width - basket.width) basket.x += 20;
    });

    let spawnInterval = setInterval(() => {
      if (!gameOver) items.push(randomItem());
    }, 800);

    let gameLoop = setInterval(() => {
      if (!gameOver) {
        updateItems();
        checkCatch();
        draw();
      }
    }, 50);
  </script>
</body>
</html>
