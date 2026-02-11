El Hechizo de Ranking de Gremios (Data Analytics)
Este es el script más avanzado. Crea una hoja nueva llamada "Guerra de Clanes" y te dice qué equipo es el más fuerte y cuál es el más arriesgado (basado en la tasa de fallos).

Métricas que calcula:

Promedio Total: Fuerza bruta del equipo.

Tasa de Fallo: ¿Cuántos intentos desperdician? (Lore: Los Iron Conjurers deberían tener una tasa alta).

JavaScript
function 3_generarRankingGremios() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getActiveSheet();
  const data = sheet.getDataRange().getValues();
  
  // Objeto para acumular estadísticas por equipo
  let guildStats = {};
  
  // Empezamos en i=1 para saltar encabezados
  for (let i = 1; i < data.length; i++) {
    let team = data[i][7]; // Columna H: Equipo
    let total = data[i][17]; // Columna R: Total (Calculado en script 1)
    
    // Contar intentos fallidos en esta fila (SQ, BP, DL) columnas 8 a 16
    let attempts = 0;
    let fails = 0;
    for (let j = 8; j <= 16; j++) {
      let val = data[i][j];
      if (val !== "") { // Si no está vacío
        attempts++;
        if (val < 0) fails++;
      }
    }

    if (!guildStats[team]) {
      guildStats[team] = { 
        lifters: 0, 
        sumTotal: 0, 
        totalAttempts: 0, 
        totalFails: 0 
      };
    }
    
    // Solo sumamos al promedio si hizo total (evita castigar por lesiones graves)
    // O puedes incluir los 0 para castigar a los Daily Max Zealots
    guildStats[team].lifters++;
    guildStats[team].sumTotal += total;
    guildStats[team].totalAttempts += attempts;
    guildStats[team].totalFails += fails;
  }

  // --- PREPARAR SALIDA ---
  let output = [];
  output.push(["Ranking", "Equipo / Gremio", "Total Promedio (kg)", "Tasa de Fallos (%)", "Descripción Táctica"]);

  let rank = 1;
  // Convertir objeto a array para ordenar
  let sortedGuilds = Object.keys(guildStats).map(key => {
    return { name: key, ...guildStats[key] };
  }).sort((a, b) => (b.sumTotal / b.lifters) - (a.sumTotal / a.lifters)); // Ordenar por promedio descendente

  sortedGuilds.forEach(guild => {
    let avgTotal = (guild.sumTotal / guild.lifters).toFixed(1);
    let failRate = ((guild.totalFails / guild.totalAttempts) * 100).toFixed(1);
    
    // Generar descripción basada en datos (Lore dinámico)
    let desc = "";
    if (failRate > 30) desc = "⚠️ Arriesgados (High Risk)";
    else if (failRate < 10) desc = "🛡️ Conservadores (Safe)";
    else desc = "⚖️ Equilibrados";

    output.push([rank++, guild.name, avgTotal, failRate + "%", desc]);
  });

  // --- CREAR/ACTUALIZAR HOJA DE RANKING ---
  let rankingSheet = ss.getSheetByName("Guerra de Clanes");
  if (!rankingSheet) {
    rankingSheet = ss.insertSheet("Guerra de Clanes");
  } else {
    rankingSheet.clear(); // Limpiar datos viejos
  }
  
  rankingSheet.getRange(1, 1, output.length, 5).setValues(output);
  
  // Dar formato bonito
  rankingSheet.getRange(1, 1, 1, 5).setFontWeight("bold").setBackground("#d9d9d9");
  rankingSheet.autoResizeColumns(1, 5);
}
