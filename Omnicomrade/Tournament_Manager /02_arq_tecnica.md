# IronSystems - Arquitectura Técnica
## Tournament Manager - Documentación de Implementación

---

## Stack Tecnológico

### Core
- **Backend:** Google Apps Script (JavaScript ES5/ES6)
- **Database:** Google Sheets (columnar storage)
- **Frontend:** HTML Service + CSS + Vanilla JS
- **Analytics:** Looker Studio (dashboards)
- **Hosting:** Apps Script Web App (serverless)

### Ventajas del Stack
✅ **Costo:** $0 (hasta 20,000 requests/day)
✅ **Deployment:** 1-click, sin CI/CD
✅ **Escalabilidad:** Hasta 50 usuarios concurrentes
✅ **Mantenimiento:** Auto-updates via Google
✅ **Colaboración:** Real-time multi-user

### Limitaciones Conocidas
⚠️ **Execution time:** 6 min max por script
⚠️ **Cell limit:** 10M cells por spreadsheet
⚠️ **Quotas:** 20k triggers/day (free tier)
⚠️ **Storage:** 15GB Drive (expandible)

---

## Estructura de Datos (Schema)

### 1. DB_Tournaments
```javascript
const TOURNAMENTS_SCHEMA = {
  columns: [
    'TournamentID',        // STRING (UUID)
    'Name',               // STRING
    'Date',               // DATE
    'Location',           // STRING
    'OrganizerName',      // STRING
    'OrganizerEmail',     // EMAIL
    'Status',             // ENUM: DRAFT|ACTIVE|CLOSED
    'CreatedAt',          // TIMESTAMP
    'WeightClassesJSON'   // JSON string
  ],
  indexes: ['TournamentID'],
  sample: [
    'a1b2c3d4-...',
    'Spring Classic 2026',
    '2026-03-15',
    'NYC Athletic Club',
    'John Smith',
    'john@example.com',
    'ACTIVE',
    '2026-02-01 10:00:00',
    '{"M":[59,66,74,83,93,105,120,"120+"],"F":[47,52,57,63,69,76,84,"84+"]}'
  ]
};
```

### 2. DB_Entries (Athlete Registrations)
```javascript
const ENTRIES_SCHEMA = {
  columns: [
    'EntryID',           // STRING (UUID)
    'TournamentID',      // FK -> DB_Tournaments
    'AthleteName',       // STRING
    'Sex',              // ENUM: M|F
    'DOB',              // DATE (optional)
    'BodyWeight',       // DECIMAL(3,1)
    'WeightClass',      // CALCULATED
    'Division',         // ENUM: Open|Junior|Master
    'Team',             // STRING (optional)
    'SQ_Opener',        // DECIMAL(4,1)
    'BP_Opener',        // DECIMAL(4,1)
    'DL_Opener',        // DECIMAL(4,1)
    'RegistrationTime', // TIMESTAMP
    'Status'            // ENUM: PENDING|CONFIRMED|SCRATCHED
  ],
  validations: {
    BodyWeight: (val) => val > 30 && val < 200,
    SQ_Opener: (val) => val > 0 && val < 500,
    BP_Opener: (val) => val > 0 && val < 400,
    DL_Opener: (val) => val > 0 && val < 500
  }
};
```

### 3. DB_Attempts (Live Competition Log)
```javascript
const ATTEMPTS_SCHEMA = {
  columns: [
    'AttemptID',        // STRING (UUID)
    'TournamentID',     // FK
    'EntryID',          // FK
    'AthleteName',      // DENORM (for speed)
    'Lift',             // ENUM: SQ|BP|DL
    'AttemptNumber',    // INT: 1|2|3
    'Weight',           // DECIMAL(4,1)
    'Result',           // ENUM: GOOD|NO_LIFT|PASS|'' (pending)
    'Order',            // INT (lifting order)
    'CompletedAt'       // TIMESTAMP (null if pending)
  ],
  indexes: ['TournamentID', 'Lift', 'AttemptNumber', 'Order']
};
```

### 4. DB_Results (Final Rankings)
```javascript
const RESULTS_SCHEMA = {
  columns: [
    'TournamentID',
    'EntryID',
    'AthleteName',
    'Sex',
    'BodyWeight',
    'WeightClass',
    'Division',
    'Team',
    'BestSQ',           // Best squat from attempts
    'BestBP',           // Best bench
    'BestDL',           // Best deadlift
    'Total',            // SUM of bests
    'DOTS',             // Calculated
    'Wilks',            // Calculated (legacy)
    'PlaceOverall',     // Rank by total
    'PlaceByClass',     // Rank within weight class
    'PlaceByDivision'   // Rank within division
  ],
  generated: true // Auto-populated by calculateResults()
};
```

