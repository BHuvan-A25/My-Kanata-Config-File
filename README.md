# ⌨️ My Kanata Config

Kanata is a tool that remaps your keyboard at the software level — no special hardware needed.

👉 [Kanata GitHub](https://github.com/jtroo/kanata) | [Docs](https://github.com/jtroo/kanata/blob/main/docs/config.adoc)

---

## What This Config Does

- **CapsLock** → tap = `Super/Win` key, hold 2 seconds = actual CapsLock
- **Home row keys** act as modifiers when held (no need to reach for Ctrl/Shift/Alt)
- **Hold Space** → number/function key layer
- **Hold `'`** → arrow key layer

---

## Home Row Mods

You type normally, but if you **hold** these keys they become modifiers:

| Key | Tap | Hold |
|-----|-----|------|
| `A` | a | Ctrl |
| `S` | s | Shift |
| `D` | d | Alt |
| `K` | k | Alt |
| `L` | l | Shift |
| `;` | ; | Ctrl |

> Hold time is 200ms — tap normally and it works as a letter, hold a little longer and it's a modifier.

---

## Layers

### Space (hold) — Numbers & Function Keys

| Keys | Output |
|------|--------|
| Q W E R T Y U I O P | F1 F2 F3 F4 F5 F6 F7 F8 F9 F10 |
| `[` `]` | F11 F12 |
| A S D F G H J K L ; | 1 2 3 4 5 6 7 8 9 0 |

### `'` (hold) — Arrow Keys & Navigation

| Key | Output |
|-----|--------|
| `W` | ↑ |
| `A` | ← |
| `S` | ↓ |
| `D` | → |
| `Q` | Home |
| `E` | End |
| `R` | Page Up |
| `F` | Page Down |

---

## Install

### Step 1 — Download Kanata

```bash
wget https://github.com/jtroo/kanata/releases/latest/download/kanata
chmod +x kanata
sudo mv kanata /usr/local/bin/kanata
```

### Step 2 — Fix Permissions (so you don't need sudo)

```bash
sudo usermod -aG input $USER
sudo usermod -aG uinput $USER

echo 'KERNEL=="uinput", MODE="0660", GROUP="uinput", OPTIONS+="static_node=uinput"' \
  | sudo tee /etc/udev/rules.d/99-uinput.rules

sudo udevadm control --reload-rules && sudo udevadm trigger
```

Then **log out and back in**.

### Step 3 — Copy the Config

```bash
mkdir -p ~/.config/kanata
cp kanata.kbd ~/.config/kanata/kanata.kbd
```

### Step 4 — Find Your Keyboard Device

```bash
ls /dev/input/by-path/
```

Copy the path that ends in `event-kbd` and update this line in `kanata.kbd`:

```lisp
(defcfg
  linux-dev /dev/input/by-path/YOUR-KEYBOARD-PATH-HERE
)
```

Not sure which one? Run `sudo evtest` to identify your keyboard.

### Step 5 — Run It

```bash
kanata --cfg ~/.config/kanata/kanata.kbd
```

---

## Autostart on Boot

```bash
mkdir -p ~/.config/systemd/user

cat > ~/.config/systemd/user/kanata.service << SVCEOF
[Unit]
Description=Kanata
After=graphical-session.target

[Service]
ExecStart=/usr/local/bin/kanata --cfg %h/.config/kanata/kanata.kbd
Restart=on-failure

[Install]
WantedBy=default.target
SVCEOF

systemctl --user enable --now kanata
```

---

## Troubleshooting

**Permission denied?**
```bash
sudo usermod -aG input $USER
# log out and back in
```

**Mods triggering too easily or not at all?**  
Edit these values in the config (lower = faster, higher = more forgiving):
```lisp
(defvar
  tap-time 200
  hold-time 200
)
```

**Check logs:**
```bash
journalctl --user -u kanata -f
```
