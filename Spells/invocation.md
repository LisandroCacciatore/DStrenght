/**
 * ------------------------------------------------------------------
 * HECHIZO DE INVOCACIÓN DE MENÚ
 * Esto crea un botón en la barra de herramientas de tu Sheet.
 * ------------------------------------------------------------------
 */
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu('🏆 Torneo Manager')
      .addItem('1. Calcular Totales y Colorear Fallos', 'procesarTorneo')
      .addItem('2. Clasificar por Edad/Peso', 'clasificarLevantadores')
      .addSeparator()
      .addItem('3. Generar Guerra de Clanes', 'generarRankingGremios')
      .addToUi();
}