### 5. CONFIG (System Settings)
```javascript
const CONFIG_SCHEMA = {
  columns: ['Key', 'Value'],
  keyValuePairs: {
    'TOURNAMENT_ID': '<active tournament UUID>',
    'CURRENT_LIFT': 'SQ|BP|DL',
    'CURRENT_ROUND': '1|2|3',
    'AUTO_INCREMENT_ORDER': 'TRUE|FALSE',
    'PLATE_LOADING_KG': '25,20,15,10,5,2.5,1.25',
    'BAR_WEIGHT_KG': '20'
  }
};
```

---

## Core Functions (Apps Script)

### Configuration Management
```javascript
/**
 * Get configuration value by key
 * @param {string} key - Configuration key
 * @returns {string|null} Value or null if not found
 */
function getConfig(key) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('CONFIG');
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === key) return data[i][1];
  }
  return null;
}

/**
 * Set configuration value
 * @param {string} key - Configuration key
 * @param {string} value - New value
 */
function setConfig(key, value) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('CONFIG');
  const data = sheet.getDataRange().getValues();
  
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === key) {
      sheet.getRange(i + 1, 2).setValue(value);
      return;
    }
  }
  sheet.appendRow([key, value]);
}
```

### Weight Class Calculation
```javascript
/**
 * Calculate weight class based on body weight
 * @param {string} sex - 'M' or 'F'
 * @param {number} bodyWeight - Body weight in kg
 * @returns {string} Weight class (e.g., '83' or '120+')
 */
function calculateWeightClass(sex, bodyWeight) {
  const classes = {
    M: [59, 66, 74, 83, 93, 105, 120],
    F: [47, 52, 57, 63, 69, 76, 84]
  };
  
  const limits = classes[sex];
  if (!limits) throw new Error('Invalid sex: must be M or F');
  
  for (let i = 0; i < limits.length; i++) {
    if (bodyWeight <= limits[i]) return limits[i].toString();
  }
  
  return sex === 'M' ? '120+' : '84+';
}
```

### DOTS Calculation
```javascript
/**
 * Calculate DOTS score (IPF standard)
 * @param {string} sex - 'M' or 'F'
 * @param {number} bodyWeight - Body weight in kg
 * @param {number} total - Total lifted in kg
 * @returns {number} DOTS score
 */
function calculateDOTS(sex, bodyWeight, total) {
  // DOTS coefficients (2024 version)
  const coefficients = {
    M: {
      a: -0.0000010930,
      b: 0.0007391293,
      c: -0.1918759221,
      d: 24.0900756,
      e: -307.75076
    },
    F: {
      a: -0.0000010706,
      b: 0.0005158568,
      c: -0.1126655495,
      d: 13.6175032,
      e: -57.96288
    }
  };
  
  const c = coefficients[sex];
  if (!c) throw new Error('Invalid sex for DOTS calculation');
  
  const bw = bodyWeight;
  const denominator = 
    c.a * Math.pow(bw, 4) +
    c.b * Math.pow(bw, 3) +
    c.c * Math.pow(bw, 2) +
    c.d * bw + c.e;
  
  const dots = (500 / denominator) * total;
  return Math.round(dots * 100) / 100; // 2 decimals
}
```

### Wilks Calculation
```javascript
/**
 * Calculate Wilks score (legacy standard, still widely used)
 * @param {string} sex - 'M' or 'F'
 * @param {number} bodyWeight - Body weight in kg
 * @param {number} total - Total lifted in kg
 * @returns {number} Wilks score
 */
function calculateWilks(sex, bodyWeight, total) {
  const coefficients = {
    M: [
      -216.0475144,
      16.2606339,
      -0.002388645,
      -0.00113732,
      7.01863E-06,
      -1.291E-08
    ],
    F: [
      594.31747775582,
      -27.23842536447,
      0.82112226871,
      -0.00930733913,
      0.00004731582,
      -0.00000009054
    ]
  };
  
  const c = coefficients[sex];
  const bw = bodyWeight;
  
  const denominator = 
    c[0] + c[1] * bw + c[2] * Math.pow(bw, 2) +
    c[3] * Math.pow(bw, 3) + c[4] * Math.pow(bw, 4) +
    c[5] * Math.pow(bw, 5);
  
  const wilks = (total * 500) / denominator;
  return Math.round(wilks * 100) / 100;
}
```

