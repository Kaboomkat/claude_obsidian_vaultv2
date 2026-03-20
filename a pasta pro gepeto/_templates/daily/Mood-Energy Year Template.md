---
title: "Tracker <% tp.file.title %>"
type: mood-tracker
frequency: yearly
year: <% tp.file.title %>
tags:
  - tracker/yearly
  - tracker/mood
---

# 📊 Tracker Anual de Humor & Energia

```dataviewjs
// ─── Configuração ─────────────────────────────────────────────────────────
const cur  = dv.current();
const year = parseInt(cur.year) || new Date().getFullYear();

const monthNames = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho",
                    "Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];

// ─── Gradiente de cor ─────────────────────────────────────────────────────
function moodColor(value) {
    if (!value && value !== 0) return "#1e1e1e";
    const match = String(value).match(/^(\d+(?:\.\d+)?)/);
    if (!match) return "#1e1e1e";
    const v = Math.max(1, Math.min(10, parseFloat(match[1])));
    const t = (v - 1) / 9;
    const r = Math.round(169 + (15  - 169) * t);
    const g = Math.round(59  + (93  - 59)  * t);
    const b = Math.round(22  + (53  - 22)  * t);
    return `#${r.toString(16).padStart(2,'0')}${g.toString(16).padStart(2,'0')}${b.toString(16).padStart(2,'0')}`;
}

function parseVal(v) {
    if (!v) return null;
    const m = String(v).match(/^(\d+(?:\.\d+)?)/);
    return m ? parseFloat(m[1]) : null;
}

// ─── Buscar reviews do ano ────────────────────────────────────────────────
const allMornings = dv.pages('"03-Daily/Morning Reviews"')
    .where(p => p.date && dv.date(p.date) && dv.date(p.date).year === year)
    .sort(p => p.date, 'asc');
const allEvenings = dv.pages('"03-Daily/Evening Reviews"')
    .where(p => p.date && dv.date(p.date) && dv.date(p.date).year === year)
    .sort(p => p.date, 'asc');

// Maps por data (YYYY-MM-DD)
const morningMap = {};
allMornings.forEach(p => { if (p.date) morningMap[dv.date(p.date).toFormat("yyyy-MM-dd")] = p; });
const eveningMap = {};
allEvenings.forEach(p => { if (p.date) eveningMap[dv.date(p.date).toFormat("yyyy-MM-dd")] = p; });

// ─── Agregar por mês ──────────────────────────────────────────────────────
const byMonth = {};
for (let m = 1; m <= 12; m++) byMonth[m] = { mm:[], em:[], mn:[], en:[], count:0 };

allMornings.forEach(p => {
    const d = dv.date(p.date); if (!d) return;
    const m = d.month;
    const mm = parseVal(p.mood_morning);   if (mm) byMonth[m].mm.push(mm);
    const em = parseVal(p.energy_morning); if (em) byMonth[m].em.push(em);
    byMonth[m].count++;
});
allEvenings.forEach(p => {
    const d = dv.date(p.date); if (!d) return;
    const m = d.month;
    const mn = parseVal(p.mood_evening);   if (mn) byMonth[m].mn.push(mn);
    const en = parseVal(p.energy_evening); if (en) byMonth[m].en.push(en);
});

const avg = arr => arr.length ? (arr.reduce((a,b)=>a+b,0)/arr.length) : null;
const fmtAvg = arr => { const a = avg(arr); return a ? a.toFixed(1) : "—"; };

// ─── Estatísticas anuais ──────────────────────────────────────────────────
let allMm=[], allEm=[], allMn=[], allEn=[];
allMornings.forEach(p => {
    const mm = parseVal(p.mood_morning);   if (mm) allMm.push(mm);
    const em = parseVal(p.energy_morning); if (em) allEm.push(em);
});
allEvenings.forEach(p => {
    const mn = parseVal(p.mood_evening);   if (mn) allMn.push(mn);
    const en = parseVal(p.energy_evening); if (en) allEn.push(en);
});

