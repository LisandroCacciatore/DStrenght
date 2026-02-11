---

## 1. El Mapa del Poder: Geografía y Biomas de Datos

Cada región no solo es un lugar, es un **ecosistema de entrenamiento** que afecta el promedio de los levantadores.

| Región | Bioma Estadístico | Lore de Entrenamiento |
| --- | --- | --- |
| **Frosthold** | *High Volume / Low Variance* | En el norte gélido, el cuerpo se calienta lento. Sus métodos (basados en **Sheiko**) priorizan el volumen masivo. Verás que casi no tienen nulos, pero sus saltos entre intentos son aburridamente constantes. |
| **Ironforge** | *High Intensity / High Mean* | Las forjas nunca descansan. Aquí nació la **Legión Lineal**. Son predecibles: si el primer intento es bueno, el segundo será exactamente 10kg más. Si falla el tercero, es por fatiga mecánica, no técnica. |
| **Shadowfen** | *The Outlier Swamp* | Un lugar pantanoso donde los **Daily Max Zealots** entrenan a vida o muerte. En tus datos, Shadowfen tendrá la mayor cantidad de **"Bomb-outs"** (0 total). O rompen el récord mundial o no terminan el torneo. |
| **Sunspire** | *Efficiency & Precision* | Hogar de los **RPE Oracles**. Sus levantadores tienen el peso corporal más bajo pero la mayor fuerza relativa. En AppScript, verás que sus 3ros intentos suelen ser válidos por apenas 2.5kg. |

---

## 2. Los "Scrolls of Progress" (Métricas de Script)

Para practicar AppScript, no pienses en "sumar columnas", piensa en **descifrar estos pergaminos**:

### A. El Índice de "Coraje del Zealot" (Success Rate)

Los **Daily Max Zealots** de Shadowfen creen que un intento válido es un intento desperdiciado (porque podrías haber tirado más).

* **Tu Script:** Debe calcular el ratio de `Intentos Negativos / Intentos Totales`.
* **Lore:** Un ratio de fallos del 33% es considerado "Honorífico". Si es del 0%, el levantador es exiliado por "cobardía estadística".

### B. El "Efecto de la Forja" (Linear Fatigue)

La **Linear Legion** de Ironforge entrena con progresiones rígidas.

* **Tu Script:** Calcula la diferencia entre `SQ_1`, `SQ_2` y `SQ_3`.
* **Lore:** Si la diferencia entre `SQ_1` y `SQ_2` es igual a la de `SQ_2` y `SQ_3` (ej. 10kg y 10kg), el levantador ha alcanzado la "Armonía de la Forja". Es una línea recta perfecta.

### C. La "Bendición del Oráculo" (Relative Strength)

Los **RPE Oracles** de Sunspire desprecian la masa bruta.

* **Tu Script:** Aplica la fórmula de **Wilks** o **IPF GL** (puedes buscar la fórmula matemática y programarla en JS).
* **Lore:** No importa quién levanta más, sino quién desafía más a su propia naturaleza (Peso levantado vs Peso corporal).

---

## 3. Dinámica de Torneo: La "Sede" importa

En tu dataset, la columna `Sede` no es aleatoria. Vamos a añadirle una regla de **Home Bias** para tu lógica de programación:

> **Regla de Lore:** "El rugido de la grada local otorga un +2.5% de fuerza, pero un +5% de exceso de confianza (riesgo de fallo)".

**Reto para tu AppScript:**
Crea una función que compare el `Total` de los levantadores que compiten en su `Origen` (Sede == Origen) contra los que viajan.

* *¿Los de Frosthold pierden fuerza cuando compiten en el calor de Ironforge?*
* *¿Los de Shadowfen fallan más terceros intentos por la presión de su público?*

---

## 4. Estructura de "Clases" (Categorías)

Para que tu código de clasificación sea robusto, usa estas definiciones de Lore para las categorías de edad:

* **Sub-Junior (The Apprentices):** < 18 años. Datos muy erráticos. Mucho potencial, técnica inestable.
* **Junior (The Squires):** 19-23 años. Los que más arriesgan.
* **Open (The Champions):** 24-39 años. El pico de los datos. Menos nulos, más kilos.
* **Masters (The Elders):** 40+ años. La vieja guardia. Suelen ser de la *Linear Legion*. Datos muy consistentes, casi nunca fallan el primer intento.

---

### ¿Qué quieres hacer a continuación con el código?

Puedo ayudarte con cualquiera de estos tres "Hechizos de AppScript":

1. **Hechizo de Limpieza:** Un script que recorre las filas y pone en **Rojo** todas las celdas con números negativos (fallidos) y calcula el total solo de los positivos.
2. **Hechizo de Clasificación:** Un script que lea la columna `Edad` y `Peso` y te rellene automáticamente las columnas `Cat_Edad` y `Cat_Peso` siguiendo las reglas del torneo.
3. **Hechizo de Clasificación de Equipos:** Un script que cree una nueva hoja con el "Ranking de Gremios" (promedio de kilos por Equipo).

¿Por cuál empezamos?