### Attempt Ordering Algorithm
```javascript
/**
 * Generate lifting order for a round
 * Rules: 
 * 1. Lighter weights go first
 * 2. If tied, earlier attempt number goes first
 * 3. If still tied, maintain previous order
 * 
 * @param {string} lift - 'SQ', 'BP', or 'DL'
 * @param {number} round - 1, 2, or 3
 */
function generateAttemptOrder(lift, round) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const entriesSheet = ss.getSheetByName('DB_Entries');
  const attemptsSheet = ss.getSheetByName('DB_Attempts');
  const tournamentId = getConfig('TOURNAMENT_ID');
  
  // Get all confirmed entries
  const entries = entriesSheet.getDataRange().getValues();
  const pendingAthletes = [];
  
  for (let i = 1; i < entries.length; i++) {
    const row = entries[i];
    if (row[1] !== tournamentId || row[13] !== 'CONFIRMED') continue;
    
    const entryId = row[0];
    const athleteName = row[2];
    
    // Determine weight for this attempt
    let weight;
    if (round === 1) {
      // Use opener
      const openerIndex = lift === 'SQ' ? 9 : lift === 'BP' ? 10 : 11;
      weight = row[openerIndex];
    } else {
      // Get last successful attempt + standard increment
      weight = getNextAttemptWeight(entryId, lift, round);
      if (weight === null) continue; // Athlete bombed out
    }
    
    pendingAthletes.push({
      entryId: entryId,
      athleteName: athleteName,
      weight: weight
    });
  }
  
  // Sort by weight (ascending), then by name (tiebreaker)
  pendingAthletes.sort((a, b) => {
    if (a.weight !== b.weight) return a.weight - b.weight;
    return a.athleteName.localeCompare(b.athleteName);
  });
  
  // Write to DB_Attempts with order
  pendingAthletes.forEach((athlete, index) => {
    attemptsSheet.appendRow([
      generateUUID(),
      tournamentId,
      athlete.entryId,
      athlete.athleteName,
      lift,
      round,
      athlete.weight,
      '', // Result (pending)
      index + 1, // Order
      '' // CompletedAt
    ]);
  });
  
  Logger.log(`Generated ${pendingAthletes.length} attempts for ${lift} Round ${round}`);
}

/**
 * Get next attempt weight based on previous performance
 * @param {string} entryId - Athlete's entry ID
 * @param {string} lift - Current lift
 * @param {number} round - Current round (2 or 3)
 * @returns {number|null} Weight in kg, or null if bombed out
 */
function getNextAttemptWeight(entryId, lift, round) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const attemptsSheet = ss.getSheetByName('DB_Attempts');
  const data = attemptsSheet.getDataRange().getValues();
  
  // Find last successful attempt
  let lastGoodWeight = null;
  for (let i = data.length - 1; i >= 1; i--) {
    const row = data[i];
    if (row[2] === entryId && row[4] === lift && row[7] === 'GOOD') {
      lastGoodWeight = row[6];
      break;
    }
  }
  
  if (lastGoodWeight === null) {
    // Athlete has no successful attempts this lift = bombed out
    return null;
  }
  
  // Standard increment: 2.5kg for rounds 2-3
  return lastGoodWeight + 2.5;
}
```

### Plate Loading Calculator
```javascript
/**
 * Calculate which plates to load on the bar
 * @param {number} targetWeight - Target weight in kg
 * @returns {Object} Plates per side and display string
 */
function calculatePlateLoading(targetWeight) {
  const barWeight = parseFloat(getConfig('BAR_WEIGHT_KG') || '20');
  const plateString = getConfig('PLATE_LOADING_KG') || '25,20,15,10,5,2.5,1.25';
  const availablePlates = plateString.split(',').map(p => parseFloat(p));
  
  const perSide = (targetWeight - barWeight) / 2;
  
  if (perSide <= 0) {
    return {
      perSide: [],
      display: 'Empty bar'
    };
  }
  
  let remaining = perSide;
  const loading = [];
  
  for (const plate of availablePlates) {
    while (remaining >= plate - 0.01) { // Float tolerance
      loading.push(plate);
      remaining -= plate;
    }
  }
  
  return {
    perSide: loading,
    display: loading.length > 0 
      ? loading.map(p => `${p}kg`).join(' + ')
      : 'Empty bar',
    exactMatch: Math.abs(remaining) < 0.01
  };
}

// Example usage in UI:
function showPlateLoading(weight) {
  const plates = calculatePlateLoading(weight);
  return `Load per side: ${plates.display}`;
}
```

