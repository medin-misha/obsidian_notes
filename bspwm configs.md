## bspwmrc
```
#! /bin/sh
pgrep -x sxhkd > /dev/null || sxhkd &
  
bspc monitor DP-2 -d 1 2 3
bspc monitor HDMI-0
  
bspc config border_width 3
bspc config window_gap 4
  
bspc config bottom_padding 2
bspc config top_padding 2
bspc config left_padding 2
bspc config right_padding 2
  
bspc config focused_border_color "#ffffff"
bspc config normal_border_color "#5e5d5e"
  
bspc config split_ratio 0.52
bspc config borderless_monocle true
bspc config gapless_monocle true
  
setxkbmap -option grp:alt_shift_toggle us,ru &
~/.screenlayout/display.sh
feh --bg-scale ~/Изображения/anime.jpg --display :0
polybar &
picom &
sxhkd
```

## polybar/config
```python
[colors]
background = #454042
background-alt = #86808c3f
foreground = #ffb0d1
foreground-alt = #70ff88
primary = #f4ffad
secondary = #f4ffad
alert = #00000000
underline = #a1a1a1

[bar/example]
width = 100%
height = 30
bottom = true
;offset-x = 2.5%
;offset-y = 1%
radius = 5
fixed-center = true

;dpi = 96

background = ${colors.background}
foreground = ${colors.foreground}

line-size = 0pt

border-size = 3pt
border-color = #00000000

padding-left = 0
padding-right = 1

module-margin = 0.z5

separator = |
separator-foreground = ${colors.disabled}

font-0 = monospace;2

modules-left = xworkspaces xwindow
modules-right = filesystem pulseaudio xkeyboard memory cpu wlan eth date

cursor-click = pointer
cursor-scroll = ns-resize

enable-ipc = true

; wm-restack = generic
wm-restack = bspwm
; wm-restack = i3

; override-redirect = true

; This module is not active by default (to enable it, add it to one of the
; modules-* list above).
; Please note that only a single tray can exist at any time. If you launch
; multiple bars with this module, only a single one will show it, the others
; will produce a warning. Which bar gets the module is timing dependent and can
; be quite random.
; For more information, see the documentation page for this module:
; https://polybar.readthedocs.io/en/stable/user/modules/tray.html
[module/systray]
type = internal/tray

format-margin = 10pt
tray-spacing = 16pt

[module/xworkspaces]
type = internal/xworkspaces

label-active = %name%
label-active-background = ${colors.background-alt}
label-active-underline= ${colors.primary}
label-active-padding = 1

label-occupied = %name%
label-occupied-padding = 1

label-urgent = %name%
label-urgent-background = ${colors.alert}
label-urgent-padding = 1

label-empty = %name%
label-empty-foreground = ${colors.disabled}
label-empty-padding = 1

[module/xwindow]
type = internal/xwindow
label = %title:0:60:...%

[module/filesystem]
type = internal/fs
interval = 25

mount-0 = /

label-mounted = %{F#F0C674}%mountpoint%%{F-} %percentage_used%%

label-unmounted = %mountpoint% not mounted
label-unmounted-foreground = ${colors.disabled}

[module/pulseaudio]
type = internal/pulseaudio

format-volume-prefix = "vol: "
format-volume-prefix-foreground = ${colors.primary}
format-volume = <label-volume>

label-volume = %percentage%%

label-muted = mut
label-muted-foreground = ${colors.disabled}

[module/xkeyboard]
type = internal/xkeyboard
blacklist-0 = num lock

label-layout = %layout%
label-layout-foreground = ${colors.primary}

label-indicator-padding = 2
label-indicator-margin = 1
label-indicator-foreground = ${colors.background}
label-indicator-background = ${colors.secondary}

[module/memory]
type = internal/memory
interval = 5
format-prefix = "ram: "
format-prefix-foreground = ${colors.primary}
label = %percentage_used:2%%

[module/cpu]
type = internal/cpu
interval = 5
format-prefix = "cpu:"
format-prefix-foreground = ${colors.primary}
label = %percentage:2%%

[network-base]
type = internal/network
interval = 5
format-connected = <label-connected>
format-disconnected = <label-disconnected>
label-disconnected = %{F#F0C674}%ifname%%{F#707880} disconnected

[module/wlan]
inherit = network-base
interface-type = wireless
label-connected = %{F#F0C674}%ifname%%{F-} %essid% %local_ip%

;[module/eth]
;inherit = network-base
;interface-type = wired
;label-connected = %{F#F0C674}%ifname%%{F-} %local_ip%

[module/date]
type = internal/date
interval = 1

date = %H:%M
date-alt = %Y-%m-%d %H:%M:%S

label = %date%
label-foreground = ${colors.primary}

[settings]
screenchange-reload = true
pseudo-transparency = true

; vim:ft=dosini
```
## sxhkdrc
```
#
# wm independent hotkeys
#

# terminal emulator
super + Escape
	kitty

# program launcher
super + @space
	rofi -show drun

# make sxhkd reload its configuration files:
super + Escape
	pkill -USR1 -x sxhkd

#скриншоты
super + shift + s
    scrot ~/Изображения/scrins/image.png

# Увеличить громкость
super + XF86AudioRaiseVolume
    amixer set Master 5%+

# Уменьшить громкость
super + XF86AudioLowerVolume
    amixer set Master 5%-

# Включить/выключить звук
super + XF86AudioMute
    amixer set Master toggle

# bspwm hotkeys
#

# quit/restart bspwm
super + alt + {q,r}
	bspc {quit,wm -r}

# close and kill
super + {_,shift + }w
	bspc node -{c,k}

# alternate between the tiled and monocle layout
super + m
	bspc desktop -l next

# send the newest marked node to the newest preselected node
super + y
	bspc node newest.marked.local -n newest.!automatic.local

# swap the current node and the biggest window
super + g
	bspc node -s biggest.window

#
# state/flags
#

# set the window state
super + {t,shift + t,s,f}
	bspc node -t {tiled,pseudo_tiled,floating,fullscreen}

# set the node flags
super + ctrl + {m,x,y,z}
	bspc node -g {marked,locked,sticky,private}

#
# focus/swap
#

# focus the node in the given direction
super + {_,shift + }{h,j,k,l}
	bspc node -{f,s} {west,south,north,east}

# focus the node for the given path jump
super + {p,b,comma,period}
	bspc node -f @{parent,brother,first,second}

# focus the next/previous window in the current desktop
super + {_,shift + }c
	bspc node -f {next,prev}.local.!hidden.window

# focus the next/previous desktop in the current monitor
super + bracket{left,right}
	bspc desktop -f {prev,next}.local

# focus the last node/desktop
super + {grave,Tab}
	bspc {node,desktop} -f last

# focus the older or newer node in the focus history
super + {o,i}
	bspc wm -h off; \
	bspc node {older,newer} -f; \
	bspc wm -h on

# focus or send to the given desktop
super + {_,shift + }{1-9,0}
	bspc {desktop -f,node -d} '^{1-9,10}'

#
# preselect
#

# preselect the direction
super + ctrl + {h,j,k,l}
	bspc node -p {west,south,north,east}

# preselect the ratio
super + ctrl + {1-9}
	bspc node -o 0.{1-9}

# cancel the preselection for the focused node
super + ctrl + space
	bspc node -p cancel

# cancel the preselection for the focused desktop
super + ctrl + shift + space
	bspc query -N -d | xargs -I id -n 1 bspc node id -p cancel

# Перемещение фокуса по направлению
super + {h,j,k,l}
    bspc node -f {west,south,north,east}

# Expand/contract a window by moving one of its side outward/inward
alt  {Left,Down,Up,Right}
    STEP=1; SELECTION={1,2,3,4}; \
    bspc node -z $(echo "left -$STEP 0,bottom 0 $STEP,top 0 -$STEP,right $STEP 0" | cut -d',' -f$SELECTION) || \
    bspc node -z $(echo "right -$STEP 0,top 0 $STEP,bottom 0 -$STEP,left $STEP 0" | cut -d',' -f$SELECTION)
```
## picom.conf
```
# Включение композитинга
backend = "glx";
corner-radius: 10;
```
## rofi/config.rasi
```
configuration {
  background-color: black;
  modes: [ combi ];
  combi-modes: [ window, drun, run ];
}
```
