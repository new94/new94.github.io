<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Roadmap — Диаграмма Ганта</title>
<style>
  * { box-sizing: border-box; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    margin: 0;
    padding: 20px;
    background: #f5f6fa;
    color: #2c3e50;
  }
  h1 { margin: 0 0 8px; font-size: 22px; }
  .subtitle { color: #7f8c8d; margin-bottom: 16px; font-size: 13px; }
  .toolbar {
    display: flex;
    gap: 12px;
    align-items: center;
    margin-bottom: 16px;
    flex-wrap: wrap;
    background: white;
    padding: 12px 16px;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.05);
  }
  .toolbar label { font-size: 13px; color: #555; }
  .toolbar input[type="date"], .toolbar input[type="number"] {
    padding: 6px 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 13px;
  }
  .toolbar button {
    padding: 6px 14px;
    background: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 13px;
  }
  .toolbar button:hover { background: #2980b9; }
  .toolbar button.secondary { background: #95a5a6; }
  .toolbar button.secondary:hover { background: #7f8c8d; }

  .gantt-container {
    background: white;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0,0,0,0.05);
    overflow: auto;
    max-height: calc(100vh - 200px);
    position: relative;
  }

  table.gantt {
    border-collapse: collapse;
    width: max-content;
    min-width: 100%;
    font-size: 12px;
  }
  table.gantt th, table.gantt td {
    border: 1px solid #ecf0f1;
    padding: 0;
    text-align: center;
    vertical-align: middle;
  }
  table.gantt thead th {
    background: #34495e;
    color: white;
    padding: 4px 4px;
    font-weight: 500;
    position: sticky;
    top: 0;
    z-index: 3;
    font-size: 11px;
  }
  table.gantt thead th.month-header {
    background: #2c3e50;
    font-size: 12px;
  }
  table.gantt .day-cell { width: 28px; min-width: 28px; }
  table.gantt .day-cell.weekend { background: #f0f0f3; }
  table.gantt .day-cell.today { background: #fff3cd; }

  table.gantt th.col-num, table.gantt td.col-num {
    position: sticky; left: 0; z-index: 2;
    background: #34495e; color: white; min-width: 36px; width: 36px;
  }
  table.gantt tbody td.col-num { background: #ecf0f1; color: #2c3e50; }
  table.gantt th.col-task, table.gantt td.col-task {
    position: sticky; left: 36px; z-index: 2;
    background: #34495e; color: white; min-width: 280px; width: 280px;
    text-align: left; padding-left: 8px;
  }
  table.gantt tbody td.col-task { background: white; color: #2c3e50; }
  table.gantt th.col-start, table.gantt td.col-start,
  table.gantt th.col-dur, table.gantt td.col-dur,
  table.gantt th.col-end, table.gantt td.col-end {
    min-width: 100px; width: 100px;
    background: #34495e; color: white;
  }
  table.gantt th.col-start { position: sticky; left: 316px; z-index: 2; }
  table.gantt th.col-dur { position: sticky; left: 416px; z-index: 2; }
  table.gantt th.col-end { position: sticky; left: 516px; z-index: 2; }
  table.gantt tbody td.col-start { position: sticky; left: 316px; z-index: 1; background: white; color: #2c3e50; }
  table.gantt tbody td.col-dur   { position: sticky; left: 416px; z-index: 1; background: white; color: #2c3e50; }
  table.gantt tbody td.col-end   { position: sticky; left: 516px; z-index: 1; background: white; color: #2c3e50; }

  table.gantt tbody tr:hover td:not(.bar-cell) { background: #f1f7fb !important; }

  table.gantt tbody tr.dragging { opacity: 0.4; }
  table.gantt tbody tr.drag-over-top td { border-top: 2px solid #e74c3c; }
  table.gantt tbody tr.drag-over-bottom td { border-bottom: 2px solid #e74c3c; }

  .drag-handle {
    cursor: grab; padding: 4px 6px; color: #95a5a6;
    display: inline-block; user-select: none;
  }
  .drag-handle:active { cursor: grabbing; }

  .task-name {
    border: none; background: transparent;
    width: calc(100% - 30px);
    font-size: 12px; padding: 4px;
    font-family: inherit; color: inherit;
  }
  .task-name:focus { outline: 1px solid #3498db; background: white; }

  .date-input, .dur-input {
    border: none; background: transparent;
    width: 100%; height: 100%;
    padding: 4px; font-size: 11px;
    text-align: center; color: inherit;
    font-family: inherit;
  }
  .date-input:focus, .dur-input:focus { outline: 1px solid #3498db; background: white; }
  .end-date { font-size: 11px; color: #7f8c8d; padding: 4px; }

  td.bar-cell { position: relative; height: 32px; }
  .bar-wrapper {
    position: absolute;
    top: 4px; bottom: 4px;
    display: flex;
    align-items: center;
    cursor: move;
    z-index: 1;
  }
  .bar-segment {
    height: 100%;
    color: white;
    font-size: 11px;
    display: flex;
    align-items: center;
    white-space: nowrap;
    overflow: hidden;
    box-shadow: 0 1px 2px rgba(0,0,0,0.15);
  }
  .bar-segment.start { border-top-left-radius: 3px; border-bottom-left-radius: 3px; padding-left: 6px; }
  .bar-segment.end   { border-top-right-radius: 3px; border-bottom-right-radius: 3px; padding-right: 6px; }
  .bar-segment.gap { opacity: 0.35; }

  .bar-wrapper.color-0 .bar-segment { background: linear-gradient(180deg, #5dade2, #3498db); }
  .bar-wrapper.color-1 .bar-segment { background: linear-gradient(180deg, #58d68d, #27ae60); }
  .bar-wrapper.color-2 .bar-segment { background: linear-gradient(180deg, #f5b041, #e67e22); }
  .bar-wrapper.color-3 .bar-segment { background: linear-gradient(180deg, #af7ac5, #8e44ad); }
  .bar-wrapper.color-4 .bar-segment { background: linear-gradient(180deg, #ec7063, #c0392b); }
  .bar-wrapper.color-5 .bar-segment { background: linear-gradient(180deg, #5499c7, #2874a6); }

  .bar-wrapper .resize-handle {
    position: absolute; top: 0; bottom: 0; width: 6px;
    cursor: ew-resize; z-index: 2;
  }
  .bar-wrapper .resize-left { left: 0; }
  .bar-wrapper .resize-right { right: 0; }

  .delete-btn {
    background: none; border: none; color: #e74c3c;
    cursor: pointer; font-size: 14px; padding: 0 4px;
  }
  .delete-btn:hover { color: #c0392b; }

  .stats {
    display: flex; gap: 24px; margin-left: auto;
    font-size: 13px; color: #555;
  }
  .stats strong { color: #2c3e50; }

  .legend { font-size: 12px; color: #7f8c8d; display: flex; gap: 16px; align-items: center; }
  .legend-box { display: inline-block; width: 12px; height: 12px; border-radius: 2px; vertical-align: middle; margin-right: 4px; }
</style>
</head>
<body>

<h1>📅 Roadmap — Диаграмма Ганта</h1>
<div class="subtitle">Длительность указана в <strong>рабочих днях</strong>. Выходные (сб/вс) исключаются из подсчёта и подсвечены серым.</div>

<div class="toolbar">
  <label>Начало проекта: <input type="date" id="projectStart"></label>
  <button id="addTaskBtn">+ Добавить задачу</button>
  <button id="resetBtn" class="secondary">Сбросить</button>
  <button id="exportBtn" class="secondary">Экспорт JSON</button>
  <div class="legend">
    <span><span class="legend-box" style="background:#f0f0f3"></span>Выходной</span>
    <span><span class="legend-box" style="background:#fff3cd"></span>Сегодня</span>
  </div>
  <div class="stats">
    <span>Задач: <strong id="taskCount">0</strong></span>
    <span>Раб. дней: <strong id="totalDays">0</strong></span>
    <span>Окончание: <strong id="projectEnd">—</strong></span>
  </div>
</div>

<div class="gantt-container">
  <table class="gantt" id="ganttTable">
    <thead>
      <tr id="monthRow"></tr>
      <tr id="dayRow"></tr>
    </thead>
    <tbody id="ganttBody"></tbody>
  </table>
</div>

<script>
// ============ ДАННЫЕ ============
const DEFAULT_TASKS = [
  { name: "Сервис аутентификации", duration: 3 },
  { name: "Сервис аудита", duration: 3 },
  { name: "Ad-hoc сервис НРД", duration: 3 },
  { name: "Backend сервис STG НРД", duration: 3 },
  { name: "Загрузчик CBonds в RAW", duration: 3 },
  { name: "Backend сервис RAW CBonds", duration: 3 },
  { name: "Backend сервис STG CBonds", duration: 3 },
  { name: "Ad-hoc сервис CBonds", duration: 3 },
  { name: "Загрузчик MOEX в RAW", duration: 3 },
  { name: "Backend сервис RAW MOEX", duration: 3 },
  { name: "Backend сервис STG MOEX", duration: 3 },
  { name: "Ad-hoc сервис MOEX", duration: 3 },
  { name: "Сервис запросов Bloomberg через BizTalk", duration: 7 },
  { name: "Сервис загрузки Bloomberg из BizTalk в RAW", duration: 14 },
  { name: "Сервис Backend RAW Bloomberg", duration: 3 },
  { name: "Сервис Backend STG Bloomberg", duration: 3 },
  { name: "Ad-hoc сервис Bloomberg через BizTalk", duration: 14 },
  { name: "Frontend", duration: 7 },
  { name: "Сервис ODS", duration: 14 },
  { name: "Сервис DMA (единые цены)", duration: 14 },
  { name: "Сервис репликатор в MS SQL/GreenPlum", duration: 14 },
  { name: "Сервис публикатор Kafka", duration: 7 },
  { name: "Мониторинг", duration: 3 },
  { name: "Калькулятор аллокации", duration: 7 },
  { name: "Калькулятор предварительной стоимости", duration: 3 },
  { name: "Сервис аудита неактуальных запросов", duration: 7 }
];

let tasks = [];
let projectStartDate;

// ============ УТИЛИТЫ ДАТ ============
function fmtDateISO(d) {
  const y = d.getFullYear(), m = String(d.getMonth()+1).padStart(2,'0'), day = String(d.getDate()).padStart(2,'0');
  return `${y}-${m}-${day}`;
}
function fmtDateShort(d) {
  return `${String(d.getDate()).padStart(2,'0')}.${String(d.getMonth()+1).padStart(2,'0')}.${String(d.getFullYear()).slice(2)}`;
}
function parseDate(iso) {
  const [y,m,d] = iso.split('-').map(Number);
  return new Date(y, m-1, d);
}
function addDays(d, n) {
  const r = new Date(d); r.setDate(r.getDate()+n); return r;
}
function diffDays(a, b) {
  return Math.round((b - a) / (1000*60*60*24));
}
function isWeekend(d) {
  const w = d.getDay(); return w === 0 || w === 6;
}
function sameDay(a, b) {
  return a.getFullYear()===b.getFullYear() && a.getMonth()===b.getMonth() && a.getDate()===b.getDate();
}

// ============ РАБОЧИЕ ДНИ ============
function toNextWorkday(d) {
  const r = new Date(d);
  while (isWeekend(r)) r.setDate(r.getDate()+1);
  return r;
}
// Получить дату последнего рабочего дня задачи (inclusive)
function getEndWorkday(startDate, workdays) {
  let d = toNextWorkday(startDate);
  let counted = 1;
  while (counted < workdays) {
    d = addDays(d, 1);
    if (!isWeekend(d)) counted++;
  }
  return d;
}
// Сколько календарных дней занимает полоска от старта до конца включительно
function getCalendarSpan(startDate, workdays) {
  const start = toNextWorkday(startDate);
  const end = getEndWorkday(start, workdays);
  return diffDays(start, end) + 1;
}
// Подсчитать рабочие дни в полуинтервале [from, to)
function countWorkdays(from, to) {
  let count = 0;
  let d = new Date(from);
  while (d < to) {
    if (!isWeekend(d)) count++;
    d = addDays(d, 1);
  }
  return count;
}

// ============ ИНИЦИАЛИЗАЦИЯ ============
function initTasks() {
  let cursor = toNextWorkday(new Date(projectStartDate));
  tasks = DEFAULT_TASKS.map((t, i) => {
    const start = toNextWorkday(cursor);
    const lastDay = getEndWorkday(start, t.duration);
    cursor = toNextWorkday(addDays(lastDay, 1));
    return {
      id: 't' + (i+1) + '_' + Date.now() + '_' + i,
      name: t.name,
      start: fmtDateISO(start),
      duration: t.duration
    };
  });
}

function loadState() {
  const saved = localStorage.getItem('ganttRoadmapWD');
  if (saved) {
    try {
      const data = JSON.parse(saved);
      projectStartDate = parseDate(data.projectStart);
      tasks = data.tasks;
      return true;
    } catch(e) {}
  }
  return false;
}

function saveState() {
  localStorage.setItem('ganttRoadmapWD', JSON.stringify({
    projectStart: fmtDateISO(projectStartDate),
    tasks: tasks
  }));
}

// ============ РЕНДЕРИНГ ============
function calcDateRange() {
  let minDate = projectStartDate;
  let maxDate = projectStartDate;
  tasks.forEach(t => {
    const s = parseDate(t.start);
    const e = getEndWorkday(s, t.duration);
    if (s < minDate) minDate = s;
    if (e > maxDate) maxDate = e;
  });
  minDate = addDays(minDate, -2);
  maxDate = addDays(maxDate, 5);
  return { minDate, maxDate };
}

function render() {
  tasks.forEach(t => {
    const s = parseDate(t.start);
    if (isWeekend(s)) t.start = fmtDateISO(toNextWorkday(s));
  });

  const { minDate, maxDate } = calcDateRange();
  const totalDays = diffDays(minDate, maxDate);
  const today = new Date(); today.setHours(0,0,0,0);

  const monthRow = document.getElementById('monthRow');
  const dayRow = document.getElementById('dayRow');
  monthRow.innerHTML = '';
  dayRow.innerHTML = '';

  const fixedHead = `
    <th class="col-num" rowspan="2">#</th>
    <th class="col-task" rowspan="2">Задача</th>
    <th class="col-start" rowspan="2">Начало</th>
    <th class="col-dur" rowspan="2">Раб. дн.</th>
    <th class="col-end" rowspan="2">Окончание</th>
  `;
  monthRow.innerHTML = fixedHead;

  const months = [];
  let cur = new Date(minDate);
  for (let i = 0; i < totalDays; i++) {
    const key = `${cur.getFullYear()}-${cur.getMonth()}`;
    if (!months.length || months[months.length-1].key !== key) {
      const monthNames = ['Январь','Февраль','Март','Апрель','Май','Июнь','Июль','Август','Сентябрь','Октябрь','Ноябрь','Декабрь'];
      months.push({ key, label: `${monthNames[cur.getMonth()]} ${cur.getFullYear()}`, count: 1 });
    } else {
      months[months.length-1].count++;
    }
    cur = addDays(cur, 1);
  }
  months.forEach(m => {
    monthRow.innerHTML += `<th class="month-header" colspan="${m.count}">${m.label}</th>`;
  });

  cur = new Date(minDate);
  const dayLetters = ['В','П','В','С','Ч','П','С'];
  for (let i = 0; i < totalDays; i++) {
    const isWE = isWeekend(cur);
    const isToday = sameDay(cur, today);
    const cls = `day-cell ${isWE ? 'weekend' : ''} ${isToday ? 'today' : ''}`;
    dayRow.innerHTML += `<th class="${cls}" title="${fmtDateShort(cur)}"><div>${cur.getDate()}</div><div style="font-size:9px;opacity:0.7">${dayLetters[cur.getDay()]}</div></th>`;
    cur = addDays(cur, 1);
  }

  const body = document.getElementById('ganttBody');
  body.innerHTML = '';

  tasks.forEach((task, idx) => {
    const tr = document.createElement('tr');
    tr.dataset.taskId = task.id;
    tr.draggable = false;

    const startD = parseDate(task.start);
    const endD = getEndWorkday(startD, task.duration);
    const offset = diffDays(minDate, startD);
    const span = getCalendarSpan(startD, task.duration);

    tr.innerHTML = `
      <td class="col-num">
        <span class="drag-handle" title="Перетащить">⋮⋮</span>
        <span>${idx+1}</span>
      </td>
      <td class="col-task">
        <input class="task-name" type="text" value="${escapeHtml(task.name)}" data-id="${task.id}" data-field="name">
        <button class="delete-btn" data-id="${task.id}" title="Удалить">✕</button>
      </td>
      <td class="col-start">
        <input class="date-input" type="date" value="${task.start}" data-id="${task.id}" data-field="start">
      </td>
      <td class="col-dur">
        <input class="dur-input" type="number" min="1" max="365" value="${task.duration}" data-id="${task.id}" data-field="duration">
      </td>
      <td class="col-end">
        <span class="end-date">${fmtDateShort(endD)}</span>
      </td>
    `;

    let dayCur = new Date(minDate);
    for (let i = 0; i < totalDays; i++) {
      const isWE = isWeekend(dayCur);
      const td = document.createElement('td');
      td.className = `bar-cell day-cell ${isWE ? 'weekend' : ''}`;
      if (i === offset) {
        const wrapper = document.createElement('div');
        wrapper.className = `bar-wrapper color-${idx % 6}`;
        wrapper.style.left = '2px';
        wrapper.style.width = `calc(${span * 28}px - 4px)`;
        wrapper.dataset.id = task.id;

        // Сегменты: разбиваем по сменам рабочий <-> выходной
        const segs = [];
        let segStart = 0;
        let segIsGap = isWeekend(startD);
        for (let j = 1; j < span; j++) {
          const dd = addDays(startD, j);
          const gap = isWeekend(dd);
          if (gap !== segIsGap) {
            segs.push({ from: segStart, to: j, gap: segIsGap });
            segStart = j;
            segIsGap = gap;
          }
        }
        segs.push({ from: segStart, to: span, gap: segIsGap });

        const innerHTML = segs.map((s, si) => {
          const w = (s.to - s.from) * 28;
          const isFirst = si === 0;
          const isLast = si === segs.length - 1;
          const cls = `bar-segment ${s.gap ? 'gap' : ''} ${isFirst ? 'start' : ''} ${isLast ? 'end' : ''}`;
          const label = isFirst && !s.gap ? `${escapeHtml(task.name)} (${task.duration} раб.дн.)` : '';
          return `<div class="${cls}" style="width:${w}px">${label}</div>`;
        }).join('');

        wrapper.innerHTML = `
          <div class="resize-handle resize-left"></div>
          ${innerHTML}
          <div class="resize-handle resize-right"></div>
        `;
        td.appendChild(wrapper);
      }
      tr.appendChild(td);
      dayCur = addDays(dayCur, 1);
    }

    body.appendChild(tr);
  });

  attachHandlers();
  updateStats();
  saveState();
}

function escapeHtml(s) {
  return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
}

function updateStats() {
  document.getElementById('taskCount').textContent = tasks.length;
  let maxEnd = projectStartDate;
  tasks.forEach(t => {
    const e = getEndWorkday(parseDate(t.start), t.duration);
    if (e > maxEnd) maxEnd = e;
  });
  const workdays = countWorkdays(projectStartDate, addDays(maxEnd, 1));
  document.getElementById('totalDays').textContent = workdays;
  document.getElementById('projectEnd').textContent = fmtDateShort(maxEnd);
}

// ============ ОБРАБОТЧИКИ ============
function attachHandlers() {
  document.querySelectorAll('.task-name, .date-input, .dur-input').forEach(inp => {
    inp.addEventListener('change', e => {
      const id = e.target.dataset.id;
      const field = e.target.dataset.field;
      const task = tasks.find(t => t.id === id);
      if (!task) return;
      if (field === 'duration') task.duration = Math.max(1, parseInt(e.target.value) || 1);
      else if (field === 'start') {
        const d = parseDate(e.target.value);
        task.start = fmtDateISO(toNextWorkday(d));
      }
      else task[field] = e.target.value;
      render();
    });
  });

  document.querySelectorAll('.delete-btn').forEach(btn => {
    btn.addEventListener('click', e => {
      const id = e.target.dataset.id;
      tasks = tasks.filter(t => t.id !== id);
      render();
    });
  });

  document.querySelectorAll('.drag-handle').forEach(handle => {
    handle.addEventListener('mousedown', () => {
      const tr = handle.closest('tr');
      tr.draggable = true;
    });
    handle.closest('tr').addEventListener('dragend', e => {
      e.currentTarget.draggable = false;
    });
  });

  let dragSrcId = null;
  document.querySelectorAll('#ganttBody tr').forEach(tr => {
    tr.addEventListener('dragstart', e => {
      dragSrcId = tr.dataset.taskId;
      tr.classList.add('dragging');
      e.dataTransfer.effectAllowed = 'move';
    });
    tr.addEventListener('dragend', () => {
      tr.classList.remove('dragging');
      document.querySelectorAll('#ganttBody tr').forEach(r => {
        r.classList.remove('drag-over-top', 'drag-over-bottom');
      });
    });
    tr.addEventListener('dragover', e => {
      e.preventDefault();
      if (tr.dataset.taskId === dragSrcId) return;
      const rect = tr.getBoundingClientRect();
      const mid = rect.top + rect.height / 2;
      tr.classList.remove('drag-over-top', 'drag-over-bottom');
      if (e.clientY < mid) tr.classList.add('drag-over-top');
      else tr.classList.add('drag-over-bottom');
    });
    tr.addEventListener('dragleave', () => {
      tr.classList.remove('drag-over-top', 'drag-over-bottom');
    });
    tr.addEventListener('drop', e => {
      e.preventDefault();
      const targetId = tr.dataset.taskId;
      if (!dragSrcId || targetId === dragSrcId) return;
      const rect = tr.getBoundingClientRect();
      const after = e.clientY > rect.top + rect.height / 2;
      const srcIdx = tasks.findIndex(t => t.id === dragSrcId);
      const [moved] = tasks.splice(srcIdx, 1);
      const newIdx = tasks.findIndex(t => t.id === targetId);
      tasks.splice(after ? newIdx + 1 : newIdx, 0, moved);
      render();
    });
  });

  document.querySelectorAll('.bar-wrapper').forEach(bar => {
    const id = bar.dataset.id;

    bar.addEventListener('mousedown', e => {
      if (e.target.classList.contains('resize-handle')) return;
      e.preventDefault();
      const startX = e.clientX;
      const task = tasks.find(t => t.id === id);
      const origStart = parseDate(task.start);

      function onMove(ev) {
        const dx = ev.clientX - startX;
        const dayShift = Math.round(dx / 28);
        const newStart = toNextWorkday(addDays(origStart, dayShift));
        task.start = fmtDateISO(newStart);
        bar.style.transform = `translateX(${dayShift * 28}px)`;
      }
      function onUp() {
        document.removeEventListener('mousemove', onMove);
        document.removeEventListener('mouseup', onUp);
        render();
      }
      document.addEventListener('mousemove', onMove);
      document.addEventListener('mouseup', onUp);
    });

    // Resize right — растягиваем по календарным ячейкам, пересчитываем рабочие дни
    bar.querySelector('.resize-right').addEventListener('mousedown', e => {
      e.preventDefault();
      e.stopPropagation();
      const startX = e.clientX;
      const task = tasks.find(t => t.id === id);
      const origDur = task.duration;
      const startD = parseDate(task.start);
      const origSpan = getCalendarSpan(startD, origDur);

      function onMove(ev) {
        const dx = ev.clientX - startX;
        const cellDelta = Math.round(dx / 28);
        const newSpan = Math.max(1, origSpan + cellDelta);
        let wd = 0;
        for (let i = 0; i < newSpan; i++) {
          if (!isWeekend(addDays(startD, i))) wd++;
        }
        const newDur = Math.max(1, wd);
        task.duration = newDur;
        bar.style.width = `calc(${newSpan * 28}px - 4px)`;
      }
      function onUp() {
        document.removeEventListener('mousemove', onMove);
        document.removeEventListener('mouseup', onUp);
        render();
      }
      document.addEventListener('mousemove', onMove);
      document.addEventListener('mouseup', onUp);
    });

    // Resize left — фиксируем конец, двигаем старт
    bar.querySelector('.resize-left').addEventListener('mousedown', e => {
      e.preventDefault();
      e.stopPropagation();
      const startX = e.clientX;
      const task = tasks.find(t => t.id === id);
      const origDur = task.duration;
      const origStart = parseDate(task.start);
      const origEnd = getEndWorkday(origStart, origDur);

      function onMove(ev) {
        const dx = ev.clientX - startX;
        const dayDelta = Math.round(dx / 28);
        const newStart = toNextWorkday(addDays(origStart, dayDelta));
        if (newStart > origEnd) return;
        let wd = 0;
        let d = new Date(newStart);
        while (d <= origEnd) {
          if (!isWeekend(d)) wd++;
          d = addDays(d, 1);
        }
        task.start = fmtDateISO(newStart);
        task.duration = Math.max(1, wd);
        const newSpan = diffDays(newStart, origEnd) + 1;
        bar.style.width = `calc(${newSpan * 28}px - 4px)`;
        bar.style.transform = `translateX(${diffDays(origStart, newStart) * 28}px)`;
      }
      function onUp() {
        document.removeEventListener('mousemove', onMove);
        document.removeEventListener('mouseup', onUp);
        render();
      }
      document.addEventListener('mousemove', onMove);
      document.addEventListener('mouseup', onUp);
    });
  });
}

// ============ TOOLBAR ============
document.getElementById('projectStart').addEventListener('change', e => {
  const oldStart = projectStartDate;
  const newStart = toNextWorkday(parseDate(e.target.value));
  const shift = diffDays(oldStart, newStart);
  projectStartDate = newStart;
  tasks.forEach(t => {
    t.start = fmtDateISO(toNextWorkday(addDays(parseDate(t.start), shift)));
  });
  document.getElementById('projectStart').value = fmtDateISO(projectStartDate);
  render();
});

document.getElementById('addTaskBtn').addEventListener('click', () => {
  let lastEnd = projectStartDate;
  tasks.forEach(t => {
    const e = getEndWorkday(parseDate(t.start), t.duration);
    if (e > lastEnd) lastEnd = e;
  });
  const newStart = toNextWorkday(addDays(lastEnd, 1));
  tasks.push({
    id: 'new_' + Date.now(),
    name: 'Новая задача',
    start: fmtDateISO(newStart),
    duration: 3
  });
  render();
});

document.getElementById('resetBtn').addEventListener('click', () => {
  if (confirm('Сбросить все изменения к исходному списку?')) {
    localStorage.removeItem('ganttRoadmapWD');
    projectStartDate = toNextWorkday(new Date());
    projectStartDate.setHours(0,0,0,0);
    document.getElementById('projectStart').value = fmtDateISO(projectStartDate);
    initTasks();
    render();
  }
});

document.getElementById('exportBtn').addEventListener('click', () => {
  const data = JSON.stringify({ projectStart: fmtDateISO(projectStartDate), tasks }, null, 2);
  const blob = new Blob([data], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'roadmap.json';
  a.click();
  URL.revokeObjectURL(url);
});

// ============ СТАРТ ============
if (!loadState()) {
  projectStartDate = new Date();
  projectStartDate.setHours(0,0,0,0);
  projectStartDate = toNextWorkday(projectStartDate);
  initTasks();
}
document.getElementById('projectStart').value = fmtDateISO(projectStartDate);
render();
</script>
</body>
</html>
