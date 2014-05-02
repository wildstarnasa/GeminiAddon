GeminiAddon
===========

Provides a template for creating addon objects for WildStar

Some features include:

* Callback when the player has been loaded and entered the world (handles both initial world loads and reload UIs)
* Settings restore is delayed until the player has been loaded and entered the world
* Module support
* Embed packages into your addon or modules if the package supports embedding

### Callback Changes

Unfortunately Apollo does not allow addons to specify or rename the callback handlers from OnSave, OnRestore and OnLoad.  GeminiAddon will reserve those callback handler names for addons using GeminiAddon and provide you with alternatives.  The new callback handler names will become:

* `OnLoad` becomes `OnInitialize`
* `OnRestore` becomes `OnRestoreSettings`
* `OnSave` becomes `OnSaveSettings`

A new callback has been provided with the name `OnEnable` and this callback will fire when the player has been loaded and entered the world.
 
The OnRestoreSettings callback will fire for each level with save data after OnEnable has been called.
 
This means the start up sequence for an addon will become: OnInitialize > OnEnable > OnRestoreSettings


### Addon Creation

The addon creation process becomes the following:
 
Place the script tag for GeminiAddon.lua at the top of your toc.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Addon APIVersion="6" Name="MyAddon" Description="Example toc.xml">
  <Script Name="GeminiAddon.lua"/>
  ...
  <Script Name="MyAddon.lua"/>
  ...
</Addon>
```

```lua
-- MyAddon.lua

-- Create the addon object and register it with Apollo in a single line
local MyAddon = Apollo.GetPackage("Gemini:Addon-1.0").tPackage:NewAddon("MyAddon", false)

-- Replaces MyAddon:OnLoad
function MyAddon:OnInitialize()
  Print("MyAddon:OnInitialize called")
end

-- Called when player has loaded and entered the world
function MyAddon:OnEnable()
  Print("MyAddon:OnEnable called - " .. GameLib.GetPlayerUnit():GetName() .. " has entered the world!")
end

-- Replaces MyAddon:OnSave
function MyAddon:OnSaveSettings(eLevel)
  Print("MyAddon:OnSaveSettings called for SaveLevel " .. eLevel)
  return { eLevel = eLevel, bIsSaved = true }
end

-- Replaces MyAddon:OnRestore
function MyAddon:OnRestoreSettings(eLevel, tSavedData)
  Print("MyAddon:OnRestoreSettings called for SaveLevel " .. eLevel)
  Print("eLevel = " .. eLevel .. ", SavedeLevel = " .. tostring(tSavedData.eLevel) .. ", bIsSaved = " .. tostring(tSavedData.bIsSaved))
end
```

GeminiAddon:NewAddon offers the same functionality that Apollo.RegisterAddon does and a little more.
`GeminiAddon:NewAddon([oAddon ,] strAddonName, bConfigure[, tDependencies, [strPkgName, ...]])`

1. `oAddon` is an optional argument where you can pass in a base object to use for the addon
2. `strAddonName` is required and the unique name for your addon
3. `bConfigure` is also required and can be either a boolean or string value.  If you pass it false, no Config button will be added to the ESC/options window.  If you pass true, it will use the strAddonName as the button text. If you pass a string, it will use that as the button text.
4. `tDependencies` is an optional table that will contain a list of packages and addons that your addon depends upon.
5. `strPkgName` are packages that you would like to embed/mixin into the addon.  The packages are required to support package embedding.


### Module Support

GeminiAddon provides basic module support out of the box.  If you are familiar to AceAddon, this should be very similar except that modules will not be extended to addon status.
 
To create a module:

```lua
-- Create a module
local MyModule = MyAddon:NewModule("MyModule")

-- Create a module with some embedded packages
local MyModule = MyAddon:NewModule("MyModule", "PkgWithEmbed-1.0", "PkgWithEmbed2-1.0")

-- Create a module with a prototype
local oPrototype = { OnEnable = function(self) print("OnEnable called!") end }
local MyModule = MyAddon:NewModule("MyModule", oPrototype)
```

To get a module you have created for the addon:

```lua
-- Get a module
local MyModule = MyAddon:GetModule("MyModule")
```

Iterate over the modules in your addon:

```lua
for strModuleName, oModule in MyAddon:IterateModules() do
  Print(oModule:GetName())
end
```

Default module state, prototypes and embeds can be specified though they will need to be done before any modules are created.
