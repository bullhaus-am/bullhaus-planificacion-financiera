# Bullhaus — Dashboard de planificación financiera

Documento de contexto para retomar este proyecto desde Claude Code.

## Qué es

Dashboard interactivo de planificación financiera de jubilación, preparado como
presentación para un cliente de Bullhaus Asset Management. Es un único archivo HTML
autónomo (`Bullhaus_Planificacion_Financiera.html`) que se abre con doble clic en
cualquier navegador y funciona **100% offline**: Chart.js y las cuatro fuentes web
están incrustadas en base64 dentro del propio archivo. No hace ninguna petición de red.

## Datos del cliente (supuestos base)

| Variable | Valor |
|----------|-------|
| Edad actual | 75 años |
| Trabaja hasta | 78 años (3 años más de ingreso laboral) |
| Horizonte de proyección | hasta los 95 años |
| Ingreso laboral actual | USD 80.000 / año |
| Gasto actual | USD 110.000 / año |
| Portfolio inicial | USD 2.000.000 |
| Moneda | USD |

Situación de partida: el cliente gasta más de lo que ingresa por trabajo (déficit de
30k/año hoy), y tras jubilarse a los 78 el portfolio pasa a cubrir la totalidad del gasto.
La tasa de retiro post-jubilación arranca en ~5,5%, que es agresiva — de ahí la
importancia de mostrar cuánto necesita ajustar el gasto.

## Variables interactivas (sliders)

1. **Rentabilidad del portfolio** — 0% a 10%, paso 0,5%. Presets: Pesimista 2% / Base 5% / Optimista 8%.
2. **Inflación de gastos** — 0% a 8%, paso 0,5%. Default 3%. Indexa el gasto año a año.
3. **Ajuste de gastos** — -40% a +20%, paso 5%. Negativo = recorte, positivo = mayor gasto.
4. **Caída del ingreso laboral** — 3 modos: corte seco a los 78 / lineal a 0 / gradual (potencia 1,6).

## Outputs en pantalla

- **4 tarjetas de resultado**: resultado a los 95 (sostenible / se agota a los N),
  portfolio final, gasto año 1 ajustado, y **retiro máximo sostenible** (calculado por
  búsqueda binaria: el gasto anual máximo que NO agota el portfolio antes de los 95).
- **Gráfico de barras apiladas**: ingreso laboral + retiro del portfolio, con línea de gasto total superpuesta.
- **Gráfico de línea**: evolución del valor del portfolio.
- **Tabla año por año**: edad, laboral, gasto, retiro, portfolio final. Filas en rojo cuando el portfolio se agota; marca "· trabaja" los años con ingreso laboral.

## Lógica del modelo (en el `<script>`)

```
function simulate(retPct, infPct, cutPct, decMode):
  para cada edad de 75 a 95:
    inc  = ingreso laboral según modo de caída (0 desde los 78)
    exp  = 110.000 * (1 + cut) * (1 + inflacion)^años
    port = port * (1 + rentabilidad)        # primero crece
    need = exp - inc
    si need > 0:  retira need del portfolio (tope: saldo disponible)
    si need < 0:  el excedente se reinvierte en el portfolio
```

`maxSustainable()` hace búsqueda binaria sobre el gasto anual (40 iteraciones,
rango 0–400k) buscando el mayor gasto con portfolio final >= 0.

Constantes al inicio del IIFE: `START_AGE=75, WORK_UNTIL=78, END_AGE=95, P0=2000000, INC0=80000, EXP0=110000`.

## Diseño / marca

- Paleta teal-verde oscura Bullhaus (fondo `#0d1f1c`, acento teal `#1d9e75`, dorado `#c9a86a`).
- Tipografías: **Cormorant Garamond** (títulos) + **Jost** (cuerpo), incrustadas en base64 (subset latin, pesos 500/600 y 400/500).
- Disclaimer legal de documento de trabajo confidencial y material ilustrativo (no asesoramiento de inversión).

## Pendientes / TODO

- [ ] **Logo real de Bullhaus**: hoy hay un placeholder (letra "B" en recuadro teal,
      clase `.brand-mark` en el HTML). Reemplazar por el PNG transparente del logo
      (texto en blanco para fondo oscuro), idealmente incrustado en base64 para mantener
      el archivo offline. El logo está en la carpeta local `nicolasalvarez/scoring`.
      Para incrustarlo: convertir a base64 y poner `<img src="data:image/png;base64,...">`
      dentro de `.brand-mark` (o reemplazar el div).
- [ ] Confirmar si los gastos deben crecer con inflación (actual) o mantenerse nominales constantes.
- [ ] (Opcional) Agregar escenario de secuencia de rendimientos / retornos variables
      para modelar el riesgo de jubilarse en un mal año de mercado.

## Cómo retomar en Claude Code

1. Abrir la carpeta del proyecto donde esté el `.html`.
2. Para editar la lógica: buscar el segundo `<script>` (el primero es Chart.js incrustado, no tocar).
3. Para cambiar supuestos del cliente: editar las constantes al inicio del IIFE.
4. Para el logo: ver el TODO de arriba.
