
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>FocusFlow - Smart Productivity</title>
<style>
:root {
  --bg: #100d1d;
  --card: #211b36;
  --purple: #a78bfa;
  --pink: #ec8bd5;
  --text: #f5f1ff;
  --muted: #a79dbd;
  --green: #7ee0b2;
}
* { box-sizing: border-box; }
body {
  margin: 0;
  padding: 30px 15px;
  min-height: 100vh;
  background: radial-gradient(circle at top right,#352052,var(--bg) 55%);
  color: var(--text);
  font-family: Arial,sans-serif;
}
.container { max-width: 1000px; margin: auto; }
header { margin-bottom: 35px; }
.logo { color: var(--purple); font-size: 22px; font-weight: bold; }
h1 { font-size: 42px; margin-bottom: 8px; }
p { color: var(--muted); }
.grid {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 20px;
}
.card {
  background: var(--card);
  border: 1px solid #382e50;
  border-radius: 20px;
  padding: 22px;
  margin-bottom: 18px;
}
input, select {
  padding: 12px;
  background: #100d1d;
  border: 1px solid #493b63;
  border-radius: 10px;
  color: white;
}
input { width: 100%; }
button {
  padding: 11px 16px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  background: linear-gradient(135deg,var(--purple),var(--pink));
  color: #1b1230;
  font-weight: bold;
}
.add { display: flex; gap: 8px; margin: 15px 0; }
.add input { flex: 1; min-width: 0; }
.task {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 13px;
  background: #ffffff08;
  border-radius: 10px;
  margin-bottom: 9px;
}
.task span { flex: 1; overflow-wrap: anywhere; }
.task.done span {
  text-decoration: line-through;
  color: var(--muted);
}
.delete { background: transparent; color: #ff9ba9; }
.stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}
.stat {
  padding: 15px;
  background: #ffffff09;
  border-radius: 12px;
}
.stat strong { display: block; font-size: 25px; }
.bar {
  height: 9px;
  background: #ffffff18;
  border-radius: 20px;
  overflow: hidden;
  margin: 12px 0;
}
.fill {
  height: 100%;
  width: 0;
  background: linear-gradient(90deg,var(--purple),var(--pink));
  transition: width .3s;
}
.timer { text-align: center; }
.clock { font-size: 52px; font-weight: bold; margin: 15px; }
.actions { display: flex; justify-content: center; gap: 10px; }
@media(max-width:700px) {
  .grid { grid-template-columns: 1fr; }
  h1 { font-size: 34px; }
  .add { flex-wrap: wrap; }
  .add input { flex-basis: 100%; }
}
</style>
</head>
<body>
<div class="container">

<header>
  <div class="logo">✦ FOCUSFLOW</div>
  <h1>Make Today Count.</h1>
  <p>Your personal productivity space.</p>
  <p id="today"></p>
</header>

<div class="grid">

<section class="card">
  <h2>Today's Tasks</h2>

  <form class="add" id="taskForm">
    <input id="taskInput"
      placeholder="Write your next task..."
      maxlength="90" required>
    <button type="submit">+ Add</button>
  </form>

  <div id="taskList"></div>
</section>

<aside>

<section class="card">
  <h2>Your Progress</h2>

  <div id="percent">0%</div>
  <div class="bar">
    <div class="fill" id="progressFill"></div>
  </div>

  <div class="stats">
    <div class="stat">
      <strong id="completed">0</strong>
      <small>Completed</small>
    </div>
    <div class="stat">
      <strong id="remaining">0</strong>
      <small>Remaining</small>
    </div>
  </div>
</section>

<section class="card timer">
  <h2>Focus Timer</h2>
  <div class="clock" id="clock">25:00</div>

  <div class="actions">
    <button id="startBtn">Start</button>
    <button id="resetBtn">Reset</button>
  </div>

  <p>25 minutes of focus. One step at a time.</p>
</section>

</aside>
</div>
</div>

<script>
const $ = id => document.getElementById(id);

const KEY = "focusflow-tasks-v1";
let tasks = [];

try {
  const saved = JSON.parse(localStorage.getItem(KEY) || "[]");
  tasks = Array.isArray(saved) ? saved : [];
} catch {
  tasks = [];
}

$("today").textContent =
  new Date().toLocaleDateString(undefined, {
    weekday: "long",
    year: "numeric",
    month: "long",
    day: "numeric"
  });

function save() {
  localStorage.setItem(KEY, JSON.stringify(tasks));
  render();
}

function render() {
  const list = $("taskList");
  list.innerHTML = "";

  if (!tasks.length) {
    list.textContent = "No tasks yet. Add your first goal!";
  }

  tasks.forEach(task => {
    const row = document.createElement("div");
    row.className = "task" + (task.done ? " done" : "");

    const check = document.createElement("input");
    check.type = "checkbox";
    check.checked = task.done;
    check.style.width = "auto";

    check.addEventListener("change", () => {
      task.done = check.checked;
      save();
    });

    const name = document.createElement("span");
    name.textContent = task.text;

    const del = document.createElement("button");
    del.className = "delete";
    del.textContent = "✕";

    del.addEventListener("click", () => {
      tasks = tasks.filter(t => t.id !== task.id);
      save();
    });

    row.append(check, name, del);
    list.append(row);
  });

  const done = tasks.filter(t => t.done).length;
  const total = tasks.length;
  const percent = total
    ? Math.round(done / total * 100)
    : 0;

  $("completed").textContent = done;
  $("remaining").textContent = total - done;
  $("percent").textContent = percent + "%";
  $("progressFill").style.width = percent + "%";
}

$("taskForm").addEventListener("submit", e => {
  e.preventDefault();

  const text = $("taskInput").value.trim();
  if (!text) return;

  tasks.unshift({
    id: Date.now() + Math.random(),
    text,
    done: false
  });

  $("taskInput").value = "";
  save();
});

// Pomodoro Timer
let seconds = 25 * 60;
let interval = null;
let running = false;

function updateClock() {
  const min = Math.floor(seconds / 60);
  const sec = seconds % 60;

  $("clock").textContent =
    String(min).padStart(2, "0") + ":" +
    String(sec).padStart(2, "0");
}

$("startBtn").addEventListener("click", () => {
  if (running) {
    clearInterval(interval);
    running = false;
    $("startBtn").textContent = "Resume";
    return;
  }

  running = true;
  $("startBtn").textContent = "Pause";

  interval = setInterval(() => {
    seconds--;
    updateClock();

    if (seconds <= 0) {
      clearInterval(interval);
      running = false;
      $("startBtn").textContent = "Start";
      alert("Great job! Focus session completed.");
    }
  }, 1000);
});

$("resetBtn").addEventListener("click", () => {
  clearInterval(interval);
  running = false;
  seconds = 25 * 60;
  updateClock();
  $("startBtn").textContent = "Start";
});

render();
updateClock();
</script>
</body>
</html>
