# Anexo aclaratorio del PDF — INC-03

Este anexo acompaña a `SOP-INC-03 (1).pdf` y precisa la lectura de sus capturas. El PDF original conserva las evidencias visuales, pero algunas leyendas y la conclusión requieren corrección. Debe reemplazarse por una exportación corregida cuando esté disponible el documento fuente editable.

## Aclaraciones sobre las figuras

1. **Figura 5:** la captura de `df -hT` muestra `/dev/sda3` con 2.6G. Corresponde al estado posterior a ampliar el disco virtual en Proxmox, pero anterior a ampliar la partición y el sistema de archivos. No es el estado final.
2. **Figura 6 superior:** muestra el procedimiento de ampliación con `parted` y `resize2fs`, y la verificación final con `df -h`: `/` quedó con 8.5G totales y 8.0G disponibles.
3. **Figura 6 inferior:** muestra `fdisk -l /dev/sda` después del cambio. Su leyenda debe ser **Figura 7**, no Figura 6.
4. Las siguientes figuras deben renumerarse de forma consecutiva: la actual Figura 8 permanece como Figura 8 si se añade la nueva Figura 7; la captura de permisos, actualmente Figura 9, también permanece como Figura 9. Al regenerar el informe, confirmar que el orden y las etiquetas de todas las figuras sean consecutivos y únicos.

## Aclaración de capacidad y causa

- El disco virtual quedó en 10 GB, según Proxmox y `fdisk`.
- La partición raíz `/dev/sda3` quedó en 8915 MB; el sistema de archivos ext4 montado en `/` reportó 8.5G totales y 8.0G disponibles.
- Por tanto, se alcanzó un **disco virtual de 10 GB**, pero no 10 GB utilizables en `/`. Si el requisito es tener 10 GB utilizables en `/`, debe ampliarse el disco virtual por encima de 10 GB y volver a verificarse.
- Antes de extender la partición, `/` ya tenía 2.4G libres y el 3% de uso; los inodos también tenían disponibilidad. Se confirmó que la partición no aprovechaba todo el disco virtual, pero **no se confirmó que esa diferencia causara el error de escritura original**.
- La prueba en `/var/lib/nginx/html` se realizó como root. El directorio pertenece a `root:root` y tiene permisos 755. Esa prueba no demuestra que el usuario de desarrollo pueda guardar o desplegar archivos.

## Conclusión corregida

Se amplió la capacidad de la partición raíz y del sistema de archivos ext4, y las comprobaciones posteriores mostraron 8.0 GB disponibles en `/`. Nginx aparecía iniciado y una prueba temporal de escritura como root pasó. La causa de la capacidad de trabajo reducida quedó identificada como una partición raíz menor que el disco virtual. Los errores originales de guardado todavía no se han reproducido con el usuario de desarrollo; por ello el ticket debe permanecer abierto o pendiente de validación funcional hasta confirmar el requisito de 10 GB y probar el flujo real de guardado.

## Mejoras requeridas al exportar la siguiente versión

- Corregir la numeración de figuras para que sea única y consecutiva.
- Cambiar la leyenda de la captura previa a la ampliación de la partición para indicar que `/` aún mide 2.6G.
- Reemplazar la conclusión que declara el incidente resuelto por la conclusión corregida de este anexo.
- Añadir una prueba de guardado con el usuario real de desarrollo y, si está disponible, una comprobación HTTP del servicio después del cambio.
- Registrar la línea exacta de `parted`, fecha/hora, responsable, snapshot/respaldo y resultado del plan de reversión; no inventar los datos que no se conservaron.