---

## Deployment Guide

### Step 1: Prepare Spreadsheet
1. Create new Google Sheet
2. Rename to: `IronSystems_Tournament_Template`
3. Create sheets with exact names:
   - `DB_Tournaments`
   - `DB_Entries`
   - `DB_Attempts`
   - `DB_Results`
   - `CONFIG`
4. Add headers as per schema above

### Step 2: Apps Script Setup
1. Extensions → Apps Script
2. Rename project: `IronSystems Tournament Manager`
3. Create files:
   - `Code.gs` (main logic)
   - `RegistrationForm.html`
   - `AttemptPanel.html`
   - `PublicResults.html`
4. Paste code from roadmap
5. Save (Ctrl+S)

### Step 3: Set Triggers
```javascript
// Manual: Edit → Current project's triggers → Add Trigger
// Or via code:
function setupTriggers() {
  // Delete existing triggers
  const triggers = ScriptApp.getProjectTriggers();
  triggers.forEach(trigger => ScriptApp.deleteTrigger(trigger));
  
  // Create onOpen trigger
  ScriptApp.newTrigger('onOpen')
    .forSpreadsheet(SpreadsheetApp.getActiveSpreadsheet())
    .onOpen()
    .create();
}
```

### Step 4: Deploy Web App
1. Deploy → New deployment
2. Type: Web app
3. Description: "Tournament Results v1.0"
4. Execute as: **Me** (important!)
5. Who has access: **Anyone**
6. Deploy
7. Copy URL: `https://script.google.com/macros/s/XXXXX/exec`

### Step 5: Test
1. Open spreadsheet
2. Menu: Tournament Manager → New Tournament
3. Fill form → Create
4. Menu: Register Athlete (add 3-5 athletes)
5. Menu: Start Competition
6. Menu: Attempt Panel → Record lifts
7. Menu: Generate Results
8. Open web app URL → See results

---

## Performance Optimization

### 1. Batch Operations
```javascript
// ❌ BAD: Multiple getRange() calls
for (let i = 0; i < athletes.length; i++) {
  sheet.getRange(i + 2, 1).setValue(athletes[i].name);
}

// ✅ GOOD: Single setValues() call
const values = athletes.map(a => [a.name]);
sheet.getRange(2, 1, values.length, 1).setValues(values);
```

### 2. Cache Heavy Reads
```javascript
// Cache tournament data
const CACHE_DURATION = 300; // 5 minutes

function getCachedTournament(tournamentId) {
  const cache = CacheService.getScriptCache();
  const cached = cache.get('tournament_' + tournamentId);
  
  if (cached) return JSON.parse(cached);
  
  const tournament = fetchTournamentData(tournamentId);
  cache.put('tournament_' + tournamentId, JSON.stringify(tournament), CACHE_DURATION);
  return tournament;
}
```

### 3. Limit Data Range
```javascript
// ❌ BAD: Read entire sheet
const data = sheet.getDataRange().getValues();

// ✅ GOOD: Read only needed rows
const lastRow = sheet.getLastRow();
const data = sheet.getRange(1, 1, lastRow, 14).getValues();
```

---

## Security Considerations

### 1. Protect Critical Sheets
```javascript
function protectSheets() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const criticalSheets = ['DB_Tournaments', 'DB_Results', 'CONFIG'];
  
  criticalSheets.forEach(sheetName => {
    const sheet = ss.getSheetByName(sheetName);
    const protection = sheet.protect();
    protection.setDescription(`Protected: ${sheetName}`);
    
    // Only owner can edit
    protection.removeEditors(protection.getEditors());
    protection.addEditor(Session.getEffectiveUser().getEmail());
  });
}
```

### 2. Input Validation
```javascript
function validateAthleteData(data) {
  const errors = [];
  
  if (!data.athleteName || data.athleteName.trim().length < 2) {
    errors.push('Name must be at least 2 characters');
  }
  
  if (!['M', 'F'].includes(data.sex)) {
    errors.push('Sex must be M or F');
  }
  
  if (data.bodyWeight < 30 || data.bodyWeight > 200) {
    errors.push('Body weight must be between 30-200kg');
  }
  
  if (data.sqOpener <= 0 || data.sqOpener > 500) {
    errors.push('Squat opener must be 0-500kg');
  }
  
  if (errors.length > 0) {
    throw new Error('Validation failed:\n' + errors.join('\n'));
  }
}
```

