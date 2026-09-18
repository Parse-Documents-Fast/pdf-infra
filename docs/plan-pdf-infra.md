# plan.md — pdf-infra

## Qué hay que construir
Un contenedor de Traefik configurado como gateway único del sistema, más un Redis en memoria pura (sin persistencia a disco) que sirve tanto de contador para rate limiting como de infraestructura compartida para lo que necesiten los demás servicios más adelante.

## Cómo construirlo, en orden
1. Levantar Traefik con `docker-compose`, entrypoints HTTP (redirige a HTTPS) y HTTPS con los certs de mkcert montados por volumen — nunca copiados a la imagen.
2. Levantar Redis al lado, con `--save "" --appendonly no` para que quede puramente en memoria.
3. Configurar el middleware de circuit breaker (latencia, ratio de errores 5xx, ratio de errores de red) como config de Traefik.
4. Configurar el middleware de rate limit apuntando a Redis como backend distribuido.
5. Montar `/etc/localtime` (no solo `/etc/timezone`) para que los logs queden en hora local de verdad.
6. Benchmark con vegeta o k6 contra un servicio de prueba (`whoami`) para confirmar que el rate limit y el circuit breaker disparan bajo carga.

## Ya resuelto
Estructura de `certs/`, `config/traefik.yml`, `config/config.yml` y `docker-compose.infra.yml` — este repo ya está en estado activo, este plan documenta cómo se llegó ahí.
