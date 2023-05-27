# Relib
A robust object-oriented UI library

# Intro

To use, you must require the module or load the raw version through loadstring; however, note that loadstring compatibility is not available yet:
```lua
local Relib =  require(RelibModule) -- loadstring(rawLink)()
local Components = Relib:GetComponents() -- Components needed to create buttons, sliders, dropdowns, etc.
--/ ...
local NewWindow = Relib.new() -- uses the standard template-- first arg can be a different template to hijack
local NewPage = NewWindow:Create('Admin Commands') -- Add a page
--
local Button = NewPage:Mount(Components.Button, {  -- Create a button
  Title = 'Reset',
  Hooks = { -- Hooks are used on all components; they simply hook a function to a certain event
    Activated = function()
      game.Players.LocalPlayer.Character:WaitForChild('Humanoid').Health = 0
    end
  }
})
local buttonEvent = Button.Activated:Connect(function() -- You could also externally connect an event; returns a custom connection. This can be done with any event on any component.
  warn("Button Pressed.")
end)
buttonEvent:Disconnect() -- behaves like normal rbxconnections
--
local Dropdown = NewPage:Mount(Components.Dropdown, { -- Heres another example of mounting a component; this time, it's with a dropdown
  Title = 'Keybind',
  Set = {
    'E',
    'F',
    'G',
    'R'
  },
  Hooks = {
    ValueChanged = function(value : string)
      warn(value)
    end)
  }
})
warn(Dropdown.Value) -- All components except buttons and embeds have a value
```

# Components

## Embeds
