---
title: "Semana {{week_start}} — {{week_end}}"
type: mood-tracker
frequency: weekly
week_start:
week_end:
year:
tags:
  - tracker/weekly
  - tracker/mood
---

# 📊 Tracker Semanal de Humor & Energia

```dataviewjs
// ─── Configuração ─────────────────────────────────────────────────────────
const cur = dv.current();
const weekStart = dv.date(cur.week_start);
const weekEnd   = dv.date(cur.week_end);

if (!weekStart || !weekEnd) {
    dv.paragraph("⚠️ **Preencha `week_start` e `week_end` no frontmatter** (ex: `2026-03-16`)");
    return;
}

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

function cell(value) {
    const color = moodColor(value);
    const text  = value ? String(value) : "—";
    const textColor = value ? "#fff" : "#666";
    return `<td style="background:${color};color:${textColor};text-align:center;border-radius:6px;padding:6px 10px;font-weight:bold;font-size:0.9em;">${text}</td>`;
}

function parseVal(v) {
    if (!v) return null;
    const m = String(v).match(/^(\d+(?:\.\d+)?)/);
    return m ? parseFloat(m[1]) : null;
}

// ─── Buscar reviews da semana ──────────────────────────────────────────────
const mornings = dv.pages('"03-Daily/Morning Reviews"')
    .where(p => p.date && dv.date(p.date) >= weekStart && dv.date(p.date) <= weekEnd);
const evenings = dv.pages('"03-Daily/Evening Reviews"')
    .where(p => p.date && dv.date(p.date) >= weekStart && dv.date(p.date) <= weekEnd);
const journals = dv.pages('"03-Daily/Journal"')
    .where(p => p.date && dv.date(p.date) >= weekStart && dv.date(p.date) <= weekEnd);

// ─── Maps por data ─────────────────────────────────────────────────────────
const morningMap = {};
mornings.forEach(p => { if (p.date) morningMap[dv.date(p.date).toFormat("yyyy-MM-dd")] = p; });
const eveningMap = {};
evenings.forEach(p => { if (p.date) eveningMap[dv.date(p.date).toFormat("yyyy-MM-dd")] = p; });
const journalMap = {};
journals.forEach(p => { if (p.date) journalMap[dv.date(p.date).toFormat("yyyy-MM-dd")] = p; });

// ─── Calcular médias ──────────────────────────────────────────────────────
let sums = {mm:0,em:0,mn:0,en:0}, counts = {mm:0,em:0,mn:0,en:0};
mornings.forEach(p => {
    const mm = parseVal(p.mood_morning);   if (mm) { sums.mm += mm; counts.mm++; }
    const em = parseVal(p.energy_morning); if (em) { sums.em += em; counts.em++; }
});
evenings.forEach(p => {
    const mn = parseVal(p.mood_evening);   if (mn) { sums.mn += mn; counts.mn++; }
    const en = parseVal(p.energy_evening); if (en) { sums.en += en; counts.en++; }
});
const avg = (s,c) => c > 0 ? (s/c).toFixed(1) : null;

// ─── Dias da semana em PT-BR ───────────────────────────────────────────────
const dayNames = ["Dom","Seg","Ter","Qua","Qui","Sex","Sáb"];

// ─── Montar HTML por dia ───────────────────────────────────────────────────
let rows = "";
let hasAnyData = false;

for (let i = 0; i < 7; i++) {
    const d = weekStart.plus({ days: i });
    const key = d.toFormat("yyyy-MM-dd");
    const mp = morningMap[key];
    const ep = eveningMap[key];
    const jp = journalMap[key];

    if (!mp && !ep && !jp) continue;
    hasAnyData = true;

    const dayLabel = dayNames[d.weekday % 7] ?? d.toFormat("EEE");
    const dateStr  = d.toFormat("dd/MM");
    const filePath = jp ? jp.file.path : (mp ? mp.file.path : ep.file.path);
    const link     = `<a href="${filePath}" class="internal-link">${dateStr} ${dayLabel}</a>`;
    const highlight = jp?.highlight
        ? `<span style="color:#aaa;font-size:0.85em">${jp.highlight}</span>`
        : "<span style='color:#555'>—</span>";

    rows += `<tr>
        <td style="padding:6px 10px;white-space:nowrap;">${link}</td>
        ${cell(mp?.mood_morning)}
        ${cell(mp?.energy_morning)}
        ${cell(ep?.mood_evening)}
        ${cell(ep?.energy_evening)}
        <td style="padding:6px 10px;max-width:200px;">${highlight}</td>
    </tr>`;
}

if (!hasAnyData) {
    rows = `<tr><td colspan="6" style="text-align:center;color:#666;padding:20px;">Nenhuma entrada encontrada para esta semana.</td></tr>`;
}

// Linha de médias
const avgRow = `<tr style="border-top:2px solid #444;font-weight:bold;">
    <td style="padding:6px 10px;">📊 Média</td>
    ${cell(avg(sums.mm,counts.mm) ? avg(sums.mm,counts.mm)+"/10" : null)}
    ${cell(avg(sums.em,counts.em) ? avg(sums.em,counts.em)+"/10" : null)}
    ${cell(avg(sums.mn,counts.mn) ? avg(sums.mn,counts.mn)+"/10" : null)}
    ${cell(avg(sums.en,counts.en) ? avg(sums.en,counts.en)+"/10" : null)}
    <td></td>
</tr>`;

const periodo = `${weekStart.toFormat("dd/MM")} – ${weekEnd.toFormat("dd/MM/yyyy")}`;

dv.container.innerHTML = `
<div style="margin-bottom:8px;color:#888;font-size:0.9em;">📅 ${periodo}</div>
<table style="width:100%;border-collapse:separate;border-spacing:0 4px;">
    <thead>
        <tr style="color:#888;font-size:0.8em;text-align:center;">
            <th style="text-align:left;padding:4px 10px;">Dia</th>
            <th style="padding:4px 8px;">😌 Humor<br><span style="font-size:0.8em">Manhã</span></th>
            <th style="padding:4px 8px;">⚡ Energia<br><span style="font-size:0.8em">Manhã</span></th>
            <th style="padding:4px 8px;">😌 Humor<br><span style="font-size:0.8em">Noite</span></th>
            <th style="padding:4px 8px;">⚡ Energia<br><span style="font-size:0.8em">Noite</span></th>
            <th style="text-align:left;padding:4px 10px;">✨ Destaque</th>
        </tr>
    </thead>
    <tbody>${rows}${avgRow}</tbody>
</table>`;
```

---

## 🔗 Navegação

↑ [[Tracker Mensal]] | [[Tracker Anual]]
