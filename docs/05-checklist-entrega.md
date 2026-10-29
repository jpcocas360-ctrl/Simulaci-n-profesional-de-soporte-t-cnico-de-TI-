# Lista de verificación — entrega INC-03

## Documentación y evidencias

- [x] El ticket recibido y los síntomas están documentados sin presentar la hipótesis como causa confirmada.
- [x] Los cinco integrantes y sus roles están registrados.
- [x] Se separó la capacidad del disco virtual del tamaño/espacio libre del sistema de archivos.
- [x] Se registraron resultados antes y después de ampliar `/dev/sda3` y ext4.
- [x] Se documentó que los inodos no estaban agotados.
- [x] Se documentó el estado activo de Nginx y los permisos de la raíz web.
- [x] Se indicó que la prueba de escritura se hizo con privilegios root y no valida el usuario de desarrollo.
- [x] Se agregó un anexo para aclarar errores de numeración y afirmaciones demasiado concluyentes en el PDF original.
- [ ] Corregir el documento fuente y volver a exportar el PDF: la captura tiene Figura 6 duplicada, falta Figura 7 y la conclusión declara resuelto el incidente sin validar el flujo del desarrollador.

## Criterios para cerrar el incidente

- [ ] El solicitante confirma si los 10 GB se refieren al tamaño total del disco virtual o al espacio disponible/utilizable en `/`.
- [ ] Si se requieren 10 GB utilizables en `/`, se amplía el disco virtual por encima de 10 GB con respaldo y autorización, y se verifica el nuevo tamaño.
- [ ] El usuario real de desarrollo/deployment reproduce y luego prueba con éxito la operación de guardado en la ruta de trabajo.
- [ ] Si la escritura falla, se registra el mensaje exacto y se aplica el permiso mínimo necesario; no usar permisos generales `777`.
- [ ] Se realiza una comprobación HTTP del sitio después del cambio y se guarda su resultado.
- [ ] El solicitante confirma que puede continuar con su trabajo.
- [ ] La bitácora indica fecha, responsable, aprobación, resultado final y cualquier limitación residual.
- [ ] El ticket se cierra solo cuando los criterios anteriores que correspondan están satisfechos.

## Entrega

- [x] El repositorio enlaza el PDF original, las pruebas, la bitácora y el informe técnico.
- [ ] Se publica el PDF corregido y se verifica que todos los enlaces abran.
- [ ] Se entrega el enlace final del repositorio en Moodle.

Revisado por:

Fecha:
