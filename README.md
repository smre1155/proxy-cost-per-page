# proxies para web scraping: cómo elegir el tipo correcto y calcular el coste real por página

Casi nadie busca "proxies para web scraping" por curiosidad. Se busca porque algo se rompió: tu scraper empezó a devolver 403, aparecieron CAPTCHAs donde antes había HTML limpio, o llegó la factura del mes y no cuadraba con lo que creías haber gastado. El problema de fondo casi nunca es "qué proveedor es el mejor", sino dos decisiones concretas: qué **tipo** de proxy necesitas y qué **modelo de cobro** encaja con tu carga de trabajo. Si esas dos están bien, el resto son detalles.

Aquí van las dos, con precios verificables y ejemplos de DataImpulse para poner números sobre la mesa.

## El tipo de proxy importa más que la marca

Un detalle que se repite en los benchmarks independientes: en sitios que bloquean bots de verdad, la diferencia entre proveedores es pequeña, y la diferencia entre tipos de proxy es grande. En el benchmark continuo de AIMultiple, con unas 17.000 URLs distintas y peticiones cada pocos minutos, los proxies residenciales y de datacenter aciertan entre el 55 % y el 75 % en objetivos protegidos, no el 95–99 % que anuncian las páginas de los proveedores.

La explicación no es que los proveedores mientan por sistema. Es que "éxito" se mide de formas distintas: un muro de login servido con HTTP 200 puede contar como éxito en una media global, y como fallo en una prueba real. Tú vas a pagar tráfico, así que te interesa la segunda métrica.

| Tipo de proxy | Precio de referencia 2026 | Para qué sirve en scraping | Cuándo evitarlo |
| --- | --- | --- | --- |
| Datacenter | ~$0,50–3/GB | Objetivos con protección ligera o nula, volumen alto, coste bajo | Cuando el sitio bloquea rangos de nube de forma explícita |
| Residencial (rotativo) | ~$1–8/GB | E-commerce, SERPs, redes sociales, precios y reseñas con geolocalización | Cuando cada página pesa mucho y el presupuesto manda |
| Móvil (4G/5G) | ~$2–15/GB | Objetivos especialmente duros, datos de apps y de web móvil | Cuando el residencial ya funciona: pagas de más |
| Residencial premium | ~$5/GB en adelante | Flujos donde la latencia y la estabilidad pesan más que el precio | Proyectos exploratorios o de bajo volumen |

El consejo más rentable que hay en este terreno: **empieza por datacenter**. Lanza 100 peticiones a tu objetivo a través de un proxy de datacenter barato. Si más del 60 % devuelve la página real, probablemente no necesites nada más caro. Si falla, sube a residencial. Mucha gente contrata residencial por reflejo y paga entre dos y diez veces más de lo necesario para rastrear páginas públicas sin protección.

## Cuánto cuesta scrapear: la cuenta que casi nadie hace

El precio por GB solo tiene sentido cuando lo conviertes en coste por página. La aritmética es simple:

1. Supón 1.000.000 de páginas al mes.
2. Una página HTML típica pesa entre 0,2 y 1 MB. Toma 0,5 MB.
3. 1.000.000 × 0,5 MB = 500 GB.
4. A $1/GB son ~$500. A $5/GB, ~$2.500.
5. Ajusta por tasa de éxito: al 95 %, divide por 0,95 → ~$526.

La misma página a $0,50/GB se queda en ~$250. La misma página a $8/GB se va a $4.000. Es decir: el precio por GB decide si tu proyecto cuesta un mes de hosting o el sueldo de un junior.

Dos matices que suelen olvidarse:

- **El renderizado pesa.** Si usas Playwright o Puppeteer sin bloquear imágenes, fuentes y scripts de terceros, no estás bajando 0,5 MB por página sino bastante más. Hay equipos que bajan de 20 MB a 5 MB por página simplemente ajustando la configuración del navegador, y eso es un 75 % menos de factura con el mismo número de páginas.
- **La tasa de éxito es parte del precio.** Un pool de $0,50/GB que te bloquea la mitad de las veces acaba costando más por dato utilizable que uno limpio de $1/GB. Compara siempre dividiendo por el porcentaje de páginas reales que obtienes.

## Pago por uso o suscripción: aquí se pierde dinero sin darse cuenta

El modelo de cobro te afecta más de lo que parece. Las suscripciones por IP tienen sentido cuando necesitas pocas direcciones estables con mucha transferencia y uso constante. Para cargas irregulares —una auditoría de precios cada tres semanas, un proyecto puntual, un pico de temporada— el pago por uso casi siempre sale mejor, sobre todo si el tráfico no caduca.

Ese "si no caduca" es la clave. Cuando los GB comprados se reinician a final de mes, el volumen que pagaste y no consumiste desaparece: si compras 100 GB para un pico y usas 40, tu coste real por GB fue 2,5 veces el anunciado. Con saldo que no expira, ese problema simplemente no existe.