### 3. Rate Limiting (for public web app)
```javascript
function checkRateLimit() {
  const cache = CacheService.getScriptCache();
  const userKey = 'ratelimit_' + Session.getTemporaryActiveUserKey();
  const count = parseInt(cache.get(userKey) || '0');
  
  if (count > 100) {
    throw new Error('Rate limit exceeded. Try again later.');
  }
  
  cache.put(userKey, (count + 1).toString(), 3600); // 1 hour window
}
```

---

## Testing Strategy

### Unit Tests
```javascript
function testDOTSCalculation() {
  // Known values (IPF calculator)
  const result1 = calculateDOTS('M', 83, 600);
  const expected1 = 403.16; // Approximate
  
  if (Math.abs(result1 - expected1) > 0.5) {
    throw new Error(`DOTS test failed: ${result1} vs ${expected1}`);
  }
  
  Logger.log('✅ DOTS calculation test passed');
}

function testWeightClass() {
  const tests = [
    {sex: 'M', bw: 82.5, expected: '83'},
    {sex: 'M', bw: 83.0, expected: '83'},
    {sex: 'M', bw: 125, expected: '120+'},
    {sex: 'F', bw: 63.2, expected: '63'}
  ];
  
  tests.forEach(test => {
    const result = calculateWeightClass(test.sex, test.bw);
    if (result !== test.expected) {
      throw new Error(`Weight class test failed: ${result} vs ${test.expected}`);
    }
  });
  
  Logger.log('✅ Weight class tests passed');
}

function runAllTests() {
  testDOTSCalculation();
  testWeightClass();
  Logger.log('✅ All tests passed');
}
```

### Integration Tests
```javascript
function testFullTournamentFlow() {
  // Create tournament
  const tournamentId = createNewTournament({
    name: 'Test Meet',
    date: '2026-12-31',
    location: 'Test Gym',
    organizerName: 'Tester',
    organizerEmail: 'test@example.com'
  });
  
  // Register athletes
  const athletes = [
    {name: 'Athlete 1', sex: 'M', bw: 82.5, sq: 200, bp: 140, dl: 240},
    {name: 'Athlete 2', sex: 'F', bw: 63.2, sq: 120, bp: 70, dl: 150}
  ];
  
  athletes.forEach(a => {
    registerAthlete({
      tournamentId: tournamentId,
      athleteName: a.name,
      sex: a.sex,
      bodyWeight: a.bw,
      division: 'Open',
      sqOpener: a.sq,
      bpOpener: a.bp,
      dlOpener: a.dl
    });
  });
  
  // Start competition
  startCompetition();
  
  // Simulate attempts (simplified)
  generateAttemptOrder('SQ', 1);
  
  Logger.log('✅ Integration test completed');
}
```

---

## Troubleshooting

### Common Issues

**Issue: "Script timeout" during results calculation**
```javascript
// Solution: Process in batches
function calculateResultsBatched() {
  const batchSize = 50;
  const entries = getConfirmedEntries();
  
  for (let i = 0; i < entries.length; i += batchSize) {
    const batch = entries.slice(i, i + batchSize);
    processBatch(batch);
    Utilities.sleep(1000); // Prevent quota exhaustion
  }
}
```

**Issue: "Concurrent edit" errors**
```javascript
// Solution: Use LockService
function safeBatchWrite(data) {
  const lock = LockService.getScriptLock();
  try {
    lock.waitLock(10000); // Wait up to 10s
    sheet.getRange(2, 1, data.length, data[0].length).setValues(data);
  } finally {
    lock.releaseLock();
  }
}
```

**Issue: Web app returns blank page**
```javascript
// Check: Execute as "Me" not "User accessing the web app"
// Check: Access set to "Anyone"
// Check: All HTML files saved in Apps Script editor
```

---

## Next: Advanced Features (Post-V1)

1. **Video Upload Integration**
   - YouTube API for private unlisted uploads
   - Embed videos in attempt panel
   - Automatic highlight clips

2. **Historical Database**
   - Cross-tournament athlete tracking
   - PR progression charts
   - Best lifts leaderboard

3. **Team Scoring**
   - Team registration
   - Team totals calculation
   - Team leaderboard

4. **Mobile PWA**
   - Offline mode for judges
   - Push notifications
   - Native app feel

5. **Advanced Analytics**
   - Success rate by attempt
   - Weight class competitiveness
   - Strength standards comparison

---

**Ready to code? Start with Week 1, Day 1 tasks from the roadmap! 🚀**
