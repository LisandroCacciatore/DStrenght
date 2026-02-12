# IronSystems - Estrategia de Contenido
## Build in Public: LinkedIn + YouTube + GitHub

**Objetivo:** Convertir el desarrollo de IronSystems en contenido que genera:
1. Visibilidad profesional (empleabilidad)
2. GitHub stars (credibilidad técnica)
3. Networking con coaches/founders (oportunidades)

---

## Principio Central: "Show, Don't Tell"

❌ **MAL:** "Estoy aprendiendo Apps Script"
✅ **BIEN:** "Construí un sistema de torneos con 1,500 líneas de código. Aquí está el problema que resuelve..."

**Fórmula de cada post:**
1. **Hook** (primera línea): problema o resultado
2. **Proceso**: qué construiste, cómo
3. **Aprendizajes**: 1-3 insights técnicos
4. **Call-to-action**: link a GitHub, pregunta, etc.

---

## Timeline de Contenido (4 Semanas)

### SEMANA 1: Announcement + Progress

**LinkedIn Post #1 (Día 1 - Kick-off)**
```
🚀 Starting a new build-in-public project

I'm building a powerlifting tournament manager from scratch. Why?

Problem: Meet directors waste 4+ hours manually calculating results and managing attempts. Excel breaks down with 50+ athletes.

Solution: A complete system running on Google Sheets (no backend needed).

Tech stack:
• Apps Script (serverless backend)
• HTML/CSS/JS (web UI)
• Looker Studio (analytics)
• 100% free to use

Goal: Ship V1 in 4 weeks. Open source on GitHub.

Following along? Comment below and I'll tag you in updates.

#BuildInPublic #PowerliftingTech #SideProject

[Image: mockup or sketch of the system]
```

**LinkedIn Post #2 (Día 5 - Technical Deep Dive)**
```
Week 1 update: Built the core backend 📊

Shipped today:
✅ Database schema (5 sheets, ~20 fields)
✅ Athlete registration system
✅ Auto weight class calculation
✅ UUID generation for entries

Technical challenge: Google Sheets as a database

Most people would use PostgreSQL/MongoDB, but I chose Sheets because:
1. Zero hosting costs
2. Real-time collaboration (judges can work simultaneously)
3. Non-technical users can fix data if needed
4. Built-in export to CSV/Excel

Trade-off: Limited to ~10M cells (enough for 100k+ athletes)

The hardest part? Writing performant Apps Script that doesn't timeout on large datasets.

Solution: Batch operations and caching → reduced execution time from 30s to 3s.

Code snippet in comments 👇

[Code screenshot showing batch operation optimization]

Progress: 25% complete
Next: Building the attempt ordering algorithm

#GoogleAppsScript #TechBuild #SoftwareEngineering
```

**YouTube Video #1 (Día 7 - Screen Recording)**
Title: "Building a Tournament Manager in Google Sheets | Week 1 Progress"
Length: 5-7 minutes

Script:
```
0:00 - Intro
"This week I built the core of a powerlifting tournament system. Let me show you what it does..."

0:30 - Problem Overview
[Screen: Excel screenshot with messy data]
"Meet directors currently do this manually. It's chaos."

1:00 - Demo of What Works So Far
[Screen: Registration form]
"You can now register athletes and it auto-calculates their weight class..."

3:00 - Technical Walkthrough
[Screen: Apps Script code]
"Here's the UUID generator. I used Utilities.getUuid()..."

5:00 - Challenges & Next Steps
"The biggest challenge was making Sheets performant..."
"Next week: attempt ordering algorithm"

6:30 - CTA
"Code is on GitHub [link]. If you want to follow along, subscribe."
```

---

### SEMANA 2: Results + Case Study

**LinkedIn Post #3 (Día 10 - Results System)**
```
Results calculation is harder than you think 🧮

This week I implemented DOTS and Wilks scoring for powerlifting.

The challenge? These formulas use 5th-degree polynomials:

DOTS = 500 / (a×BW⁴ + b×BW³ + c×BW² + d×BW + e) × Total

Where BW = body weight and a,b,c,d,e are sex-specific coefficients.

But here's what I learned building this:

1️⃣ Floating point precision matters
JavaScript: 83.3 + 2.5 = 85.79999999999
Solution: Round to 2 decimals

2️⃣ Edge cases are everywhere
What if athlete bombs out? (0 total → division by zero)
What if body weight is 0? (impossible, but users try)

3️⃣ Test with known values
I validated against IPF's official calculator with 50+ test cases.

The result? Accurate to 0.01 points vs official results.

Progress: 50% complete

[Code snippet of DOTS calculation]

#SoftwareEngineering #Algorithms #TechLeadership
```

