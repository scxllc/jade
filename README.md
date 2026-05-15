# Documentation


## Quick Start

```lua
loadstring(game:HttpGet('https://raw.githubusercontent.com/0x1cx/jade/refs/heads/main/jade'))()
UILib:ShowDemoMenu()
```

---

## Menu

The root `UILib` object (global). Controls the overall menu window.

### `UILib:SetMenuTitle(newTitle: string)`
Sets the title displayed in the menu's title bar.
```lua
UILib:SetMenuTitle("My Script")
```

### `UILib:SetMenuSize(newSize: Vector2)`
Sets the width and height of the menu window.
```lua
UILib:SetMenuSize(Vector2.new(500, 400))
```

### `UILib:SetMenuPosition(newPos: Vector2)`
Sets the top-left position of the menu window on screen.
```lua
UILib:SetMenuPosition(Vector2.new(100, 100))
```

### `UILib:GetMenuSize() -> Vector2`
Returns the current menu size as a `Vector2`.
```lua
local size = UILib:GetMenuSize()
print(size.x, size.y)
```

### `UILib:CenterMenu()`
Centers the menu on the screen.
```lua
UILib:CenterMenu()
```

### `UILib:SetWatermarkEnabled(enabled: boolean)`
Shows or hides the watermark.
```lua
UILib:SetWatermarkEnabled(false)
```

### `UILib:Tab(tabName: string) -> Tab`
Creates a new tab in the side panel. Returns a `Tab` object.
```lua
local mainTab = UILib:Tab("Main")
```

### `UILib:Notification(text: string, duration: number)`
Shows a temporary notification popup.
```lua
UILib:Notification("Settings saved!", 3)
```

### `UILib:RegisterActivity(callback: function) -> Activity`
Registers a function that runs every frame (inside the render loop). Returns an `Activity` object.
```lua
local activity = UILib:RegisterActivity(function()
    -- runs every frame
    print("tick")
end)
```

### `UILib:SetBlockRobloxInput(enabled: boolean)`
Sets whether the menu blocks game input (e.g., mouse clicks, keyboard presses) from reaching Roblox.
```lua
UILib:SetBlockRobloxInput(true) -- Block Roblox input
```

### `UILib:CreateSettingsTab(customName?: string) -> settingsTab, menuSection, themingSection`
Creates a pre-built settings tab with menu key override and theme switching. Returns the tab and its two sections.
```lua
local settings, menuSec, themeSec = UILib:CreateSettingsTab("Settings")
```

### `UILib:ShowDemoMenu()`
Creates a demo menu showcasing all available UI elements. Useful for testing. Has its own built-in render loop.
```lua
UILib:ShowDemoMenu()
```

### `UILib:Unload()`
Removes all drawn elements and restores Roblox input. Cleans up the library entirely.
```lua
UILib:Unload()
```

### `UILib:Step()`
The main render/update function. Must be called every frame. Handles all input processing, drawing, and callbacks.
```lua
while true do
    UILib:Step()
end
```

> **Note:** `ShowDemoMenu()` has its own built-in render loop, so you don't need to call `Step()` manually when using it.

---

## Activity

Returned by `UILib:RegisterActivity()`.

### `Activity:Remove()`
Stops the activity from running each frame.
```lua
local activity = UILib:RegisterActivity(function() end)
activity:Remove() -- stops it
```

---

## Tab

Returned by `UILib:Tab()`.

### `Tab:Section(sectionName: string) -> Section`
Creates a new section within the tab. Sections are laid out in two columns — the first section goes left, the second goes right, third left, etc.
```lua
local mainTab = UILib:Tab("Main")
local section1 = mainTab:Section("Combat")
local section2 = mainTab:Section("Movement")
```

### `Tab:TabbedSection(tabNames: table) -> table<string, Section>`
Creates a section with clickable mini tabs in the header. Only the active mini tab's items are shown. Returns a table of `Section` objects keyed by tab name.

| Parameter | Type | Description |
|-----------|------|-------------|
| `tabNames` | `table` | Array of mini tab name strings |

```lua
local mainTab = UILib:Tab("Main")
local tabs = mainTab:TabbedSection({"Farming", "Settings", "Events"})

-- each key returns a standard Section
tabs["Farming"]:Toggle("Auto Farm", false, function(v) end)
tabs["Farming"]:Dropdown("Mode", {"Nearest"}, {"Nearest", "Furthest"}, false)
tabs["Settings"]:Slider("Speed", 100, 10, 0, 500, "%")
tabs["Events"]:Toggle("Sea Events", false)
```

