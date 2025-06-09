<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Gimble Gamble</title>
  <style>
    body {
      font-family: "Segoe UI", sans-serif;
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      padding: 15px;
      box-sizing: border-box;
      overflow-y: auto;
    }
    .quiz-box {
      background: rgba(0, 0, 0, 0.75);
      padding: 2rem;
      border-radius: 20px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.5);
      width: 90%;
      max-width: 450px;
      text-align: center;
      position: relative;
    }
    h2 {
      margin-bottom: 1rem;
    }
    select, button {
      width: 100%;
      padding: 0.75rem;
      margin: 0.5rem 0;
      border: none;
      border-radius: 12px;
      font-size: 1rem;
      cursor: pointer;
      transition: transform 0.2s ease, background 0.3s ease;
    }
    select {
      background: #f1c40f;
      color: #333;
    }
    button {
      background: linear-gradient(to right, #00c6ff, #0072ff);
      color: white;
      font-weight: bold;
    }
    button:hover, select:hover {
      transform: scale(1.03);
    }
    .question {
      font-size: 1.3rem;
      margin-bottom: 1rem;
      min-height: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    .options {
      display: flex;
      flex-direction: column;
      margin-bottom: 1rem;
    }
    .option {
      background: #2c3e50;
      color: white;
      padding: 0.75rem;
      margin: 0.3rem 0;
      border-radius: 10px;
      cursor: pointer;
      transition: background 0.3s ease, transform 0.1s ease;
      border: 1px solid #2c3e50;
      text-align: center;
    }
    .option:hover {
      background: #34495e;
      border-color: #f1c40f;
    }
    .option.correct {
      background: #27ae60;
      border-color: #27ae60;
    }
    .option.incorrect {
      background: #e74c3c;
      border-color: #e74c3c;
    }
    .score, .result {
      margin-top: 1rem;
      font-weight: bold;
    }
    .result {
      font-size: 1.8rem;
      color: #2ecc71;
    }
    .restart {
      background: linear-gradient(to right, #ff416c, #ff4b2b);
      margin-top: 1.5rem;
    }
    #timer {
      font-size: 1.8rem;
      font-weight: bold;
      margin-bottom: 1.5rem;
      color: #f1c40f;
      display: none;
    }
    .message-box {
      background-color: #38b2ac;
      color: #e2e8f0;
      padding: 15px;
      border-radius: 8px;
      margin-top: 20px;
      display: none;
      font-size: 1rem;
      font-weight: 500;
    }
    .message-box.error {
      background-color: #e53e3e;
    }
    .message-box.success {
      background-color: #48bb78;
    }
    .sound-control {
      position: absolute;
      top: 15px;
      right: 15px;
      background: rgba(255, 255, 255, 0.2);
      border-radius: 50%;
      width: 40px;
      height: 40px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      z-index: 10;
    }
    .sound-control i {
      font-size: 20px;
    }
    .sound-control.muted {
      background: rgba(231, 76, 60, 0.5);
    }
  </style>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body>
  <div class="quiz-box" id="quiz">
    <div class="sound-control" id="soundControl" title="Toggle sound">
      <i class="fas fa-volume-up"></i>
    </div>
    <div id="setup">
      <h2>🎯 Choose Topic & Difficulty</h2>
      <select id="topicSelect">
        <option value="general">🌍 General</option>
        <option value="science">🔬 Science</option>
        <option value="math">➗ Math</option>
        <option value="coding" disabled>💻 Coding (Coming Soon)</option>
      </select>
      <select id="difficultySelect">
        <option value="easy">😀 Easy</option>
        <option value="medium">😎 Medium</option>
        <option value="hard">🤯 Hard</option>
      </select>
      <button onclick="startQuiz()">🚀 Start Quiz</button>
    </div>

    <div id="timer"></div>
    <div class="question" id="question" style="display:none"></div>
    <div class="options" id="options" style="display:none"></div>
    <div class="score" id="score" style="display:none"></div>
    <div id="messageBox" class="message-box"></div>
  </div>

  <script>
    const correctSound = new Audio("https://www.soundjay.com/buttons/sounds/button-3.mp3");
    const wrongSound = new Audio("https://www.soundjay.com/buttons/sounds/button-10.mp3");
    const timerEndSound = new Audio("https://www.soundjay.com/buttons/sounds/button-09.mp3");

    correctSound.preload = 'auto';
    wrongSound.preload = 'auto';
    timerEndSound.preload = 'auto';

    let soundEnabled = true;

    const allQuestions = { /* insert your existing question pools here */ };

    let currentQuestionIndex = 0;
    let score = 0;
    let selectedQuestions = [];
    let timerInterval;
    const timePerQuestion = 15;

    const setupDiv = document.getElementById("setup");
    const questionEl = document.getElementById("question");
    const optionsEl = document.getElementById("options");
    const scoreEl = document.getElementById("score");
    const timerEl = document.getElementById("timer");
    const messageBox = document.getElementById("messageBox");
    const soundControl = document.getElementById("soundControl");

    function playSound(sound) {
      if (!soundEnabled) return;
      try {
        sound.pause();
        sound.currentTime = 0;
        sound.play().catch(() => {});
      } catch {}
    }

    soundControl.addEventListener("click", () => {
      soundEnabled = !soundEnabled;
      soundControl.classList.toggle("muted", !soundEnabled);
      soundControl.innerHTML = `<i class="fas fa-volume-${soundEnabled ? "up" : "mute"}"></i>`;
      showMessage(`Sound ${soundEnabled ? "enabled" : "disabled"}`, soundEnabled ? "success" : "error");
    });

    function showMessage(message, type = 'info') {
      messageBox.textContent = message;
      messageBox.className = 'message-box';
      if (type === 'error') messageBox.classList.add('error');
      else if (type === 'success') messageBox.classList.add('success');
      messageBox.style.display = 'block';
      setTimeout(() => messageBox.style.display = 'none', 3000);
    }

    function shuffleArray(arr) {
      for (let i = arr.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [arr[i], arr[j]] = [arr[j], arr[i]];
      }
      return arr;
    }

    function startQuiz() {
      const topic = document.getElementById("topicSelect").value;
      const difficulty = document.getElementById("difficultySelect").value;
      let questions = allQuestions[topic][difficulty];

      if (!questions || questions.length === 0) {
        showMessage("No questions available for this topic and difficulty.", "error");
        return;
      }

      selectedQuestions = shuffleArray([...questions]).slice(0, 10);
      currentQuestionIndex = 0;
      score = 0;

      setupDiv.style.display = "none";
      timerEl.style.display = "block";
      questionEl.style.display = "flex";
      optionsEl.style.display = "flex";
      scoreEl.style.display = "block";
      messageBox.style.display = "none";

      showQuestion();
    }

    function showQuestion() {
      clearInterval(timerInterval);

      if (currentQuestionIndex >= selectedQuestions.length) return showResult();

      const current = selectedQuestions[currentQuestionIndex];
      questionEl.textContent = current.question;
      optionsEl.innerHTML = "";

      current.options.forEach(opt => {
        const btn = document.createElement("button");
        btn.className = "option";
        btn.textContent = opt;
        btn.onclick = () => selectOption(opt);
        optionsEl.appendChild(btn);
      });

      scoreEl.textContent = `Question ${currentQuestionIndex + 1} of ${selectedQuestions.length} | Score: ${score}`;
      startTimer();
    }

    function startTimer() {
      let timeLeft = timePerQuestion;
      timerEl.textContent = `⏱️ ${timeLeft}s`;
      timerInterval = setInterval(() => {
        timeLeft--;
        timerEl.textContent = `⏱️ ${timeLeft}s`;
        if (timeLeft <= 0) {
          clearInterval(timerInterval);
          playSound(timerEndSound);
          selectOption(null);
        }
      }, 1000);
    }

    function selectOption(selected) {
      clearInterval(timerInterval);

      const current = selectedQuestions[currentQuestionIndex];
      const buttons = optionsEl.querySelectorAll(".option");

      buttons.forEach(btn => {
        if (btn.textContent === current.answer) btn.classList.add("correct");
        else if (btn.textContent === selected) btn.classList.add("incorrect");
        btn.disabled = true;
      });

      if (selected === current.answer) {
        score++;
        playSound(correctSound);
      } else {
        playSound(wrongSound);
      }

      setTimeout(() => {
        currentQuestionIndex++;
        showQuestion();
      }, 1200);
    }

    function showResult() {
      questionEl.textContent = `🎉 Final Score: ${score}/${selectedQuestions.length}`;
      optionsEl.innerHTML = "";
      timerEl.style.display = "none";
      scoreEl.style.display = "none";

      const restartBtn = document.createElement("button");
      restartBtn.textContent = "🔁 Restart Quiz";
      restartBtn.className = "restart";
      restartBtn.onclick = () => location.reload();
      optionsEl.appendChild(restartBtn);
    }
  </script>
</body>
</html>