// Melhor e pior mês (por média de humor)
let bestMonth = null, worstMonth = null, bestVal = -Infinity, worstVal = Infinity;
for (let m = 1; m <= 12; m++) {
    const a = avg([...byMonth[m].mm, ...byMonth[m].mn]);
    if (a !== null) {
        if (a > bestVal)  { bestVal = a;  bestMonth = m; }
        if (a < worstVal) { worstVal = a; worstMonth = m; }
    }
}

// Contar dias únicos registrados
const totalDaysLogged = new Set([...Object.keys(morningMap), ...Object.keys(eveningMap)]).size;

// ─── Cards dos meses ──────────────────────────────────────────────────────
let monthCards = "";
for (let m = 1; m <= 12; m++) {
    const data = byMonth[m];
    const moodAvg = avg([...data.mm, ...data.mn]);
    const color = moodColor(moodAvg ? moodAvg.toFixed(1) + "/10" : null);
    const entries = data.count || byMonth[m].mn.length;
    const mStr = String(m).padStart(2, '0');

    monthCards += `
    <div style="background:${color};border-radius:10px;padding:12px;text-align:center;min-height:80px;display:flex;flex-direction:column;justify-content:space-between;">
        <div style="font-weight:bold;color:#fff;font-size:0.9em;"><a href="00-Dashboard/Mood-Energy Tracker/Monthly/${year}/${mStr}" class="internal-link" style="color:#fff;text-decoration:none;">${monthNames[m-1]}</a></div>
        <div style="font-size:1.3em;font-weight:bold;color:#fff;">${moodAvg ? moodAvg.toFixed(1) : "—"}</div>
        <div style="font-size:0.7em;color:rgba(255,255,255,0.7);">${Math.max(data.mm.length, data.mn.length)} entradas</div>
    </div>`;
}

// ─── Heatmap anual (365 pontos) ───────────────────────────────────────────
const isLeap = (year % 4 === 0 && year % 100 !== 0) || year % 400 === 0;
const totalDays = isLeap ? 366 : 365;

const startDate = new Date(year, 0, 1);
const startDow = startDate.getDay();
const heatmapOffset = (startDow === 0) ? 6 : startDow - 1;

let dots = "";
for (let i = 0; i < heatmapOffset; i++) dots += `<div style="width:10px;height:10px;"></div>`;
for (let i = 0; i < totalDays; i++) {
    const d = new Date(year, 0, i + 1);
    const dateStr = `${year}-${String(d.getMonth()+1).padStart(2,'0')}-${String(d.getDate()).padStart(2,'0')}`;
    const mp = morningMap[dateStr];
    const ep = eveningMap[dateStr];
    const mm = mp ? parseVal(mp.mood_morning) : null;
    const mn = ep ? parseVal(ep.mood_evening)  : null;
    const mood = (mm && mn) ? (mm + mn) / 2 : (mm || mn || null);
    const color = mood ? moodColor(mood.toFixed(1)+"/10") : "#1e1e1e";
    const border = mood ? "" : "border:1px solid #333;";
    const title = `title="${dateStr}${mood ? ': ' + mood.toFixed(1) + '/10' : ''}"`;
    dots += `<div ${title} style="width:10px;height:10px;border-radius:2px;background:${color};${border}"></div>`;
}

