# PDF Infra - API Gateway & Rate Limiter

Infraestructura base (`.infra`) para la arquitectura de microservicios del extractor de PDFs. Provee el punto de entrada unificado (*API Gateway*), balanceo de carga y limitación de tráfico distribuido por IP usando Redis.

---

## Componentes

### API Gateway & Proxy Inverso (Traefik)
  - *Access logs* y logs del sistema con sincronización de zona horaria local (`/etc/timezone`).
  - Enrutamiento y vinculación de middleware por *labels*. Faltan los labels correspondientes a la logica de negocio de del sistema.
  - Circuit Breaker del middleware definido como `cb-documents`, abre el circuito en caso de que el 50% de las peticiones tarda más de 200ms o que el 25% de las respuestas sean 5xx. 

### Persistencia de Rate Limit (Redis)
  - Control distribuido de cuota fijado a **70 peticiones/segundo por IP**.
  - Política de gestión de memoria `volatile-lfu` (100 MB max) para rotación de claves temporales.
  - En el futuro Redis tambien va a manejar el cache que sea necesario en la logica de negocio del sistema.

### queue de mensajes (Redis)
  - Segunda instancia de Redis, cola de mensajes para satisfacer ADR-0004.
  - Memoria maxima de 200 megas.
  - Persistencia ligera (AOF / Append Only File) para que, si el contenedor de infraestructura se reinicia, los trabajos pendientes en los streams `queue:extraction` o `queue:conversion` no se pierdan.
  - `noeviction` si se llena la cola no se aceptan mas trabajos.

### cache hit (Redis)
  - Tercera instancia de Redis, cache tradicional que se comunica con `PDF-persistancie`
  - Memoria de 64 megas
  - Cache ligero volatil, `appendonly: no`.

### Red Compartida (`fast_pdf_network`)
  - Red bridge interna que interconecta el proxy con los contenedores de la lógica de negocio.

---

## Levantamiento de la Infraestructura

```bash
# Iniciar el entorno base (Traefik + Redis)
docker compose -f docker-compose.infra.yml up -d
# whoamis de prueba
cd whoami
docker compose docker-compose.yml up
```