**LinkedIn Post #4 (Día 14 - Attempt Panel Demo)**
```
Built a live scoring panel today ⚡

Watch how it works [GIF/video]:
• Attempts load in order (lightest first)
• Judge clicks Good/No Lift/Pass
• System auto-advances to next athlete
• Calculates next round order automatically

Technical win: This is a "modeless dialog" in Apps Script

Unlike modal dialogs that block the UI, modeless dialogs:
✅ Stay open while you work
✅ Auto-refresh every 5 seconds
✅ Don't freeze the spreadsheet

Implementation tip: Use setInterval() with google.script.run callbacks

The UI is simple HTML/CSS, but the UX feels like a real app.

50% of meet directors' pain = attempt management. This solves it.

Code on GitHub: [link]

[Screen recording of attempt panel in action]

#ProductDevelopment #UXDesign #AppsScript
```

**YouTube Video #2 (Día 14 - Feature Walkthrough)**
Title: "Live Scoring Panel for Powerlifting Meets | Apps Script Tutorial"
Length: 8-10 minutes

```
0:00 - Intro & Problem
"Managing 150+ attempts manually is chaos. Here's the solution..."

1:00 - Demo of Attempt Panel
[Live screen: click through actual UI]

3:00 - Code Walkthrough
"Let me show you how this works under the hood..."
[Show: generateAttemptOrder() function]

5:00 - Modeless Dialog Deep Dive
"The secret? Using HtmlService.createTemplateFromFile()..."

7:00 - Testing with Sample Data
[Run through full round simulation]

9:00 - CTA
"Next week: public results page. Link in description."
```

---

### SEMANA 3: MVP Complete + First User

**LinkedIn Post #5 (Día 17 - MVP Shipped)**
```
V1 is live 🎉

After 3 weeks, IronSystems Tournament Manager is feature-complete.

What it does:
✅ Athlete registration (unlimited)
✅ Automated attempt ordering
✅ Live scoring panel
✅ DOTS/Wilks calculation
✅ Public results page (shareable URL)
✅ CSV export

What it costs: $0

Built with: 1,500 lines of Apps Script, 800 lines of HTML/CSS/JS

The most satisfying part? It just works.

I tested it with a simulated 50-athlete meet and it handled everything flawlessly.

Next step: Getting it used at a real tournament.

If you organize powerlifting meets and want to test this, DM me.

GitHub: [link]
Live demo: [link]

[Screenshot grid: 4 key features]

#ProductLaunch #OpenSource #PowerliftingTech
```

**LinkedIn Post #6 (Día 21 - Case Study)**
```
Update: IronSystems was used at a real meet this weekend 📊

Organizer: [Name] from [Location]
Athletes: 32
Lifts recorded: 288
Manual calculations: 0

Feedback from the organizer:
"This saved us 4+ hours. The attempt panel is a game-changer."

What I learned from real-world testing:

1️⃣ Edge cases you never expect
   • Athlete with same name (solution: add UUID display)
   • Weight entered in lbs not kg (solution: validation)

2️⃣ Performance under load
   • 32 athletes × 9 attempts = 288 DB writes
   • Apps Script handled it without timeout

3️⃣ User feedback is gold
   • "Can you add plate loading calculator?" → added in 20 mins
   • "Results page needs team filter" → now on roadmap

This is why shipping early matters. You can't predict what users need until they use it.

V2 roadmap:
• Team scoring
• Historical athlete tracking
• Mobile PWA

[Photo from the meet with IronSystems visible on screen]

#CustomerFeedback #ProductDevelopment #AgileMethodology
```

**YouTube Video #3 (Día 21 - Case Study Video)**
Title: "First User Interview: IronSystems at a Real Powerlifting Meet"
Length: 6-8 minutes

```
0:00 - Intro
"Last weekend, IronSystems was used at its first real meet. Here's what happened..."

0:30 - Meet Overview
[B-roll from meet, if available]
"32 athletes, 3 judges, 1 system"

1:30 - Organizer Interview
[Screen recording of call or written quotes]
"What were the pain points before?"
"How did IronSystems solve them?"

3:00 - Live Demo from Meet
[Screen recording showing actual data]
"Here's the scoring panel in action..."

5:00 - Lessons Learned
"3 things I'd do differently..."

6:30 - What's Next
"V2 features based on feedback"

7:30 - CTA
"Try it for your next meet: [link]"
```

---

### SEMANA 4: Launch + Distribution

