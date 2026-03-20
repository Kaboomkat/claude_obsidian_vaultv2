---
title: "Tracker {{year}}-{{month}}"
type: mood-tracker
frequency: monthly
year:
month:
tags:
  - tracker/monthly
  - tracker/mood
---

# 📊 Tracker Mensal de Humor & Energia

```dataviewjs
// ─── Configuração ─────────────────────────────────────────────────────────
const cur   = dv.current();
const year  = cur.year  || new Date().getFullYear();
const month = cur.month || (new Date().getMonth() + 1);

const monthNames = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho",
                    "Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
const monthName = monthNames[month - 1];

// ─── Gradiente de cor ─────────────────────────────────────────────────────
function moodColor(value) {
    if (!value && value !== 0) return "#2a2a2a";
    const match = String(value).match(/^(\d+(?:\.\d+)?)/);
    if (!match) return "#2a2a2a";
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

// ─── Buscar reviews do mês ────────────────────────────────────────────────
const mornings = dv.pages('"03-Daily/Morning Reviews"')
    .where(p => { const d = dv.date(p.date); return d && d.year === year && d.month === month; });
const evenings = dv.pages('"03-Daily/Evening Reviews"')
    .where(p => { const d = dv.date(p.date); return d && d.year === year && d.month === month; });
const journals = dv.pages('"03-Daily/Journal"')
    .where(p => { const d = dv.date(p.date); return d && d.year === year && d.month === month; });

// Indexar por dia do mês
const morningByDay = {};
mornings.forEach(p => { const d = dv.date(p.date); if (d) morningByDay[d.day] = p; });
const eveningByDay = {};
evenings.forEach(p => { const d = dv.date(p.date); if (d) eveningByDay[d.day] = p; });
const journalByDay = {};
journals.forEach(p => { const d = dv.date(p.date); if (d) journalByDay[d.day] = p; });

// ─── Humor médio do dia ────────────────────────────────────────────────────
function avgMoodForDay(day) {
    const mp = morningByDay[day];
    const ep = eveningByDay[day];
    const mm = mp ? parseVal(mp.mood_morning) : null;
    const mn = ep ? parseVal(ep.mood_evening)  : null;
    if (mm && mn) return (mm + mn) / 2;
    return mm || mn || null;
}

// ─── Estatísticas do mês ──────────────────────────────────────────────────
let totals = {mm:0,em:0,mn:0,en:0}, cnts = {mm:0,em:0,mn:0,en:0};
mornings.forEach(p => {
    const mm = parseVal(p.mood_morning);   if (mm) { totals.mm += mm; cnts.mm++; }
    const em = parseVal(p.energy_morning); if (em) { totals.em += em; cnts.em++; }
});
evenings.forEach(p => {
    const mn = parseVal(p.mood_evening);   if (mn) { totals.mn += mn; cnts.mn++; }
    const en = parseVal(p.energy_evening); if (en) { totals.en += en; cnts.en++; }
});
const avgStr = (s,c) => c > 0 ? (s/c).toFixed(1) : "—";
const totalEntries = new Set([...Object.keys(morningByDay), ...Object.keys(eveningByDay)]).size;

// ─── Construir calendário ─────────────────────────────────────────────────
const daysInMonth = new Date(year, month, 0).getDate();
const firstDay    = new Date(year, month - 1, 1).getDay(); // 0=Dom

// Semanas no mês (para links)
function getWeekNumber(d) {
    const date = new Date(Date.UTC(d.getFullYear(), d.getMonth(), d.getDate()));
    const dayNum = date.getUTCDay() || 7;
    date.setUTCDate(date.getUTCDate() + 4 - dayNum);
    const yearStart = new Date(Date.UTC(date.getUTCFullYear(), 0, 1));
    return Math.ceil((((date - yearStart) / 86400000) + 1) / 7);
}

const weeksInMonth = new Set();
for (let d = 1; d <= daysInMonth; d++) {
    weeksInMonth.add(getWeekNumber(new Date(year, month - 1, d)));
}

const dayHeaders = `<th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Seg</th>
    <th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Ter</th>
    <th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Qua</th>
    <th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Qui</th>
    <th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Sex</th>
    <th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Sáb</th>
    <th style="color:#888;padding:4px;text-align:center;font-size:0.8em;">Dom</th>`;

const offset = (firstDay === 0) ? 6 : firstDay - 1;

let calCells = [];
for (let i = 0; i < offset; i++) calCells.push(null);
for (let d = 1; d <= daysInMonth; d++) calCells.push(d);

let calRows = "";
for (let i = 0; i < calCells.length; i += 7) {
    const week = calCells.slice(i, i + 7);
    while (week.length < 7) week.push(null);
    let rowCells = "";
    week.forEach(day => {
        if (!day) {
            rowCells += `<td style="padding:4px;"></td>`;
        } else {
            const mood = avgMoodForDay(day);
            const jp   = journalByDay[day];
            const color = mood ? moodColor(mood + "/10") : "#1a1a1a";
            const textColor = mood ? "#fff" : "#444";
            const filePath = jp ? jp.file.path : null;
            const link = filePath
                ? `<a href="${filePath}" class="internal-link" style="color:${textColor};text-decoration:none;">${day}</a>`
                : `<span style="color:${textColor}">${day}</span>`;
            const tooltip = mood ? ` title="Humor médio: ${mood.toFixed(1)}/10"` : "";
            rowCells += `<td${tooltip} style="width:40px;height:40px;text-align:center;vertical-align:middle;background:${color};border-radius:6px;padding:2px;font-size:0.9em;font-weight:bold;">${link}</td>`;
        }
    });
    calRows += `<tr style="margin-bottom:4px;">${rowCells}</tr>`;
}

// ─── Links para semanas ────────────────────────────────────────────────────
const mStr = String(month).padStart(2, '0');
const weekLinks = Array.from(weeksInMonth).sort().map(w => {
    return `<a href="00-Dashboard/Mood-Energy Tracker/Weekly/${year}/${String(w).padStart(2,'0')}" class="internal-link" style="color:#888;text-decoration:none;">Semana ${w}</a>`;
}).join(" · ");

// ─── Estatísticas resumidas ────────────────────────────────────────────────
const statsHtml = `
<div style="display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-top:16px;">
    <div style="background:${moodColor(avgStr(totals.mm,cnts.mm)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.75em;color:#eee;">😌 Humor Manhã</div>
        <div style="font-size:1.4em;font-weight:bold;color:#fff;">${avgStr(totals.mm,cnts.mm)}</div>
    </div>
    <div style="background:${moodColor(avgStr(totals.em,cnts.em)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.75em;color:#eee;">⚡ Energia Manhã</div>
        <div style="font-size:1.4em;font-weight:bold;color:#fff;">${avgStr(totals.em,cnts.em)}</div>
    </div>
    <div style="background:${moodColor(avgStr(totals.mn,cnts.mn)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.75em;color:#eee;">😌 Humor Noite</div>
        <div style="font-size:1.4em;font-weight:bold;color:#fff;">${avgStr(totals.mn,cnts.mn)}</div>
    </div>
    <div style="background:${moodColor(avgStr(totals.en,cnts.en)+"/10")};border-radius:8px;padding:10px;text-align:center;">
        <div style="font-size:0.75em;color:#eee;">⚡ Energia Noite</div>
        <div style="font-size:1.4em;font-weight:bold;color:#fff;">${avgStr(totals.en,cnts.en)}</div>
    </div>
</div>`;

dv.container.innerHTML = `
<h2 style="margin-bottom:4px;">${monthName} ${year}</h2>
<div style="color:#888;font-size:0.85em;margin-bottom:16px;">${totalEntries} entradas registradas</div>

<table style="border-collapse:separate;border-spacing:4px;">
    <thead><tr>${dayHeaders}</tr></thead>
    <tbody>${calRows}</tbody>
</table>

${statsHtml}

<div style="margin-top:16px;color:#888;font-size:0.85em;">
    📅 Semanas: ${weekLinks || "—"}
</div>`;
```

---

## 🔗 Navegação

↑ [[../../Yearly/<% tp.frontmatter.year %>|Tracker Anual <% tp.frontmatter.year %>]]

Meses: [[01|Jan]] · [[02|Fev]] · [[03|Mar]] · [[04|Abr]] · [[05|Mai]] · [[06|Jun]] · [[07|Jul]] · [[08|Ago]] · [[09|Set]] · [[10|Out]] · [[11|Nov]] · [[12|Dez]]
