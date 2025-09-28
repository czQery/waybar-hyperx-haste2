# waybar-hyperx-haste2

1. move hyperx-haste2 inside ~/.config/waybar/scripts

2. create udev rule: /etc/udev/rules.d/70-hyperx.rules

```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="03f0", ATTRS{idProduct}=="0f98", MODE="0666"
```

3. insert waybar entry:

```jsonc
"custom/hyperx-haste2": {
	"interval": 3600,
	"format": " {text}",
	"tooltip-format": "HyperX Pulsefire Haste 2",
	"exec": "~/.config/waybar/scripts/hyperx-haste2"
},
```