**LinkedIn Post #7 (Día 24 - Technical Writeup)**
```
How I built a serverless tournament system with zero backend 🧵

Tech decisions that made this possible:

1️⃣ Google Sheets as database
   • Pros: Free, collaborative, exportable
   • Cons: 10M cell limit, slower than SQL
   • When it makes sense: <100k records, non-technical users

2️⃣ Apps Script for logic
   • Pros: Integrated with Sheets, no hosting
   • Cons: 6-min execution limit, ES5 syntax
   • Workaround: Batch operations, use V8 runtime

3️⃣ HTML Service for UI
   • Pros: Standard HTML/CSS/JS, responsive
   • Cons: Limited to sandboxed iframes
   • Pattern: Template literals for dynamic content

4️⃣ Looker Studio for analytics
   • Pros: Free, beautiful charts, auto-updates
   • Cons: Limited customization vs D3.js

The result? A complete SaaS product with:
• 0 server costs
• 0 database fees
• 0 hosting complexity
• ~100 hours of development

Would I recommend this stack for all projects? No.

But for MVPs, internal tools, or low-volume apps? Absolutely.

Full architecture breakdown: [link to GitHub]

What "unorthodox" tech stacks have you used successfully?

[Architecture diagram]

#SystemDesign #SoftwareArchitecture #CloudComputing
```

**LinkedIn Post #8 (Día 26 - Open Source Announcement)**
```
IronSystems is now 100% open source 🎁

I'm releasing the full codebase (1,500+ lines) under MIT license.

What's included:
📂 Complete Apps Script code
📂 HTML/CSS templates
📂 Setup documentation
📂 Sample data generator
📂 Test suite

Why open source?

1. Give back to the powerlifting community
2. Learn from code reviews (already got 3 PRs!)
3. Build in public credibility

Star the repo if you find it useful: [GitHub link]

If you're learning Apps Script, this is a real-world example of:
• Database design in Sheets
• Web app deployment
• Complex calculations (DOTS/Wilks)
• Modeless dialogs
• Batch operations

Contributors welcome 🙏

[GitHub stats card showing stars/forks]

#OpenSource #GitHub #CodingCommunity
```

**LinkedIn Post #9 (Día 28 - Reflection Post)**
```
4 weeks, 1 product, 10 lessons 🧠

What I learned building IronSystems from scratch:

1. Ship fast > Ship perfect
   V1 had bugs. Users found them. I fixed them. That's the process.

2. Real users teach you things docs can't
   "Add plate loading calculator" wasn't on my roadmap. Users asked for it.

3. Build-in-public creates accountability
   Knowing people were watching made me ship faster.

4. Open source attracts talent
   Got 2 code reviews that improved my architecture significantly.

5. Side projects are better with a purpose
   Building for a real community (powerlifters) kept me motivated.

6. Technical debt is fine in V1
   I'll refactor later. Shipping > perfecting.

7. Documentation compounds
   Good README = less support DMs = more time coding.

8. Constraints breed creativity
   No budget → Google Sheets → actually better UX for target users.

9. Show the process, not just the result
   This post has 10x engagement vs my "finished product" announcement.

10. Community > code
    The connections I made matter more than the lines written.

Next project starting soon. What should I build?

[Project stats graphic]

#ProductDevelopment #TechLeadership #BuildInPublic
```

**YouTube Video #4 (Día 28 - Full Tutorial)**
Title: "Complete Tutorial: Build a Tournament Manager in Google Sheets"
Length: 25-30 minutes

```
0:00 - Intro & Overview
"In this tutorial, I'll show you exactly how to build..."

2:00 - Setup & Architecture
[Whiteboard-style diagram]

5:00 - Database Design
[Screen: creating sheets, headers]

10:00 - Core Functions
[Code walkthrough: registration, attempts, results]

18:00 - Web App Deployment
[Screen: deploy process step-by-step]

23:00 - Testing & Debugging
[Live: run through full tournament]

27:00 - Advanced Features
"Here's how to extend this..."

29:00 - Outro & Resources
"All code on GitHub. Questions in comments."
```

---

## Formatos de Contenido Adicional

### GitHub README Template
```markdown
# 🏋️ IronSystems Tournament Manager

![Demo](demo.gif)

> Free, open-source powerlifting meet management system built on Google Sheets

[Live Demo](#) | [Documentation](#) | [Video Tutorial](#)

## ⚡ Quick Start

1. Copy [template](#)
2. Enable Apps Script
3. Deploy web app
4. Run your meet!

## 🎯 Features

- ✅ Athlete registration & weigh-in
- ✅ Automated attempt ordering
- ✅ Live scoring panel
- ✅ DOTS & Wilks calculation
- ✅ Public results page
- ✅ CSV export

## 🛠️ Tech Stack

- **Backend:** Apps Script (JavaScript)
- **Database:** Google Sheets
- **Frontend:** HTML/CSS/JS
- **Analytics:** Looker Studio

## 📚 Documentation

See [SETUP_GUIDE.md](SETUP_GUIDE.md) for installation.

See [ARCHITECTURE.md](ARCHITECTURE.md) for technical details.

## 🤝 Contributing

Contributions welcome! See [CONTRIBUTING.md](CONTRIBUTING.md)

## 📄 License

MIT © [Your Name]

---

Made with ❤️ for the powerlifting community
```

