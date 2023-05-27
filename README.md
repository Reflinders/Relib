# Relib
A robust object-orientated UI library made by Reflinders (@Mlgisbetter on Roblox). Inspired by other UI libraries, such as Orion and Rayfield.

# Intro

To load Relib, you must require the module or load the raw version through loadstring; however, note that loadstring compatibility is not available yet:
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

In order to add buttons and functional ui to a page, you must add `Relib Components`. `Relib:GetComponents()` will return a dictionary of all components available. 
Practically all components follow the same practice in adding them, but some may have different parameters or events. This section will cover those differences and the general practices.

```lua
local Components : {
  Boolean : {}, 
  Slider : {},
  Textbox : {},
  Embed : {},
  Dropdown : {},
  Button : {},
  Colorpicker : {}
} = Relib:GetComponents()
--
local Embed = Components.Embed
Page:Mount(Embed.new, {
  Title = 'Hello World!',
  Description = 'This is a description.'
})
```

## Embeds
Embeds only have two properties in their parameters: Title and Description. Embeds have no signals, so don't try to connect/hook events.

```lua
local Arguments = {
  Title = 'Hello World!',
  Description = 'This is a description.'
}
Page:Mount(Embed.new, Arguments)
```

## Buttons
Buttons are more complex than Embeds, but still fairly simple. The only property it has is Title; and it has one signal referenced as Activated, which fires when the button is pressed.

```lua
local Arguments = {
  Title = 'Hello World!',
  Hooks = {
    Activated = function()
      print("Hello World!")
    end
  }
}
Page:Mount(Button.new, Arguments)
```

## Textbox
Textboxes contain the basic properties like Title, and it's own property, ClearTextOnFocus. Textboxes have 2 signals: ValueChanged, which is a basic signal that will show up in practically every component other than Buttons and Embeds; and Focused, which is fired when the text box is focused.

```lua
local Arguments = {
  Title = 'Kill Some Player',
  ClearTextOnFocus = true,
  Hooks = {
    ValueChanged = function(text : string)
      local players = game.Players
      local plrToKill = players:FindFirstChild(text)
      if plrToKill then
        plrToKill.Character.Humanoid.Health = 0
      end
    end,
    
    Focused = function()
      print('Textbox focused!')
    end
  }
}
Page:Mount(Textbox.new, Arguments)
```

## Slider
Slider has Min, Max, and Title being its properties, with Min being the minimum value it can get and Max being the opposite. Slider only has the basic signal(s)—ValueChanged.

```lua
local Arguments = {
  Title = '# of Bananas',
  Min = -5, Max = 5; -- Note that the value will default to the min
  Hooks = {
    ValueChanged = function(value : number)
      _G.Bananas = value
    end,
  }
}
Page:Mount(Slider.new, Arguments)
```

## Boolean
The boolean component is a toggle (which is valued at true or false). It only has the basic properties and signals.

```lua
local Arguments = {
  Title = 'Inf Health',
  Hooks = {
    ValueChanged = function(value : number)
      if value then 
        game.Players.LocalPlayer.Character.Health = math.huge
      else
        game.Players.LocalPlayer.Character.Health = 100
      end
    end,
  }
}
Page:Mount(Boolean.new, Arguments)
```
