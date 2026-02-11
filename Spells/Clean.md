/**
 * ------------------------------------------------------------------
 * HECHIZO 1: LIMPIEZA Y CÁLCULO
 * Pone rojos los fallos, calcula el mejor intento y suma el Total.
 * ------------------------------------------------------------------
 */
function procesarTorneo() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const lastRow = sheet.getLastRow();
  
  if (lastRow < 2) return; // No hay datos

  // Leemos desde la fila 2 hasta la última, columna A(1) hasta R(18)
  const dataRange = sheet.getRange(2, 1, lastRow - 1, 18); 
  const data = dataRange.getValues();
  
  // Índices de columnas (basado en el Array, empieza en 0)
  // SQ: Col I,J,K (Indices 8,9,10)
  // BP: Col L,M,N (Indices 11,12,13)
  // DL: Col O,P,Q (Indices 14,15,16)
  const cols = { sq: [8, 9, 10], bp: [11, 12, 13], dl: [14, 15, 16] };
  
  let fontColors = dataRange.getFontColors();
  let totals = [];

  for (let i = 0; i < data.length; i++) {
    let row = data[i];
    
    // Procesar cada movimiento usando la función auxiliar
    let resSQ = procesarIntento(row, cols.sq, i, fontColors);
    let resBP = procesarIntento(row, cols.bp, i, fontColors);
    let resDL = procesarIntento(row, cols.dl, i, fontColors);

    // Obtener los mejores levantamientos validos
    let bestSQ = resSQ.max; 
    let bestBP = resBP.max;
    let bestDL = resDL.max;

    // Calcular Total:
    // Opción estricta: Si te blanqueas en uno (bombed=true), el total es 0.
    let total = 0;
    if (!resSQ.bombed && !resBP.bombed && !resDL.bombed) {
      total = bestSQ + bestBP + bestDL;
    }
    totals.push([total]);
  }

  // Aplicar colores y escribir totales
  dataRange.setFontColors(fontColors);
  sheet.getRange(2, 18, totals.length, 1).setValues(totals); // Columna 18 es R
  sheet.getRange(1, 18).setValue("TOTAL (kg)").setFontWeight("bold");
}

/**
 * Función auxiliar para colorear celdas y buscar el máximo válido.
 * Retorna objeto { max: numero, bombed: boolean }
 */
function procesarIntento(row, indices, rowIndex, colorMatrix) {
  let maxLift = 0;
  let validCount = 0;

  indices.forEach(colIndex => {
    let val = row[colIndex];
    if (val < 0) {
      colorMatrix[rowIndex][colIndex] = "#FF0000"; // Rojo fallo
    } else if (val > 0) {
      colorMatrix[rowIndex][colIndex] = "#2E8B57"; // Verde válido
      maxLift = Math.max(maxLift, val);
      validCount++;
    }
    // Si es 0 o vacío se ignora
  });

  // 'bombed' es true si intentó levantar (hay datos en el 1er intento) pero no metió ninguno válido
  let hasData = (row[indices[0]] !== "" && row[indices[0]] !== undefined);
  return { max: maxLift, bombed: (hasData && validCount === 0) }; 
}
