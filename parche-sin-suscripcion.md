# Parche automático para eliminar el aviso de suscripción de Proxmox

## Qué hace

Elimina el popup "No valid subscription" que aparece al entrar a la interfaz web
de Proxmox cuando no se tiene suscripción de pago. El parche se aplica
automáticamente después de cada actualización del sistema.

## Cómo funciona

Es un hook de apt que se ejecuta después de cada `apt install` o `apt upgrade`.
Comprueba si el fichero `proxmoxlib.js` fue modificado por la actualización y,
si es así, aplica el parche y reinicia el servicio web de Proxmox.

## Crear el parche desde cero

```bash
cat > /etc/apt/apt.conf.d/99-proxmox-disable-nag << 'HOOK'
DPkg::Post-Invoke { "dpkg -V proxmox-widget-toolkit | grep -q /proxmoxlib\.js && sed -Ezi.bak 's/(Ext\.Msg\.show\(\{\\s+title: gettext\(..No valid sub)/void\(\{ \/\/\1/g' /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service; true"; };
HOOK
```

## Aplicar el parche manualmente (sin esperar a una actualización)

```bash
sed -Ezi.bak 's/(Ext\.Msg\.show\(\{\\s+title: gettext\(..No valid sub)/void\(\{ \/\/\1/g' \
  /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
systemctl restart pveproxy.service
```

## Verificar que el parche está activo

```bash
grep -n "No valid subscription" \
  /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
```

La línea debe mostrar `void({` antes del bloque, no `Ext.Msg.show({`.

## Verificar que el hook existe

```bash
cat /etc/apt/apt.conf.d/99-proxmox-disable-nag
```

## Notas

- El fichero de backup del parche queda en `proxmoxlib.js.bak`
- El hook termina en `; true` para no interrumpir actualizaciones si falla
- Compatible con Proxmox VE 9.x sobre Debian 13 (trixie)
