# Control de brillo funcional en MacBook Pro 5,5 con MX Linux 23 Fluxbox

Este documento explica cómo habilitar el control de brillo de la pantalla (teclas **F1/F2**) en **MX Linux 23 Fluxbox (64-bit)** instalado en un **MacBook Pro 5,5 (2009)** con GPU Nvidia 9400M.  
Los pasos fueron probados y verificados en un sistema funcional.

---

## 🧭 1. Verificar el dispositivo de brillo

Abrir una terminal y ejecutar:

```bash
ls /sys/class/backlight/
````

En el **MacBook Pro 5,5** debería aparecer:

```
nv_backlight
```

Para probar el control manual:

```bash
echo 50 | sudo tee /sys/class/backlight/nv_backlight/brightness
```

Si el brillo cambia, el dispositivo está funcionando correctamente.

---

## ⚙️ 2. Instalar herramienta de control de brillo

```bash
sudo apt install brightnessctl
```

Probar que funciona (debería mostrar el estado del brillo):

```bash
brightnessctl -l
```

Ejemplo de salida:

```
Device 'nv_backlight' of class 'backlight':
    Current brightness: 40 (40%)
    Max brightness: 100
```

---

## 🔒 3. Permitir que el usuario cambie el brillo sin usar `sudo`

Por defecto, solo *root* puede escribir en el archivo `/sys/class/backlight/nv_backlight/brightness`.
Para permitir que el usuario lo modifique, crear una regla de **udev**:

```bash
sudo tee /etc/udev/rules.d/90-backlight.rules > /dev/null <<'EOF'
ACTION=="add", SUBSYSTEM=="backlight", KERNEL=="nv_backlight", RUN+="/bin/chgrp video /sys/class/backlight/%k/brightness", RUN+="/bin/chmod g+w /sys/class/backlight/%k/brightness"
EOF
```

Recargar las reglas y aplicarlas:

```bash
sudo udevadm control --reload
sudo udevadm trigger --subsystem-match=backlight
```

Asegurarse de que el usuario pertenece al grupo `video`:

```bash
groups $USER
```

Si no aparece `video`, agregarlo:

```bash
sudo usermod -aG video $USER
```

Cerrar sesión o reiniciar el sistema.

Luego comprobar permisos:

```bash
ls -l /sys/class/backlight/nv_backlight/brightness
```

Debe mostrar algo como:

```
-rw-rw-r-- 1 root video ... /sys/class/backlight/nv_backlight/brightness
```

---

## 🎹 4. Configurar teclas F1 y F2 en Fluxbox

Editar el archivo de teclas:

```bash
nano ~/.fluxbox/keys
```

Agregar al final:

```bash
# Control de brillo con F1 y F2
XF86MonBrightnessDown :ExecCommand brightnessctl set 10%-
XF86MonBrightnessUp   :ExecCommand brightnessctl set +10%
```

Guardar (**Ctrl+O**, **Enter**, **Ctrl+X**) y aplicar los cambios:

```bash
fluxbox-remote reconfigure
```

Ahora las teclas **F1/F2** ajustan el brillo correctamente sin requerir `sudo`.

---

## 💡 (Opcional) Mostrar notificación del brillo

Instalar el paquete de notificaciones:

```bash
sudo apt install libnotify-bin
```

Y usar estas líneas en lugar de las anteriores:

```bash
XF86MonBrightnessDown :ExecCommand bash -c 'brightnessctl set 10%- && notify-send "🔅 Brillo: $(brightnessctl g)"'
XF86MonBrightnessUp   :ExecCommand bash -c 'brightnessctl set +10% && notify-send "🔆 Brillo: $(brightnessctl g)"'
```

Esto mostrará una notificación visual cada vez que se cambie el brillo.

---

## ✅ Resultado final

* El brillo se puede ajustar con F1/F2 sin usar `sudo`.
* Los permisos se aplican automáticamente al iniciar el sistema.
* Funciona perfectamente en **MX Linux 23 Fluxbox** sobre **MacBook Pro 5,5 (Nvidia 9400M)**.
* Configuración persistente y limpia, sin modificar archivos del sistema gráfico.

---

