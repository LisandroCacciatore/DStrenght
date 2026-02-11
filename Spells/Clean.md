function 1_procesarTorneo() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  // Asumimos que hay encabezados, empezamos en fila 2
  const lastRow = sheet.getLastRow();
  const dataRange = sheet.getRange(2, 1, lastRow - 1, 18); // Ajusta columnas si necesitas
  const data = dataRange.getValues();
  
  // Índices de las columnas (basado en el CSV V2.0):
  // SQ: 8,9,10 | BP: 11,12,13 | DL: 14,15,16
  const cols = { sq: [8, 9, 10], bp: [11, 12, 13], dl: [14, 15, 16] };
  
  // Arrays para guardar colores y resultados para escribir en bloque al final (más rápido)
  let fontColors = dataRange.getFontColors();
  let totals = [];

  for (let i = 0; i < data.length; i++) {
    let row = data[i];
    let bestSQ = 0, bestBP = 0, bestDL = 0;
    
    // --- LÓGICA DE SQUAT ---
    bestSQ = procesarMovimiento(row, cols.sq, i, fontColors);
    
    // --- LÓGICA DE BENCH PRESS ---
    bestBP = procesarMovimiento(row, cols.bp, i, fontColors);
    
    // --- LÓGICA DE DEADLIFT ---
    bestDL = procesarMovimiento(row, cols.dl, i, fontColors);

    // CÁLCULO DEL TOTAL (Regla: Si fallas los 3 de un tipo, Total = 0)
    let total = 0;
    if (bestSQ > 0 && bestBP > 0 && bestDL > 0) {
      total = bestSQ + bestBP + bestDL;
    }
    totals.push([total]);
  }

  // Escribir colores y totales de una sola vez
  dataRange.setFontColors(fontColors);
  
  // Asumimos que la columna 18 (R) será para el TOTAL
  sheet.getRange(2, 18, totals.length, 1).setValues(totals);
  sheet.getRange(1, 18).setValue("TOTAL (kg)"); // Poner título
}

// Función auxiliar para no repetir código
function procesarMovimiento(row, indices, rowIndex, colorMatrix) {
  let maxLift = 0;
  let bombedOut = true; // Asumimos que falló todo hasta demostrar lo contrario

  indices.forEach(colIndex => {
    let val = row[colIndex];
    if (val < 0) {
      colorMatrix[rowIndex][colIndex] = "#FF0000"; // Rojo fallido
    } else if (val > 0) {
      colorMatrix[rowIndex][colIndex] = "#2E8B57"; // Verde válido
      maxLift = Math.max(maxLift, val);
      bombedOut = false;
    }
    // Si es 0 o vacío, lo dejamos negro
  });

  return bombedOut ? 0 : maxLift;
}
