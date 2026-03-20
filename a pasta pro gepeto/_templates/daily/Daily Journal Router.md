<%*
const fileName = tp.file.title;

if (fileName.includes("Morning Review")) {
  const date = fileName.replace(" Morning Review", "");
  const dayNames = ["Domingo","Segunda-feira","Terça-feira","Quarta-feira","Quinta-feira","Sexta-feira","Sábado"];
  const monthNames = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
  const d = new Date(date + "T12:00:00");
  const dayName = dayNames[d.getDay()];
  const monthName = monthNames[d.getMonth()];
  const dayNum = d.getDate();
  const year = d.getFullYear();

  tR += `---
title: "${date} Morning Review"
type: review
frequency: morning
date: ${date}
created: ${date}
mood_morning:
energy_morning:
tags:
  - review/morning
---

# ☀️ Revisão da Manhã — ${dayName}, ${dayNum} de ${monthName} de ${year}

> Como estou chegando hoje?
> **Humor:** \`= this.mood_morning\` / 10  |  **Energia:** \`= this.energy_morning\` / 10

---

## 1. 📅 Calendário
O que está agendado para hoje?


---

## 2. 🎯 Top 3 Prioridades

- [ ] 
- [ ] 
- [ ] 

---

## 3. ✍️ Intenção de Escrita

**Projeto:** 

**Meta:** ___ palavras / ___ minutos

**Foco:** 

---

## 4. 🧠 Intenção do Dia
Com qual energia / intenção quero atravessar hoje?


---

## 🔗 Navegação

[[03-Daily/Journal/${date}|← Diário do dia]] | [[03-Daily/Evening Reviews/${date} Evening Review|🌙 Revisão da Noite →]]`;

} else if (fileName.includes("Evening Review")) {
  const date = fileName.replace(" Evening Review", "");
  const dayNames = ["Domingo","Segunda-feira","Terça-feira","Quarta-feira","Quinta-feira","Sexta-feira","Sábado"];
  const monthNames = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
  const d = new Date(date + "T12:00:00");
  const dayName = dayNames[d.getDay()];
  const monthName = monthNames[d.getMonth()];
  const dayNum = d.getDate();
  const year = d.getFullYear();

  tR += `---
title: "${date} Evening Review"
type: review
frequency: evening
date: ${date}
created: ${date}
mood_evening:
energy_evening:
tags:
  - review/evening
---

# 🌙 Revisão da Noite — ${dayName}, ${dayNum} de ${monthName} de ${year}

> Como estou fechando hoje?
> **Humor:** \`= this.mood_evening\` / 10  |  **Energia:** \`= this.energy_evening\` / 10

---

## 1. ✅ O que consegui fazer hoje?

- 

---

## 2. ✍️ Escrita feita

**Palavras escritas:** 

**Projeto trabalhado:** 

**Como foi:** 

---

## 3. 💭 O que está na minha mente?
*(Capture aqui — vai para a caixa de entrada depois)*

- 

---

## 4. 🙏 Gratidão do dia

1. 

---

## 5. 🔮 Prioridade de amanhã

- 

---

## 🔗 Navegação

[[03-Daily/Journal/${date}|← Diário do dia]] | [[03-Daily/Morning Reviews/${date} Morning Review|☀️ Revisão da Manhã →]]`;

} else {
  // É um Daily Note normal — usa o template de diário completo
  const dailyQuotes = [
    '"Comece a escrever, não importa o quê. A água não flui até que a torneira seja aberta." — Louis L\'Amour',
    '"O momento mais assustador é sempre logo antes de começar." — Stephen King',
    '"Você sempre pode editar uma página ruim. Você não pode editar uma página em branco." — Jodi Picoult',
    '"Um escritor é alguém para quem escrever é mais difícil do que para outras pessoas." — Thomas Mann',
    '"Comece onde você está. Use o que você tem. Faça o que você pode." — Arthur Ashe',
    '"O segredo de avançar é começar." — Mark Twain',
    '"Escreva o que não deve ser esquecido." — Isabel Allende',
    '"Ou escreva algo que valha a pena ler, ou faça algo que valha a pena escrever." — Benjamin Franklin',
    '"Se você quer ser escritor, deve fazer duas coisas acima de todas as outras: ler muito e escrever muito." — Stephen King',
    '"Não há maior agonia do que carregar uma história não contada dentro de você." — Maya Angelou',
    '"Você não começa escrevendo boas histórias. Você começa escrevendo lixo e achando que é bom, e gradualmente vai melhorando." — Octavia E. Butler',
    '"O primeiro rascunho é apenas você contando a história para si mesmo." — Terry Pratchett',
    '"Escrevo para descobrir o que sei." — Flannery O\'Connor',
    '"A imaginação é o começo da criação." — George Bernard Shaw',
    '"Feche a porta. Escreva sem ninguém olhando por cima do seu ombro." — Barbara Kingsolver',
    '"Para sobreviver, você deve contar histórias." — Umberto Eco',
    '"Escrevemos para saborear a vida duas vezes, no momento e em retrospecto." — Anaïs Nin',
    '"Preencha seu papel com as respirações do seu coração." — William Wordsworth',
    '"O desejo de escrever cresce com a escrita." — Erasmus',
    '"Criatividade é a inteligência se divertindo." — Albert Einstein',
    '"Posso sacudir tudo enquanto escrevo; minhas tristezas desaparecem, minha coragem renasce." — Anne Frank',
    '"As palavras são uma lente para focar a mente." — Ayn Rand',
    '"Sem lágrimas no escritor, sem lágrimas no leitor." — Robert Frost',
    '"Quase toda boa escrita começa com esforços terríveis." — Anne Lamott',
    '"Escreva com força e clareza sobre o que dói." — Ernest Hemingway',
    '"Leia, leia, leia. Leia tudo." — William Faulkner',
    '"Um dia vou encontrar as palavras certas, e serão simples." — Jack Kerouac',
    '"O papel do escritor não é dizer o que todos podemos dizer, mas o que somos incapazes de dizer." — Anaïs Nin',
    '"Amanhã pode ser o inferno, mas hoje foi um bom dia de escrita, e nos bons dias de escrita nada mais importa." — Neil Gaiman',
    '"Não intelectualizo demais o processo. Tento manter simples: conte a maldita história." — Tom Clancy',
  ];
  const date = fileName;
  const d = new Date(date + "T12:00:00");
  const dayNames = ["Domingo","Segunda-feira","Terça-feira","Quarta-feira","Quinta-feira","Sexta-feira","Sábado"];
  const monthNames = ["Janeiro","Fevereiro","Março","Abril","Maio","Junho","Julho","Agosto","Setembro","Outubro","Novembro","Dezembro"];
  const dayName = dayNames[d.getDay()];
  const monthName = monthNames[d.getMonth()];
  const dayNum = d.getDate();
  const year = d.getFullYear();
  const start = new Date(d.getFullYear(), 0, 0);
  const dayOfYear = Math.floor((d - start) / 86400000);
  const quote = dailyQuotes[dayOfYear % dailyQuotes.length];

  const yesterday = new Date(d); yesterday.setDate(d.getDate() - 1);
  const tomorrow = new Date(d); tomorrow.setDate(d.getDate() + 1);
  const fmt = (dt) => dt.toISOString().split("T")[0];

  // Bloco DataviewJS para Estado do Dia (busca das revisões)
  const dvEstado = [
    "```dataviewjs",
    `const _d = dv.date(dv.current().date).toFormat("yyyy-MM-dd");`,
    `const _mp = dv.pages('"03-Daily/Morning Reviews"').find(p => p.date && dv.date(p.date).toFormat("yyyy-MM-dd") === _d);`,
    `const _ep = dv.pages('"03-Daily/Evening Reviews"').find(p => p.date && dv.date(p.date).toFormat("yyyy-MM-dd") === _d);`,
    `const _mm = _mp?.mood_morning    ?? "—"; const _em = _mp?.energy_morning ?? "—";`,
    `const _mn = _ep?.mood_evening    ?? "—"; const _en = _ep?.energy_evening ?? "—";`,
    `dv.container.innerHTML = "<table style='width:100%;border-collapse:collapse'>"`,
    `    + "<thead><tr><th style='text-align:left;padding:4px 8px;color:#888'></th><th style='padding:4px 12px;color:#888'>Manhã</th><th style='padding:4px 12px;color:#888'>Noite</th></tr></thead><tbody>"`,
    `    + "<tr><td style='padding:4px 8px'>😌 Humor</td><td style='padding:4px 12px;text-align:center;font-weight:bold'>" + _mm + "</td><td style='padding:4px 12px;text-align:center;font-weight:bold'>" + _mn + "</td></tr>"`,
    `    + "<tr><td style='padding:4px 8px'>⚡ Energia</td><td style='padding:4px 12px;text-align:center;font-weight:bold'>" + _em + "</td><td style='padding:4px 12px;text-align:center;font-weight:bold'>" + _en + "</td></tr>"`,
    `    + "</tbody></table>";`,
    "```"
  ].join("\n");

  tR += `---
title: "${date}"
type: daily
date: ${date}
day: ${dayName}
highlight:
created: ${date}
tags:
  - daily
  - journal
---

# ${dayName}, ${dayNum} de ${monthName} de ${year}

> [!quote]
> *${quote}*

---

## 📊 Estado do Dia

${dvEstado}

---

## 📋 Tarefas do Dia

### Deve Fazer
- [ ] 

### Deveria Fazer
- [ ] 

### Poderia Fazer
- [ ] 

---

## 📝 Notas & Pensamentos



---

## ✨ Destaque do Dia
\`= this.highlight\`

---

## 🔗 Navegação

[[${fmt(yesterday)}|← Ontem]] | [[${fmt(tomorrow)}|Amanhã →]]

**Reviews do dia:**
- [[03-Daily/Morning Reviews/${date} Morning Review|☀️ Revisão da Manhã]]
- [[03-Daily/Evening Reviews/${date} Evening Review|🌙 Revisão da Noite]]

[[Daily Dashboard|⬆ Daily Dashboard]]`;
}
%>