// ─── HTML final ────────────────────────────────────────────────────────────
dv.container.innerHTML = `
<h2>${year}</h2>
<div style="color:#888;font-size:0.85em;margin-bottom:20px;">${totalDaysLogged} dias registrados</div>

<!-- Cards dos meses -->
<div style="display:grid;grid-template-columns:repeat(4,1fr);gap:10px;margin-bottom:24px;">
${monthCards}
</div>

<!-- Estatísticas anuais -->
<div style="display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:24px;">
    <div style="background:${moodColor(fmtAvg(allMm)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.7em;color:#eee;">😌 Humor Manhã</div>
        <div style="font-size:1.5em;font-weight:bold;color:#fff;">${fmtAvg(allMm)}</div>
    </div>
    <div style="background:${moodColor(fmtAvg(allEm)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.7em;color:#eee;">⚡ Energia Manhã</div>
        <div style="font-size:1.5em;font-weight:bold;color:#fff;">${fmtAvg(allEm)}</div>
    </div>
    <div style="background:${moodColor(fmtAvg(allMn)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.7em;color:#eee;">😌 Humor Noite</div>
        <div style="font-size:1.5em;font-weight:bold;color:#fff;">${fmtAvg(allMn)}</div>
    </div>
    <div style="background:${moodColor(fmtAvg(allEn)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.7em;color:#eee;">⚡ Energia Noite</div>
        <div style="font-size:1.5em;font-weight:bold;color:#fff;">${fmtAvg(allEn)}</div>
    </div>
</div>

<!-- Melhor / Pior mês -->
<div style="display:flex;gap:12px;margin-bottom:24px;">
    <div style="flex:1;background:#1a2a1a;border:1px solid #0F5D35;border-radius:8px;padding:12px;text-align:center;">
        <div style="color:#0F5D35;font-size:0.8em;margin-bottom:4px;">🏆 Melhor Mês</div>
        <div style="font-weight:bold;color:#fff;">${bestMonth ? monthNames[bestMonth-1] + " (" + bestVal.toFixed(1) + ")" : "—"}</div>
    </div>
    <div style="flex:1;background:#2a1a1a;border:1px solid #A93B16;border-radius:8px;padding:12px;text-align:center;">
        <div style="color:#A93B16;font-size:0.8em;margin-bottom:4px;">📉 Mês Mais Difícil</div>
        <div style="font-weight:bold;color:#fff;">${worstMonth ? monthNames[worstMonth-1] + " (" + worstVal.toFixed(1) + ")" : "—"}</div>
    </div>
</div>

<!-- Heatmap do ano -->
<div style="margin-bottom:8px;color:#888;font-size:0.8em;">📅 Heatmap ${year}</div>
<div style="display:grid;grid-template-columns:repeat(53,10px);gap:2px;overflow-x:auto;">
${dots}
</div>
<div style="display:flex;gap:4px;align-items:center;margin-top:8px;font-size:0.75em;color:#666;">
    <div style="width:10px;height:10px;background:#1e1e1e;border:1px solid #333;border-radius:2px;"></div><span>Sem dados</span>
    <div style="width:10px;height:10px;background:#A93B16;border-radius:2px;margin-left:8px;"></div><span>1/10</span>
    <div style="width:10px;height:10px;background:#5C4C25;border-radius:2px;"></div><span>5/10</span>
    <div style="width:10px;height:10px;background:#0F5D35;border-radius:2px;"></div><span>10/10</span>
</div>`;
```

---

## 🔗 Meses

[[Monthly/<% tp.file.title %>/01|Janeiro]] · [[Monthly/<% tp.file.title %>/02|Fevereiro]] · [[Monthly/<% tp.file.title %>/03|Março]] · [[Monthly/<% tp.file.title %>/04|Abril]] · [[Monthly/<% tp.file.title %>/05|Maio]] · [[Monthly/<% tp.file.title %>/06|Junho]] · [[Monthly/<% tp.file.title %>/07|Julho]] · [[Monthly/<% tp.file.title %>/08|Agosto]] · [[Monthly/<% tp.file.title %>/09|Setembro]] · [[Monthly/<% tp.file.title %>/10|Outubro]] · [[Monthly/<% tp.file.title %>/11|Novembro]] · [[Monthly/<% tp.file.title %>/12|Dezembro]]

---

[[../Hub|⬆ Mood Tracker Hub]]