### `Tab:SetSectionOrder(order: table)`
Explicitly sets the rendering order of sections within the tab. Pass an array of section names (for tabbed sections, use the joined name `"Tab1|Tab2|Tab3"`).
```lua
local tab = UILib:Tab("Main")
local s1 = tab:Section("Alpha")
local s2 = tab:Section("Beta")
tab:SetSectionOrder({"Beta", "Alpha"}) -- swap order
```

---

### `UILib:SetTabOrder(order: table)`
Explicitly sets the rendering order of all side tabs. Pass an array of tab names.
```lua
UILib:Tab("Combat")
UILib:Tab("Visuals")
UILib:Tab("Settings")
UILib:SetTabOrder({"Settings", "Combat", "Visuals"}) -- custom order
```

---

## Section

Returned by `Tab:Section()`. Use these methods to add UI elements.

### `Section:Toggle(label, value, callback, unsafe?, tooltip?) -> ToggleItem`
Adds an on/off toggle.

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Display text |
| `value` | `boolean` | Initial state |
| `callback` | `function(value: boolean)` | Called when toggled |
| `unsafe` | `boolean?` | If true, shows in yellow as a warning |
| `tooltip` | `string?` | Hover hint text (shown with `(?)` icon) |

```lua
local aimbot = section:Toggle("Aimbot", false, function(value)
    print("Aimbot:", value)
end, false, "Locks onto nearest target")
```

---

### `Section:Slider(label, value, step, min, max, suffix, callback) -> SliderItem`
Adds a draggable number slider.

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Display text |
| `value` | `number` | Initial value |
| `step` | `number` | Increment step (e.g. `1`, `0.5`) |
| `min` | `number` | Minimum value |
| `max` | `number` | Maximum value |
| `suffix` | `string` | Unit suffix displayed after value (e.g. `"ms"`, `"%"`) |
| `callback` | `function(value: number)` | Called when value changes |

```lua
local fov = section:Slider("FOV", 90, 1, 30, 180, "°", function(value)
    print("FOV:", value)
end)
```

---

### `Section:Dropdown(label, value, choices, multi, callback) -> DropdownItem`
Adds a dropdown selector. Dropdowns with more than 15 choices automatically show a draggable scrollbar on the right side. Click the track to jump, or drag the thumb to scroll. Dropdowns with 15 or fewer choices behave normally with no scroll UI.

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Display text |
| `value` | `table` | Initial selected values (array of strings) |
| `choices` | `table` | Available options (array of strings) |
| `multi` | `boolean` | Allow multiple selections |
| `callback` | `function(value: table)` | Called when selection changes |

```lua
-- single select
local mode = section:Dropdown("Mode", {"Silent"}, {"Legit", "Silent", "Rage"}, false, function(value)
    print("Selected:", value[1])
end)

-- multi select
local parts = section:Dropdown("Target Parts", {"Head"}, {"Head", "Torso", "Legs"}, true, function(value)
    print("Selected:", table.concat(value, ", "))
end)
```

---

### `Section:Button(label, callback) -> ButtonItem`
Adds a clickable button.

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Button text |
| `callback` | `function()` | Called when clicked |

```lua
section:Button("Destroy All", function()
    print("Destroyed!")
end)
```

---

### `Section:Textbox(label, value, callback) -> TextboxItem`
Adds a text input field.

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Display text |
| `value` | `string` | Initial text |
| `callback` | `function(value: string)` | Called when text changes |

```lua
local nameBox = section:Textbox("Target Name", "", function(value)
    print("Name:", value)
end)
```

---

### `Section:Label(text, color?) -> LabelItem`
Adds a text label to the section.

| Parameter | Type | Description |
|-----------|------|-------------|
| `text` | `string` | Label text |
| `color` | `Color3?` | Optional text color |

```lua
section:Label("Version 1.0.0", Color3.fromRGB(0, 255, 128))
```

---

## Item Methods

Methods available on items returned by Section element constructors.

### `Item:Set(value: any)`
Programmatically set the item's value. Works on: **Toggle**, **Slider**, **Dropdown**, **Textbox**.
```lua
local toggle = section:Toggle("ESP", false, function(v) end)
toggle:Set(true) -- turns it on
```

---

### `Item:AddKeybind(value?, mode?, canChange?, callback?) -> KeybindItem`
Adds a keybind to a **Toggle** item. The bound key will activate/deactivate the toggle.

> **Note:** A toggle can have either a keybind OR a colorpicker, not both.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `value` | `string?` | `nil` | Initial key name (e.g. `"f"`, `"mb4"`) |
| `mode` | `string?` | `"Toggle"` | Activation mode: `"Toggle"`, `"Hold"`, or `"Always"` |
| `canChange` | `boolean?` | `true` | Whether the user can right-click to change mode |
| `callback` | `function?` | `nil` | `function(keyId, mode)` — called when keybind changes |