### Short-Form Content (Twitter/X Thread)
```
🧵 I built a powerlifting tournament system in 4 weeks. Here's what I learned about shipping fast:

1/ Problem: Meet directors waste hours on manual calculations
Solution: Automate everything in Google Sheets

2/ Tech choice: Apps Script (not sexy, but practical)
Why? $0 hosting, real-time collaboration, familiar UI

3/ Biggest challenge: Making Sheets performant
Optimization: Batch writes, caching → 10x speedup

4/ First user feedback changed everything
"Add plate calculator" → 20 mins to implement → now most-used feature

5/ Key insight: Ship V1 fast, iterate with users
Perfect is the enemy of done

Full code on GitHub: [link]

What problem should I solve next? 👇
```

### Reddit Post Template (r/powerlifting)
```
Title: [I Made This] Free tournament management system for meet directors

Body:
Hi r/powerlifting! I built a free tool to help meet directors run smoother competitions.

**What it does:**
- Registers athletes & calculates weight classes
- Orders attempts automatically (lightest first)
- Live scoring panel for judges
- Calculates DOTS/Wilks
- Generates public results page

**Why I built it:**
Helped organize a local meet and saw directors struggling with Excel. This automates 90% of the admin work.

**How it works:**
Runs entirely on Google Sheets (no server needed). Just copy the template and you're ready.

**It's free & open source:** [GitHub link]

**Demo video:** [YouTube link]

Happy to answer questions or get feedback!
```

---

## Métricas de Éxito (Trackear Semanalmente)

### LinkedIn
- [ ] Impresiones por post (target: 1,000+)
- [ ] Engagement rate (target: 3-5%)
- [ ] Connection requests (target: 10-20/semana)
- [ ] Profile views (target: 200+/semana)

### GitHub
- [ ] Stars (target: 50+ en 4 semanas)
- [ ] Forks (target: 10+)
- [ ] Issues opened (señal de uso real)
- [ ] Contributors (target: 1-2)

### YouTube
- [ ] Views por video (target: 100+ primeros 7 días)
- [ ] Subscribers ganados (target: 20-50)
- [ ] Comments (engagement signal)
- [ ] CTR on GitHub link (target: 5-10%)

### Distribución
- [ ] Template downloads (via Google Sheet "Make a copy")
- [ ] Web app visits (Analytics)
- [ ] Real meets using system (target: 3-5)

---

## Plantillas de Respuesta (Para Comments/DMs)

**Pregunta técnica:**
```
Great question! [Respuesta breve]

For more details, check out the [technical doc/code] here: [link]

Feel free to open a GitHub issue if you run into problems.
```

**Pedido de feature:**
```
Love this idea! I'll add it to the roadmap.

In the meantime, you could [workaround si existe].

Track progress here: [GitHub issue link]
```

**Oferta de colaboración:**
```
Awesome! I'd love to collaborate.

What area interests you most?
- Frontend/UX improvements
- New features (team scoring, etc.)
- Documentation/tutorials
- Testing with real meets

Let's hop on a quick call: [Calendly link]
```

**Recruiter/Job inquiry:**
```
Thanks for reaching out! I'm [open to opportunities/focused on current role].

You can see more of my work here: [portfolio link]

Best way to reach me: [email]
```

---

## Recursos para Crear Contenido

### Grabación de Pantalla
- **Mac:** QuickTime (built-in)
- **Windows:** OBS Studio (free)
- **Browser:** Loom (free tier ok)

### Edición de Video
- **Simple:** iMovie / Windows Video Editor
- **Pro:** DaVinci Resolve (free)
- **Thumbnails:** Canva

### Gráficos/Diagramas
- **Architecture:** Excalidraw (free)
- **Flowcharts:** draw.io (free)
- **Mockups:** Figma (free tier)

### Code Screenshots
- **carbon.now.sh** (beautiful code snippets)
- **ray.so** (alternative with themes)

---

## Next Actions (Checklist)

**ANTES de escribir código:**
- [ ] Crear GitHub repo (público)
- [ ] Escribir README inicial
- [ ] Setup LinkedIn posts calendar (draft 9 posts)
- [ ] Grabar intro para Video #1

**Durante desarrollo (semanal):**
- [ ] Publicar 2-3 LinkedIn posts
- [ ] Hacer 1 video (screen recording + walkthrough)
- [ ] Commit a GitHub daily (show progress)
- [ ] Responder comments/DMs (community building)

**Post-launch:**
- [ ] Distribution push (Reddit, FB groups, direct outreach)
- [ ] Collect testimonials from first users
- [ ] Write reflection post (lessons learned)
- [ ] Plan next project (based on feedback/interest)

---

**Ready to build in public? Start posting Day 1! 🚀**
