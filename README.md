# Control de brillo funcional en MacBook Pro 5,5 con MX Linux 23 Fluxbox

Este documento explica cómo habilitar el control de brillo de la pantalla (teclas **F1/F2**) en **MX Linux 23 Fluxbox (64-bit)** instalado en un **MacBook Pro 5,5 (2009)** con GPU Nvidia 9400M.  
Los pasos fueron probados y verificados en un sistema funcional.

---

## 1. Verificar el dispositivo de brillo

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

## 2. Instalar herramienta de control de brillo

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

## 3. Permitir que el usuario cambie el brillo sin usar `sudo`

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

ejemplo de cómo debe aparecer:

```bash
$ groups $USER
wachin : wachin lp dialout cdrom floppy sudo audio dip video plugdev users netdev lpadmin scanner sambashare
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

si desea probar como funciona ponga en la terminal para bajar el brillo:

```bash
brightnessctl set 10%-
```

y para subirlo:

```bash
brightnessctl set 10%+
```

**Nota:** Estos comandos también se pueden añadir en algún otro Linux en las opciones de atajos de teclado, para hacer funcionar esas teclas.

---

## 4. Configurar teclas F1 y F2 en Fluxbox

Editar el archivo de teclas:

```bash
nano ~/.fluxbox/keys
```

Agregar al final:

```bash
# Control de brillo con F1 y F2
XF86MonBrightnessDown :ExecCommand brightnessctl set 10%-
XF86MonBrightnessUp   :ExecCommand brightnessctl set 10%+
```

Guardar (**Ctrl+O**, **Enter**, **Ctrl+X**) y aplicar los cambios:

```bash
fluxbox-remote reconfigure
```

Ahora las teclas **F1/F2** ajustan el brillo correctamente sin requerir `sudo`.

---

## (Opcional) Mostrar notificación del brillo

Instalar el paquete de notificaciones:

```bash
sudo apt install libnotify-bin
```

Y usar estas líneas en lugar de las anteriores:

```bash
XF86MonBrightnessDown :ExecCommand bash -c 'brightnessctl set 10%- && notify-send "🔅 Brillo: $(brightnessctl g)"'
XF86MonBrightnessUp   :ExecCommand bash -c 'brightnessctl set 10%+ && notify-send "🔆 Brillo: $(brightnessctl g)"'
```

Esto mostrará una notificación visual cada vez que se cambie el brillo.

---

### ⚠️ Nota importante

En algunos sistemas, las teclas **F5/F6 no envían la señal correcta**.

Para comprobarlo:

```bash
xev
```

Presionar F5 y F6 y verificar qué evento generan.
Si no funcionan, se pueden reasignar usando el nombre correcto que muestre `xev`.

---

## Resultado final

* El brillo se puede ajustar con F1/F2 sin usar `sudo`.
* Los permisos se aplican automáticamente al iniciar el sistema.
* Funciona perfectamente en **MX Linux 23 Fluxbox** sobre **MacBook Pro 5,5 (Nvidia 9400M)**.
* Configuración persistente y limpia, sin modificar archivos del sistema gráfico.

---

## 💡 Control del brillo del teclado (F5 / F6) en MacBook Pro 5,5

Además del brillo de pantalla (F1/F2), el **MacBook Pro 5,5** cuenta con **retroiluminación de teclado**, controlada por las teclas **F5 y F6**.

En Linux, este control **no usa `/sys/class/backlight`**, sino el subsistema de LEDs.

---

### 1. Verificar el dispositivo del teclado

Abrir una terminal y ejecutar:

```bash
ls /sys/class/leds/
```

Debe aparecer:

```
smc::kbd_backlight
```

Si aparece, el sistema reconoce correctamente el teclado retroiluminado.

---

### 2. Probar el control manual

```bash
brightnessctl -d smc::kbd_backlight set 50%
```

Si aparece el error de permisos:

```
Permission denied
```

es normal, se soluciona en el siguiente paso.

---

### 3. Permitir control sin `sudo`

Crear una regla de **udev**:

```bash
sudo tee /etc/udev/rules.d/91-kbd-backlight.rules > /dev/null <<'EOF'
ACTION=="add", SUBSYSTEM=="leds", KERNEL=="smc::kbd_backlight", RUN+="/bin/chgrp video /sys/class/leds/%k/brightness", RUN+="/bin/chmod g+w /sys/class/leds/%k/brightness"
EOF
```

Recargar reglas:

```bash
sudo udevadm control --reload
sudo udevadm trigger
```

**Nota:** Pero si deseas cierra sesión y vuelve a entrar porque ejemplo si tenías hecha alguna configuración con por ejemplo el touchpad, se borrará hasta que reinicies.

Asegurarse de pertenecer al grupo `video`, bueno eso ya está en el paso anterior.

---

### 4. Probar el control del teclado

Para subir el brillo del teclado:

```bash
brightnessctl -d smc::kbd_backlight set 10%+
```

Para bajar el brillo del teclado:

```bash
brightnessctl -d smc::kbd_backlight set 10%-
```

---

### 5. Configurar teclas F5 y F6 en Fluxbox

Editar:

```bash
nano ~/.fluxbox/keys
```

Agregar:

```bash
# Brillo teclado MacBook
F5 :ExecCommand brightnessctl -d smc::kbd_backlight set 10%-
F6 :ExecCommand brightnessctl -d smc::kbd_backlight set 10%+
```

Aplicar cambios:

```bash
fluxbox-remote reconfigure
```

---

### ⚠️ Nota importante

En algunos sistemas, las teclas **F5/F6 no envían la señal correcta**.

Para comprobarlo:

```bash
xev
```

Presionar F5 y F6 y verificar qué evento generan.
Si no funcionan, se pueden reasignar usando el nombre correcto que muestre `xev`.

---

### ✅ Resultado final

* El teclado se ilumina correctamente.
* Control total sin usar `sudo`.
* Integración con teclas F5 y F6.
* Configuración persistente y limpia.

---