**Modes:**
| Mode | Behavior |
|------|----------|
| `Toggle` | Press to flip on/off |
| `Hold` | On while held, off when released |
| `Always` | Permanently on (cannot be toggled off by key) |

```lua
local esp = section:Toggle("ESP", false, function(v) end)
local keybind = esp:AddKeybind("v", "Toggle", true)
```

**Available key names:**
`m1`, `m2`, `mb`, `mb4`, `mb5`, `tab`, `enter`, `shift`, `ctrl`, `alt`, `space`, `a`-`z`, `0`-`9`, `f1`-`f12`, `numpad0`-`numpad9`, `insert`, `delete`, `home`, `end`, `pageup`, `pagedown`, `lshift`, `rshift`, `lctrl`, `rctrl`, `lalt`, `ralt`, `capslock`, `numlock`, `scrolllock`, and more.

---

### `Item:SetTextColor(color: Color3)`
Overrides the default theme text color for this specific item.
```lua
local dangerToggle = section:Toggle("Dangerous Feature", false, function(v) end)
dangerToggle:SetTextColor(Color3.fromRGB(255, 100, 100)) -- Make text red
```

---

### `Item:AddColorpicker(label, value?, overwrite?, callback?) -> ColorpickerItem`
Adds an HSV color picker to a **Toggle** item.

> **Note:** A toggle can have either a colorpicker OR a keybind, not both.

| Parameter | Type | Description |
|-----------|------|-------------|
| `label` | `string` | Colorpicker label |
| `value` | `Color3?` | Initial color (defaults to accent color) |
| `overwrite` | `boolean?` | If true, replaces the toggle checkbox with the color preview |
| `callback` | `function(color: Color3)` | Called when color changes |

```lua
local esp = section:Toggle("ESP", false, function(v) end)
esp:AddColorpicker("ESP Color", Color3.fromRGB(255, 0, 0), false, function(color)
    print("Color:", color)
end)
```

---

### `Item:UpdateChoices(newChoices: table)`
Updates the available options for a **Dropdown** item.
```lua
local dropdown = section:Dropdown("Players", {}, {}, false, function(v) end)

-- later, update the choices dynamically
dropdown:UpdateChoices({"Player1", "Player2", "Player3"})
```

---

### `KeybindItem:Set(value: string, mode?: string)`
Programmatically set the keybind's key and optionally its mode.
```lua
keybind:Set("g", "Hold")
```

### `ColorpickerItem:Set(value: Color3)`
Programmatically set the colorpicker's color.
```lua
colorpicker:Set(Color3.fromRGB(0, 255, 128))
```

---

### `LabelItem:SetText(newText: string)`
Programmatically update the label's text.
```lua
local label = section:Label("Status: Idle")
label:SetText("Status: Running...")
```

---

## Full Example

```lua
loadstring(game:HttpGet('https://raw.githubusercontent.com/0x1cx/jade/refs/heads/main/jade'))()

UILib:SetMenuTitle("My Script")
UILib:SetMenuSize(Vector2.new(500, 400))
UILib:CenterMenu()

-- create tabs
local combat = UILib:Tab("Combat")
local visuals = UILib:Tab("Visuals")
UILib:CreateSettingsTab("Settings")

-- combat section
local aimSection = combat:Section("Aim Assist")

local aimbot = aimSection:Toggle("Aimbot", false, function(enabled)
    print("Aimbot:", enabled)
end, false, "Automatically aims at enemies")
aimbot:AddKeybind("mb5", "Toggle", true)

aimSection:Slider("FOV", 120, 1, 30, 360, "°", function(value)
    print("FOV:", value)
end)

aimSection:Dropdown("Target", {"Head"}, {"Head", "Torso", "Arms", "Legs"}, false, function(value)
    print("Target:", value[1])
end)

-- visuals section
local espSection = visuals:Section("ESP")

local esp = espSection:Toggle("Player ESP", false, function(enabled)
    print("ESP:", enabled)
end)
esp:AddColorpicker("ESP Color", Color3.fromRGB(0, 200, 120), false, function(color)
    print("ESP Color:", color)
end)

local chams = espSection:Toggle("Chams", false, function(enabled)
    print("Chams:", enabled)
end)
chams:AddKeybind("c", "Toggle", true)

espSection:Slider("Max Distance", 500, 10, 100, 2000, " studs", function(value)
    print("Max dist:", value)
end)

-- start
UILib:Notification("Script loaded!", 3)
while true do
    UILib:Step()
end
```


