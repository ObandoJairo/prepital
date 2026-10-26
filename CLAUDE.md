# Comunicaciones Prepital

Herramienta web que genera el copy de **correos y carteleras internas** de Prepital (UT Prepacol - 3T Capital), firmadas normalmente por Gestión de Talento Humano. La persona llena un formulario y la IA redacta la pieza con la voz de marca y el manual de estilo del área de copy.

Responsable: Jairo Obando. Todo el contenido y la interfaz están en **español (Colombia)**.

## Arquitectura

```
index.html  ──POST /api/generar──►  api/generar.js  ──►  API de OpenAI
(navegador)                         (función de Vercel)
```

- **Despliegue:** Vercel, conectado a este repositorio. Cada push a `main` se publica solo en uno o dos minutos. No hay paso de build.
- **Framework preset en Vercel:** Other. Root Directory: raíz del repositorio.
- **Variables de entorno en Vercel:**
  - `OPENAI_API_KEY` (obligatoria). Nunca va en el código ni en el repositorio.
  - `MODEL` (opcional). Por defecto `gpt-5.4-mini`.
- GitHub Pages **no sirve** para esta versión, porque no ejecuta la función de `api/`.

## Archivos

| Archivo | Qué contiene |
|---|---|
| `index.html` | Página completa: HTML, CSS y JS en un solo archivo. Formulario, renderizado de la pieza, botones de copiar, historial local (localStorage) y pedido de ajustes. Envía `{accion, solicitud, actual?, ajuste?}` a `/api/generar`. |
| `api/generar.js` | Función serverless (ESM, `export default handler`). Contiene `GUIA` (voz de marca, piezas de referencia, reglas y manual de estilo) y `FORMATO` (esquema JSON de respuesta). Arma el mensaje según `accion` (`generar`, `responder`, `ajustar`) y llama a Chat Completions con `response_format: json_object`. Devuelve `{text}`. |
| `package.json` | Solo `"type": "module"`. |

Si aparece un `generar.js` suelto en la raíz, sobra y se puede eliminar.

## Contrato entre página y función

La IA devuelve un JSON con esta forma, que `index.html` sabe pintar:

```json
{
  "preguntas": [],
  "correo": { "asunto": "", "titulo": "", "etiqueta": "", "cuerpo": "", "cta": "", "cierre": "" },
  "cartelera": { "titulo": "", "subtitulo": "", "copy": "" },
  "notas": []
}
```

Convenciones del campo `cuerpo`, que `index.html` interpreta:

- Párrafos separados por línea en blanco.
- Viñetas con `- `.
- Negritas con `**así**`.
- Bloque destacado con líneas que empiezan por `> `.
- Una línea que dice exactamente `[CTA]` marca dónde va el botón; la página la muestra como botón con el texto de `cta`.

`correo` o `cartelera` vienen en `null` cuando no se pidieron. Si `preguntas` trae elementos y ambas piezas vienen en `null`, la página muestra las preguntas para que la persona las responda antes de redactar.

**Si cambias este esquema, actualiza `FORMATO` en `api/generar.js` y la función `render()` en `index.html` en el mismo commit.**

## Reglas del copy (no cambiar sin pedido explícito)

Las reglas completas están en `GUIA`, dentro de `api/generar.js`. Las esenciales:

- **Voz:** cercana, cálida, positiva e invitacional ("te invitamos", "contamos con su compromiso"). Saludo fijo: "¡Hola, equipo!". Ideas clave en negrita.
- **Estructura del correo:** título corto y aspiracional → "¡Hola, equipo!" → párrafo de contexto y motivo → CTA en la mitad → desarrollo → recuadro de 2 o 3 puntos clave → compromiso → cierre de agradecimiento.
- **Cartelera:** título exclamativo o tipo eslogan, una línea de contexto, 3 o 4 puntos cortos, entre 30 y 60 palabras. Si hay CTA, invita a revisar el correo.
- **CTA:** lo propone la IA a partir de la intención que escribe la persona (el formulario no pide el texto del botón). Va en infinitivo y con **máximo 3 palabras**, contando artículos y preposiciones.
- **Asunto:** **uno solo**, el mejor. Entre 25 y 45 caracteres, sin señales de spam, conectado con el título y el primer párrafo. La IA evalúa por dentro varios enfoques (curiosidad, beneficio, urgencia, exclusividad, recordación, conversación cercana) y entrega solo el ganador.
- **No inventar** fechas, datos, beneficios ni condiciones que no estén en la solicitud. Si falta algo imprescindible, la IA pregunta.
- **Trato:** tú o ustedes, según el formulario, en toda la pieza.
- **Manual de estilo:** números, puntuación, mayúsculas, siglas y extranjerismos según el manual del área de copy, resumido en `GUIA`. Las piezas de referencia mandan en voz y estructura; el manual manda en ortografía y redacción.

## Interfaz (decisiones ya tomadas)

- Título de la página: **Comunicaciones Prepital**.
- Layout: formulario arriba (dos columnas en pantallas grandes) y "Pieza generada" abajo, a todo el ancho. En PC, el encabezado va en dos columnas: título y saludo a la izquierda, explicación en un recuadro a la derecha.
- La página abre con el formulario vacío y el panel de resultado en estado vacío. No mostrar piezas de ejemplo.
- El CTA se muestra **solo dentro del cuerpo**, como botón. No hay una fila separada de CTA.
- Los colores se definen como variables CSS, con modo claro y oscuro. Fuentes: Montserrat (títulos) y Source Sans 3 (texto).

## Cómo trabajar en este repositorio

- Edita los archivos y haz commit y push directo a `main`. Vercel publica solo.
- Mensajes de commit en español, cortos y descriptivos.
- Antes de hacer push, verifica la sintaxis del JS (por ejemplo, extrae el `<script>` de `index.html` y córrelo con `node --check`; para `api/generar.js`, `node --check`).
- No agregues dependencias ni pasos de build, salvo que se pida.
- Nunca escribas claves, tokens ni secretos en ningún archivo.
- Los textos que ve el usuario deben cumplir el mismo manual de estilo: signos de apertura ¿ ¡, mayúscula solo inicial en títulos y sin punto final en títulos aislados.
