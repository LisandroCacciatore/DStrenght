function guardarEnHistorial() {
     const ss = SpreadsheetApp.getActiveSpreadsheet();
     const dataSheet = ss.getSheets()[0];
     const histSheet = ss.getSheetByName('HISTORIAL') || ss.insertSheet('HISTORIAL');
     
     // Pedir nombre del torneo
     const ui = SpreadsheetApp.getUi();
     const response = ui.prompt('Nombre del Torneo', 
       'Ej: Copa Primavera 2026', ui.ButtonSet.OK_CANCEL);
     
     if (response.getSelectedButton() != ui.Button.OK) return;
     
     const nombreTorneo = response.getResponseText();
     const fecha = new Date();
     
     // Leer datos procesados
     const data = dataSheet.getRange(2, 1, dataSheet.getLastRow()-1, 18).getValues();
     
     data.forEach(row => {
       // Solo guardar si tiene Total > 0
       if (row[17] > 0) {
         histSheet.appendRow([
           fecha,
           nombreTorneo,
           row[0], // Nombre
           row[1], // Edad
           row[2], // Peso
           row[3], // Cat_Edad
           row[4], // Cat_Peso
           Math.max(row[8], row[9], row[10]), // Mejor Squat
           Math.max(row[11], row[12], row[13]), // Mejor Bench
           Math.max(row[14], row[15], row[16]), // Mejor Deadlift
           row[17], // Total
           row[7], // Equipo
           // Ranking se puede calcular después
         ]);
       }
     });
     
     ui.alert('✅ Torneo guardado: ' + nombreTorneo);
   }