DataImpulse trabaja así: pago por uso, sin cuota mensual, y el tráfico comprado no caduca. El precio de entrada en residencial es **$1/GB**, el de datacenter **$0,50/GB** y el de móvil **$2/GB**, con un mínimo de recarga de $5. AIMultiple, que prueba proveedores de forma continua, lo sitúa entre las opciones de precio de entrada más bajo del mercado precisamente por ese $5 de arranque.

👉 [Ver precios y empezar con el plan de $5](https://bit.ly/dataimPulse)

## Rotación y sesiones sticky: cuándo usar cada una

Otra decisión que cambia resultados más que el proveedor.

- **Rotación por petición**: cada request sale con una IP nueva. Es lo que quieres para listados de productos, resultados de búsqueda o cualquier página independiente. Reduce el riesgo de rate limit porque ninguna IP acumula demasiadas peticiones.
- **Sesión sticky**: mantienes la misma IP durante una secuencia. Necesario cuando hay carrito, paginación con estado, login o un flujo de varios pasos. Si la IP cambia a mitad de flujo, el sitio te expulsa y no siempre sabrás por qué.

DataImpulse soporta ambos modos en sus productos, con HTTP/HTTPS y SOCKS5. La red residencial son más de 90 millones de IP de origen ético —pool propio, no revendido— en 195 países, y la tasa de éxito que publican es del 99,51 %. Cifra de la casa, así que tómala como referencia de marketing y mídela tú mismo con tus objetivos.

## Cómo se configura en tu scraper

Un proxy para scraping se usa casi siempre igual: cambia el endpoint y sigues con tu código. Con `requests` en Python sería algo así (el endpoint concreto lo genera tu panel al crear el plan):

python
import requests

proxy = "http://USUARIO:CONTRASEÑA@GATEWAY:PUERTO"

r = requests.get(
    "https://ejemplo.com/producto/123",
    proxies={"http": proxy, "https": proxy},
    timeout=20,
)
print(r.status_code, len(r.content))


Y con curl, para comprobar qué IP estás viendo realmente:

bash
curl -x "http://USUARIO:CONTRASEÑA@GATEWAY:PUERTO" https://httpbin.org/ip


Un detalle práctico de DataImpulse: el destino geográfico se pasa como parámetro dentro del propio endpoint, no como un ajuste separado en el panel por cada ubicación. Eso simplifica el código cuando rastreas varios países a la vez, porque puedes tener una variable por mercado en lugar de una configuración por país.

Sobre el targeting: la selección por país está incluida en la tarifa base. Ciudad, estado, código postal y ASN son opcionales de pago, y AIMultiple señala que en los planes residenciales estándar el tráfico que pasa por esos filtros avanzados se factura al doble de la tarifa. Verifica ese punto con soporte antes de montar un presupuesto que dependa de segmentación fina: la diferencia entre incluir ciudad o no puede duplicar tu coste por página.

## Los errores que disparan la factura

- **Medir el precio por GB en lugar del coste por página.** Es el error que más caro sale y el más común.
- **Usar residencial donde bastaba datacenter.** Ya lo hemos visto: prueba primero el barato.
- **Renderizar con navegador sin bloquear recursos.** Imágenes, fuentes y trackers se pagan como ancho de banda.
- **Ignorar los reintentos.** Cada timeout y cada CAPTCHA son tráfico facturado que no te dio un dato.
- **Comprar paquetes que caducan.** Si tu volumen varía mes a mes, es dinero que regalas.
- **Fiarse de listas de proxies gratis.** Un análisis de 640.600 proxies medidos encontró tasas de supervivencia muy bajas y documentó prácticas como inyección de anuncios o reescritura de enlaces de afiliado en el tráfico que pasa por proxies abiertos. Para aprender a configurar un proxy en curl valen; para un crawling real, no. La regla mínima: nunca envíes logins, API keys ni cookies por un proxy que no sea tuyo.

Bloque aparte merecen los objetivos difíciles. Si un sitio usa Cloudflare o Akamai en modo agresivo, el datacenter devolverá 403 aunque envíes cabeceras y cookies correctas. Ahí la decisión es residencial, ISP o directamente un servicio de desbloqueo. Antes de escalar de proxy, comprueba que tu cliente HTTP envía las cabeceras que el sitio espera y que estás navegando a la URL de datos en lugar de haciendo clic por cinco páginas.

## Planes y precios de DataImpulse

Estos son los planes que la compañía publica para sus cuatro líneas de producto. Los precios son por compra de tráfico, no por suscripción, y el tráfico no caduca. Los importes listados corresponden a los precios públicos consultados al preparar esta guía; conviene confirmarlos en la página del producto antes de comprar, porque cambian.

### Proxies residenciales — 90M+ IPs en 195 países, $1/GB

| Plan | Tráfico | Precio | Precio por GB | Enlace |
| --- | --- | --- | --- | --- |
| Intro | 5 GB | $5 | $1,00 | [Empezar con 5 GB](https://dataimpulse.com/residential-proxies/?aff=86938) |
| Basic | 50 GB | $50 | $1,00 | [Comprar 50 GB](https://dataimpulse.com/residential-proxies/?aff=86938) |
| Advanced | 1 TB | $800 | $0,80 | [Comprar 1 TB con descuento](https://dataimpulse.com/residential-proxies/?aff=86938) |
| Custom+ | desde 5 TB | precio a medida | negociado | [Solicitar volumen a medida](https://dataimpulse.com/residential-proxies/?aff=86938) |

### Proxies de datacenter — uptime del 99,9 %, $0,50/GB

| Plan | Tráfico | Precio | Precio por GB | Enlace |
| --- | --- | --- | --- | --- |
| Intro | 10 GB | $5 | $0,50 | [Empezar con 10 GB](https://dataimpulse.com/datacenter-proxies/?aff=86938) |
| Basic | 100 GB | $50 | $0,50 | [Comprar 100 GB](https://dataimpulse.com/datacenter-proxies/?aff=86938) |
| Advanced | 1 TB | $450 | $0,45 | [Comprar 1 TB de datacenter](https://dataimpulse.com/datacenter-proxies/?aff=86938) |
| Custom+ | desde 5 TB | precio a medida | negociado | [Solicitar volumen mayorista](https://dataimpulse.com/datacenter-proxies/?aff=86938) |

### Proxies móviles — 4G/5G/LTE, $2/GB

| Plan | Tráfico | Precio | Precio por GB | Enlace |
| --- | --- | --- | --- | --- |
| Intro | 2,5 GB | $5 | $2,00 | [Empezar con 2,5 GB móviles](https://dataimpulse.com/mobile-proxies/?aff=86938) |
| Basic | 25 GB | $50 | $2,00 | [Comprar 25 GB móviles](https://dataimpulse.com/mobile-proxies/?aff=86938) |
| Advanced | 1 TB | $1.600 | $1,60 | [Comprar 1 TB móvil con descuento](https://dataimpulse.com/mobile-proxies/?aff=86938) |
| Custom+ | desde 5 TB | precio a medida | negociado | [Consultar plan personalizado](https://dataimpulse.com/mobile-proxies/?aff=86938) |

### Proxies residenciales premium — pool de alta velocidad, $5/GB

| Plan | Tráfico | Precio | Precio por GB | Enlace |
| --- | --- | --- | --- | --- |
| Intro | 1 GB | $5 | $5,00 | [Probar el pool premium](https://dataimpulse.com/premium-residential-proxies/?aff=86938) |
| Basic | 10 GB | $50 | $5,00 | [Comprar 10 GB premium](https://dataimpulse.com/premium-residential-proxies/?aff=86938) |
| Custom+ | volumen alto | precio a medida | negociado | [Hablar con el equipo](https://dataimpulse.com/premium-residential-proxies/?aff=86938) |

El plan premium incluye gestor de cuenta dedicado, uptime del 99,9 % y todas las opciones de segmentación sin recargo — que es justo lo que en los planes residenciales estándar se paga aparte. Para la mayoría de proyectos de scraping, ese extra no compensa; para flujos con SLA y objetivos duros, sí puede.

## Cómo empezar en tres pasos

1. Crea la cuenta y entra al panel.
2. Añade un plan nuevo y elige tipo de proxy — residencial si tus objetivos están protegidos, datacenter si no lo están.
3. Recarga saldo. Con $5 tienes 5 GB de residencial, 10 GB de datacenter o 2,5 GB de móvil, y ese tráfico no se te va a caducar mientras mides tus costes reales.

Las compras iniciales con tarjeta incluyen garantía de devolución de 7 días, siempre que no hayas consumido más del 80 % del tráfico. Las compras con criptomonedas en planes Intro no son reembolsables, así que si tu intención es probar y decidir, paga con tarjeta. Tampoco hay prueba gratuita sin pago: el acceso empieza siempre por esa recarga de $5.

👉 [Crear cuenta y activar el plan de entrada](https://bit.ly/dataimPulse)

## Preguntas frecuentes

**¿Residencial o datacenter para web scraping?**
Depende del objetivo, no del tipo de proyecto. Si el sitio bloquea rangos de datacenter, residencial. Si es contenido público sin protección, datacenter a $0,50/GB y te ahorras hasta el 50 % del coste. Haz una prueba de 100 peticiones antes de decidir.

**¿Cuántos GB necesito para X páginas?**
Multiplica el número de páginas por el peso medio de la página y súmale el tráfico de los reintentos. Un millón de páginas de 0,5 MB son unos 500 GB; con renderizado de navegador sin optimizar, bastante más.

**¿Sirve una lista de proxies gratuitos para un scraping en producción?**
No de forma sensata. El coste real no es cero: añades comprobaciones de salud, reintentos, detección de proxies muertos y verificas que nadie manipuló el contenido en tránsito. Dos días de ingeniería al mes cuestan más que un plan pagado.

**¿Los GB caducan?**
En DataImpulse no: el tráfico comprado permanece en tu saldo hasta que lo consumes. Es la diferencia práctica frente a proveedores que resetean el contador cada mes.

**¿Qué métrica debo vigilar después de escalar?**
El coste por página recogida con éxito, no el precio por GB. Divide tu factura mensual entre el número de registros válidos que obtuviste. Si esa cifra baja al optimizar cabeceras y recursos, vas por buen camino; si sube, el problema es tu configuración, no el proveedor.
