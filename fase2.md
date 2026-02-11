# 🏋️ IronSystems Ecosystem - Arquitectura Completa

> **Un ERP para Entrenadores de Fuerza que escala desde Google Sheets hasta SaaS Cloud**

---

## 📋 Índice

1. [Concepto Central](#concepto-central)
2. [Training Tracker (IronSystems: Core)](#training-tracker-ironsystems-core)
3. [Tournament Manager (IronSystems: Command)](#tournament-manager-ironsystems-command)
4. [Conexión del Ecosistema](#conexión-del-ecosistema)
5. [Propuesta de Valor](#propuesta-de-valor)
6. [Estrategia de Retención](#estrategia-de-retención)
7. [Roadmap de Escalamiento](#roadmap-de-escalamiento)
8. [Modelo de Ingresos](#modelo-de-ingresos)
9. [Plan de Ejecución](#plan-de-ejecución)

---

## 🎯 Concepto Central

### **IronSystems**: El Motor de Datos Deportivo

El valor no está en anotar kilos, sino en **interpretar la data para tomar decisiones**.

```
┌─────────────────────────────────────────────────────────────┐
│                     IRONSYSTEMS ECOSYSTEM                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────┐              ┌──────────────────┐    │
│  │  TRAINING CORE   │◄────────────►│  MEET COMMAND    │    │
│  │   (Uso Diario)   │   The Bridge │  (Día del Evento)│    │
│  │                  │              │                  │    │
│  │  • Coach Tool    │              │  • Organizador   │    │
│  │  • Multi-Atleta  │              │  • Ranking Live  │    │
│  │  • Auto-Ajuste   │              │  • Histórico     │    │
│  └──────────────────┘              └──────────────────┘    │
│           │                                   │              │
│           │         UUID del Atleta          │              │
│           └───────────────┬──────────────────┘              │
│                           ▼                                  │
│                  ┌─────────────────┐                        │
│                  │  Perfil Global  │                        │
│                  │   del Atleta    │                        │
│                  └─────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

### Los 3 Pilares

| Componente | Propósito | Usuario |
|------------|-----------|---------|
| **Training Core (TC)** | Programación y seguimiento diario | Coach |
| **Meet Command (MC)** | Gestión de torneos y ranking | Organizador |
| **The Bridge** | ID único que conecta ambos mundos | Sistema |

---

## 💪 Training Tracker (IronSystems: Core)

### Arquitectura Técnica

#### Estructura de Base de Datos (Google Sheets)

**⚠️ Regla Fundamental**: NO usar una pestaña por atleta (error fatal que impide escalar)

```
ESTRUCTURA DE SHEETS:
┌──────────────────────────────────────────────────────────┐
│ 📊 DB_Athletes                                           │
│ ─────────────────────────────────────────────────────── │
│ UUID | Nombre | Peso | Altura | Email | Max_SQ | Max_BP | Max_DL
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ 📈 DB_Program                                            │
│ ─────────────────────────────────────────────────────── │
│ Block_ID | Microciclo | Exercise_ID | Sets | Reps | %TM
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ 📝 DB_Logs                                               │
│ ─────────────────────────────────────────────────────── │
│ Timestamp | Athlete_UUID | Exercise_ID | Set | Reps | Load | RPE | e1RM
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ 📺 Front_End_Coach                                       │
│ ─────────────────────────────────────────────────────── │
│ Dashboard dinámico con QUERY() y FILTER()
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ 📱 Input_Mobile                                          │
│ ─────────────────────────────────────────────────────── │
│ Google Form / AppSheet → DB_Logs
└──────────────────────────────────────────────────────────┘
```

### Funcionalidades Clave (Apps Script)

#### 1️⃣ Algoritmo de Auto-Regulación

```javascript
/**
 * Detecta cuando el atleta está sobrecargado
 * y sugiere ajuste automático del Training Max
 */
function checkAutoRegulation() {
  // Si RPE_Real > RPE_Target por 2 semanas consecutivas
  // → Sugerir reducción de TM en 2.5%
  
  const logs = getAthleteWeeks(athleteUUID, 2);
  const overload = logs.filter(log => log.rpeReal > log.rpeTarget);
  
  if (overload.length >= 10) { // 2 semanas ≈ 10 sesiones
    adjustTrainingMax(athleteUUID, -0.025); // -2.5%
    sendAlert("Auto-regulación activada");
  }
}
```

#### 2️⃣ Detección de Estancamiento

```javascript
/**
 * Calcula la pendiente de mejora en e1RM
 * Alerta si no hay progreso en 4 microciclos
 */
function detectPlateau() {
  const cycles = getLast4Microcycles(athleteUUID);
  const e1RMs = cycles.map(c => c.estimated1RM);
  
  const slope = calculateSlope(e1RMs);
  
  if (slope <= 0) {
    flagAthlete(athleteUUID, "REVIEW_NEEDED");
    generateRecommendations(athleteUUID);
  }
}
```

#### 3️⃣ Generador de Reportes PDF

```javascript
/**
 * Genera PDF profesional con logo del coach
 * y envía por email al atleta
 */
function generateMonthlyReport(athleteUUID) {
  const data = getMonthData(athleteUUID);
  const charts = createCharts(data); // Charts Service
  
  const pdf = createPDF({
    logo: getCoachLogo(),
    athlete: getAthleteName(athleteUUID),
    charts: charts,
    summary: generateSummary(data)
  });
  
  sendEmail(getAthleteEmail(athleteUUID), pdf);
}
```

### Diferenciación por Capas

| Feature | **Lead Magnet** | **FREE** | **PRO ($49/mo)** |
|---------|----------------|----------|------------------|
| **Atletas** | N/A | Máximo 3 | ✅ Ilimitado |
| **Input de datos** | Web form simple | Manual en celda | AppSheet Whitelabel |
| **Historial** | N/A | Último mes | Histórico completo |
| **Analítica** | Cálculo 1RM puntual | Gráfico simple | Tendencia + Fatiga + Reportes PDF |
| **Automatización** | ❌ | ❌ | ✅ Ajuste automático de cargas |
| **Detección estancamiento** | ❌ | ❌ | ✅ Alertas automáticas |
| **Export a torneo** | ❌ | ❌ | ✅ JSON/CSV directo |
| **Reportes PDF** | ❌ | ❌ | ✅ Con logo personalizado |

### 🎁 Lead Magnet #1: "Calculadora de Intentos para Torneo"

**Propósito**: Captura de emails

**Funcionalidad**:
- Input: Tu 1RM actual
- Output: Opener (90%), 2do intento (95%), 3er intento (102%)
- Basado en probabilidades estadísticas de éxito

**Implementación**: Google Sheet bloqueada o landing page simple

---

## 🏆 Tournament Manager (IronSystems: Command)

### Arquitectura Técnica

```
ESTRUCTURA DE SHEETS:
┌──────────────────────────────────────────────────────────┐
│ 📋 DB_Entries                                            │
│ ─────────────────────────────────────────────────────── │
│ UUID | Nombre | Categoría | Peso_Corporal | Opener_SQ | Opener_BP | Opener_DL
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ ⚡ Live_Scoring                                          │
│ ─────────────────────────────────────────────────────── │
│ Interfaz de carga rápida para jueces
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ 📺 Public_Display                                        │
│ ─────────────────────────────────────────────────────── │
│ Diseñada para proyectar en TV - Ranking en vivo
└──────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────┐
│ 📊 DB_Historical                                         │
│ ─────────────────────────────────────────────────────── │
│ Base de datos maestra - Ranking histórico acumulativo
└──────────────────────────────────────────────────────────┘
```

### Funcionalidades Clave

#### 1️⃣ Cálculo en Tiempo Real

```javascript
// Actualización automática tras cada intento
=ARRAYFORMULA(
  IF(C2:C<>"", 
    SQ1 + SQ2 + SQ3 + BP1 + BP2 + BP3 + DL1 + DL2 + DL3,
    ""
  )
)

// Coeficiente DOTS
=ARRAYFORMULA(
  IF(E2:E<>"",
    E2:E / (500 - 450 * EXP(-0.01 * F2:F)),
    ""
  )
)
```

#### 2️⃣ Gestión de Flights

```javascript
/**
 * Ordena levantadores por peso de barra ascendente
 * Se ejecuta automáticamente tras cada ronda
 */
function sortFlight() {
  const flight = getCurrentFlight();
  const sorted = flight.sort((a, b) => 
    a.nextAttemptWeight - b.nextAttemptWeight
  );
  
  updateFlightOrder(sorted);
}
```

#### 3️⃣ Histórico Consolidado

```javascript
/**
 * Al finalizar torneo, exporta resultados
 * a la base de datos maestra
 */
function closeEvent() {
  const results = getFinalResults();
  
  // Exportar a DB_Historical
  appendToMasterDB(results);
  
  // Generar certificados
  generateCertificates(results);
  
  // Actualizar rankings globales
  updateGlobalRankings();
}
```

### Diferenciación por Capas

| Feature | **Lead Magnet** | **FREE** | **PRO ($49/mo)** |
|---------|----------------|----------|------------------|
| **Atletas/Evento** | N/A | Máximo 20 | ✅ Ilimitado |
| **Pantalla en vivo** | ❌ | Estática | ✅ Dinámica (Auto-refresh) |
| **Histórico** | ❌ | Solo ese evento | ✅ Ranking de Liga acumulativo |
| **Branding** | Genérico | Logo IronSystems | ✅ Logo Organizador + Sponsors |
| **Certificados PDF** | ❌ | ❌ | ✅ Automáticos |
| **Export de datos** | ❌ | Manual | ✅ JSON/CSV automático |
| **Estadísticas avanzadas** | ❌ | ❌ | ✅ DOTS, Wilks, tendencias |

### 🎁 Lead Magnet #2: "Generador de Reglas de Competencia"

**Propósito**: Captura de emails + posicionar autoridad

**Funcionalidad**:
- Wizard paso a paso para configurar reglas del torneo
- Exporta PDF profesional listo para imprimir
- Incluye templates de categorías (IPF, USAPL, etc.)

**Implementación**: Google Form con lógica condicional + Apps Script para generar PDF

---

## 🔗 Conexión del Ecosistema

### El Loop que Genera Valor Exponencial

```
┌─────────────────────────────────────────────────────────────┐
│                     ECOSYSTEM LOOP                           │
└─────────────────────────────────────────────────────────────┘

1️⃣ FLUJO PRE-TORNEO
   ┌──────────────┐
   │ Training Core│
   │   (Coach)    │
   └──────┬───────┘
          │ Selecciona atletas para competir
          │ Exporta: UUID + Nombre + Best Marks
          ▼
   ┌──────────────┐
   │Meet Command  │
   │ (Organizador)│
   └──────────────┘
   Import automático → CERO tipeo manual

2️⃣ FLUJO POST-TORNEO
   ┌──────────────┐
   │Meet Command  │
   │ (Organizador)│
   └──────┬───────┘
          │ Finaliza evento
          │ Genera "Performance Report" por atleta
          ▼
   ┌──────────────┐
   │ Training Core│
   │   (Coach)    │
   └──────────────┘
   Import automático → Actualiza 1RM oficiales
                     → Recalcula % para próximo ciclo

3️⃣ VALOR COMPUESTO
   Más torneos = Más data histórica = Más insights
   = Mayor dependencia del sistema
```

### El Bridge: UUID del Atleta

```javascript
// Estructura del identificador único
const athleteUUID = generateUUID(); 
// Ejemplo: "ATH-2024-AR-0001"

// Presente en ambos sistemas:
{
  uuid: "ATH-2024-AR-0001",
  name: "Juan Pérez",
  trainingHistory: [...],  // Training Core
  competitionHistory: [...] // Meet Command
}
```

---

## 💎 Propuesta de Valor

### El Problema Emocional del Coach

**Síndrome del Impostor Profesional**

> "Siento que cobro caro para mandar un Excel que mi cliente podría descargar gratis de internet"

**Aspiración**: Sentirse un **Director de Rendimiento**, no un contador de repeticiones.

### El Problema Operativo

- ⏰ 4 horas los domingos actualizando rutinas manualmente
- 📉 Pierde el historial cuando el atleta cambia de planilla
- 🤷 No tiene métricas claras para saber si el programa funciona
- 📊 No puede demostrar objetivamente su valor como coach

### Diferenciador vs Excel Común

| Excel Tradicional | IronSystems |
|-------------------|-------------|
| 📋 Estático | ⚡ Dinámico |
| 💾 Guarda datos | 🧠 **Toma decisiones** |
| 👨 Require intervención manual | 🤖 Sugiere pesos, alerta fatiga |
| 📈 Gráficos manuales | 📊 Visualización automática |
| 💔 Se pierde cuando cambia de planilla | 🔒 Historial permanente y consolidado |

### ¿Por qué $49/mes es Lógico?

**Matemática del Coach**:
- 1 cliente de coaching = $50-$150/mes
- Tiempo ahorrado por IronSystems = 4-5 horas/mes
- Si cobra $20/hora, ahorra $80-$100/mes en tiempo
- **ROI = 200%** solo en tiempo

**Beneficio Adicional**:
- Retención mejorada (reportes profesionales)
- Cierre de nuevos clientes (demo impresionante)
- Escalabilidad (puede manejar más atletas sin caos)

### ¿Por qué el Bundle ($79/mes) tiene sentido?

**Para el Coach que también organiza torneos locales**:
- Training Core: $49/mes
- Tournament Manager: $49/mes
- Bundle: $79/mes → **Ahorro de $19/mes**

**Beneficio Real**:
- Exporta sus atletas al torneo con 1 clic
- Importa resultados del torneo con 1 clic
- Actualización automática de 1RM oficiales
- **Valor percibido = 10x el precio**

---

## 🔐 Estrategia de Retención (Stickiness)

### 1️⃣ Visualización Adictiva

**Gráfico Estrella**: Evolución de Fuerza Relativa

```
Fuerza Relativa = Kilos Levantados / Peso Corporal

┌─────────────────────────────────────┐
│  SQ: 140kg / 75kg = 1.87 BW        │
│  BP: 100kg / 75kg = 1.33 BW        │
│  DL: 180kg / 75kg = 2.40 BW        │
│                                     │
│     📈 Ver esa línea subir          │
│        = DOPAMINA                   │
└─────────────────────────────────────┘
```

**Efecto Psicológico**: El coach y el atleta se vuelven adictos a ver el progreso visual.

### 2️⃣ Coste de Salida (Data Lock-in)

**Principio**: Mientras más ciclos y torneos registrados, más valiosa es la base de datos.

| Tiempo de Uso | Valor del Historial | Dificultad de Migrar |
|---------------|---------------------|----------------------|
| 1 mes | Bajo | Fácil ✅ |
| 6 meses | Medio | Tedioso 🟡 |
| 1 año | Alto | Doloroso 🔴 |
| 2+ años | **Irreemplazable** | **Imposible** ⛔ |

**Estrategia**:
- Exportación de datos permitida (no ser malvados)
- Pero el valor está en la **interpretación**, no en el CSV raw

### 3️⃣ El Reporte PDF como Marketing Viral

```
┌─────────────────────────────────────────────────────┐
│  FLUJO DE VIRALIDAD ORGÁNICA                       │
└─────────────────────────────────────────────────────┘

1. Coach envía reporte PDF al atleta
2. Atleta lo comparte en Instagram
3. Reporte lleva marca de agua "Powered by IronSystems"
4. Otros atletas preguntan a su coach "¿Por qué no me das reportes así?"
5. Coach no puede dejar el sistema (atletas esperan el reporte)

➜ Network Effect Inverso: Los clientes del coach
  exigen que el coach use tu sistema
```

---

## 🚀 Roadmap de Escalamiento

### Principio de Diseño: Desacoplar Frontend del Backend Progresivamente

```
┌────────────────────────────────────────────────────────────┐
│               EVOLUTION PATH                                │
└────────────────────────────────────────────────────────────┘

   Sheets       →    Hybrid     →    Database   →   Full SaaS
   Engine            App              Migration       Platform
   
   Mes 1-6          Mes 6-12         Año 1           Año 2+
```

### Fase 1: "The Spreadsheet Engine" (Mes 1-6)

**Stack Tecnológico**:
- 🗄️ Backend: Google Sheets
- 💻 Frontend Coach: Google Sheets
- 📱 Frontend Atleta: Google Forms / AppSheet (gratis)
- ⚙️ Lógica: Apps Script

**Objetivo**: Validar la lógica matemática y flujos de usuario

**Métricas de Éxito**:
- [ ] 10 coaches pagando
- [ ] 50+ atletas registrados
- [ ] 3+ torneos gestionados
- [ ] Churn rate < 10%

**Entregables**:
```
✅ Script de auto-regulación
✅ Script de detección de estancamiento
✅ Generador de PDF
✅ Dashboard funcional
✅ Lead magnets operativos
```

### Fase 2: "Hybrid App" (Mes 6-12)

**Stack Tecnológico**:
- 🗄️ Backend: Google Sheets (sigue siendo la DB)
- 💻 Frontend Coach: AppSheet App (white-label)
- 📱 Frontend Atleta: AppSheet App
- ⚙️ Lógica: Apps Script + AppSheet Automation

**Mejoras**:
- ✨ UX superior
- 📲 Notificaciones push
- 🔌 Modo offline
- 🎨 Branding personalizado

**Inversión**: ~$0 (AppSheet tiene plan gratuito generoso)

### Fase 3: "Database Migration" (Año 1)

**Stack Tecnológico**:
- 🗄️ Backend: **Supabase** (PostgreSQL)
- 🔌 Middleware: Apps Script como conector temporal
- 💻 Frontend: Mantener AppSheet O migrar a WebApp (React/Next.js)

**Ventajas**:
- ⚡ Velocidad real
- 👥 Usuarios ilimitados
- 🔒 Seguridad de datos empresarial
- 📊 Queries complejas

**Inversión**: ~$25/mes (Supabase Pro)

**Estrategia de Migración Sin Fricción**:
```javascript
// Dual-write durante transición
function logWorkout(data) {
  writeToSheets(data);  // Sistema viejo
  writeToSupabase(data); // Sistema nuevo
  
  // Después de 1 mes de dual-write:
  // Solo writeToSupabase(data);
}
```

### Fase 4: "Full SaaS" (Año 2)

**Stack Tecnológico**:
- 🗄️ Backend: Supabase + API REST
- 💻 Frontend: Web App completa (Next.js + Tailwind)
- 📱 Mobile: React Native o PWA
- 🔌 API Pública: Permitir integraciones externas

**Nuevas Features**:
- 🏋️ Marcadores electrónicos conectados a Tournament Manager
- 📊 Analytics avanzado con ML
- 🌐 Marketplace de programas de entrenamiento
- 👥 Red social de coaches y atletas

**Monetización Adicional**:
- API Access: $99/mes para torneos grandes
- Whitelabel: $299/mes para federaciones
- Certificaciones: Cursos pagos usando la plataforma

---

## 💰 Modelo de Ingresos

### Estructura de Pricing

| Plan | Precio | Target | Límites |
|------|--------|--------|---------|
| **Free** | $0 | Lead Gen | 3 atletas, 1 mes historial |
| **Coach Pro** | $49/mes | Coaches serios | Ilimitado |
| **Tournament Pro** | $49/mes | Organizadores | 1 evento/mes |
| **Bundle** | $79/mes | Coach + Organizador | Todo incluido |

### Matemática de Escalamiento

#### 🎯 Meta $1,000/mes (Validación)

**Necesitas**: 20 Coaches en plan Pro

**Estrategia de Adquisición**:
1. Venta directa por Instagram a entrenadores con >1000 seguidores
2. Ofrecer "Auditoría de Programación" gratuita
3. Mostrar cómo el sistema detecta fallos en sus excels actuales
4. Demo en vivo del generador de PDF

**Timeline**: 3-4 meses

#### 💪 Meta $5,000/mes (Sostenibilidad)

**Necesitas**: 102 Coaches Pro (o mix con Bundles)

**Estrategias**:
1. **Programa de Referidos**:
   - Coach trae Organizador → Ambos reciben 1 mes gratis
   - Coach trae otro Coach → 20% de comisión recurrente

2. **Directorio de Atletas**:
   - Crear ranking global público
   - SEO play: "Ranking Powerlifting [Ciudad]"
   - Coaches quieren que sus atletas aparezcan

3. **Content Marketing**:
   - YouTube: Análisis de fallas técnicas con data
   - Instagram: Transformaciones con gráficos del sistema
   - Podcast: Entrevistas a coaches exitosos que usan IronSystems

**Timeline**: 8-12 meses

#### 🚀 Meta $10,000/mes (Escala)

**Necesitas**: 200+ usuarios activos

**Estrategias**:
1. **Paid Ads**:
   - Facebook/Instagram Ads dirigidos a:
     - Coaches con certificación NSCA/ISSA
     - Miembros de grupos de powerlifting
     - Asistentes a seminarios de fuerza

2. **Alianzas Estratégicas**:
   - Federaciones locales adoptan Tournament Manager como oficial
   - Descuento corporativo para gyms (licencias múltiples)

3. **Enterprise**:
   - Plan $499/mes para cadenas de gyms
   - Incluye: Branding completo, soporte prioritario, training

**Timeline**: 18-24 meses

---

## 📍 Plan de Ejecución

### ⚠️ REGLA DE ORO: No construyas todo antes de vender

### Fase 0: Validación Pre-Construcción (Semana 1-2)

**Objetivo**: Vender antes de construir

**Acciones**:
```
1. Crear landing page simple
   - Problema que resuelves
   - Demo en video (Loom)
   - Formulario de early access

2. Promocionar en grupos de Facebook/Reddit
   - r/powerlifting
   - Grupos de coaches en español
   - "Estoy construyendo esto, ¿te interesa?"

3. Meta: 50 emails en waiting list
   - Si no llegas a 50 en 2 semanas → Pivotar
   - Si llegas a 50+ → Continuar
```

### Fase 1: MVP (Semana 3-6)

**Construir SOLO**:
1. ✅ Calculadora de 1RM (Lead Magnet #1)
2. ✅ Generador de Reporte PDF básico
3. ✅ Google Sheet con lógica de auto-regulación

**Vender a $9/mes o Lifetime $49**:
- Enviar email a los 50 de waiting list
- Meta: 10 ventas
- Si 10+ personas pagan → Construir ecosistema completo

### Fase 2: Training Core Completo (Mes 2-3)

**Construir**:
- [ ] DB_Athletes, DB_Program, DB_Logs
- [ ] Apps Script: Auto-regulación + Detección estancamiento
- [ ] Dashboard Coach con QUERY()
- [ ] Google Form para atleta

**Lanzar**:
- Plan Free (3 atletas)
- Plan Pro $49/mes
- Onboarding de los 10 early adopters

### Fase 3: Tournament Manager (Mes 4-5)

**Construir**:
- [ ] DB_Entries, Live_Scoring, Public_Display
- [ ] Scripts de ranking automático
- [ ] Lead Magnet #2 (Generador de Reglas)

**Lanzar**:
- Plan Free (20 atletas)
- Plan Pro $49/mes
- Probar en 1 torneo real

### Fase 4: Integración (Mes 6)

**Construir**:
- [ ] Export/Import entre Training Core ↔ Tournament Manager
- [ ] UUID único del atleta
- [ ] Flujo completo Pre-torneo y Post-torneo

**Lanzar**:
- Bundle $79/mes
- Caso de estudio: "Cómo un coach gestionó a 15 atletas en un torneo sin estrés"

---

## 📊 Métricas Clave a Trackear

### Producto
- **Retention Rate**: % usuarios activos mes a mes
  - Meta: >80% mes 1-3, >60% mes 6+
- **Feature Adoption**: % usuarios que usan auto-regulación
  - Meta: >50%
- **Time to Value**: Días hasta primera sesión logueada
  - Meta: <3 días

### Negocio
- **MRR (Monthly Recurring Revenue)**
  - Mes 3: $500
  - Mes 6: $1,000
  - Mes 12: $5,000
- **CAC (Customer Acquisition Cost)**
  - Meta: <$50 con organic
  - Meta: <$100 con paid
- **LTV (Lifetime Value)**
  - Meta: >$500 (10+ meses de retención)

### Growth
- **Virality**: Atletas compartiendo reportes PDF
  - Meta: 1 share cada 5 reportes generados
- **Referral Rate**: % coaches que refieren otro coach
  - Meta: >20%

---

## 🎯 Siguiente Paso AHORA MISMO

### ¿Qué hacer en las próximas 48 horas?

```
┌────────────────────────────────────────────────┐
│  DECISIÓN CRÍTICA                              │
└────────────────────────────────────────────────┘

Opción A: Generar el código Apps Script
         └─ Sistema de detección de estancamiento
         └─ Listo para copiar/pegar

Opción B: Diseñar estructura de columnas
         └─ Master Database en Sheets
         └─ Con nomenclatura y tipos de datos

Opción C: Crear Landing Page de validación
         └─ Carrd.co o Notion
         └─ Probar demanda antes de construir

➜ RECOMENDACIÓN: Opción C primero
  Luego A, luego B
```

---

## 📚 Recursos Necesarios

### Tecnología
- ✅ Google Account (Sheets + Apps Script) - **GRATIS**
- ✅ Dominio propio - **$12/año**
- ✅ Carrd.co para landing - **$19/año**
- ❌ NO necesitas: Servidor, database, hosting (por ahora)

### Skills Requeridos
- 🟢 Google Sheets (nivel intermedio)
- 🟡 Apps Script / JavaScript básico
- 🟢 Fórmulas (QUERY, FILTER, ARRAYFORMULA)
- 🟡 Product design / UX thinking
- 🟢 Ventas directas / Instagram DM

### Tiempo Estimado
- **Validación**: 2 semanas (2-3 hrs/día)
- **MVP**: 4 semanas (3-4 hrs/día)
- **Lanzamiento completo**: 3 meses (4-5 hrs/día)

---

## ✅ Checklist Final

### Pre-Launch
- [ ] Definir nombre comercial y dominio
- [ ] Crear landing page de validación
- [ ] Conseguir 50 emails en waiting list
- [ ] Validar willingness to pay ($9 o $49)

### MVP
- [ ] Construir Lead Magnet #1 (Calculadora)
- [ ] Construir generador de PDF básico
- [ ] Setup Google Sheets estructura
- [ ] Escribir Apps Script core

### Launch
- [ ] Onboarding de 10 early adopters
- [ ] Documentación de uso
- [ ] Video tutorial
- [ ] Canal de soporte (WhatsApp/Telegram)

### Growth
- [ ] Caso de estudio documentado
- [ ] Testimonios en video
- [ ] Programa de referidos
- [ ] Content calendar (Instagram/YouTube)

---

## 🎬 Conclusión

**IronSystems no es una planilla bonita.**

Es un **motor de decisiones deportivas** que:
- ✅ Ahorra tiempo al coach
- ✅ Mejora resultados del atleta
- ✅ Profesionaliza la industria
- ✅ Genera data valiosa
- ✅ Escala sin límite

**La pregunta no es si funcionará.**

La pregunta es: **¿Cuándo empezamos?**

---

> **"El mejor momento para plantar un árbol fue hace 20 años. El segundo mejor momento es ahora."**
> 
> — Proverbio chino aplicado a SaaS 🌱

---

**¿Listo para generar el código Apps Script o diseñar la estructura de la base de datos?**

**Di la palabra y arrancamos. 🚀**
