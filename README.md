# Relib
A robust and flexible object-orientated UI library made by Reflinders (@Mlgisbetter on Roblox). Inspired by other UI libraries, such as Orion and Rayfield. 
It should be noted that understanding how Relib works requires experience/understanding in Luau.

# Intro
To load Relib, you must require the module or load the raw version through loadstring; however, note that loadstring compatibility is not available yet
Here's an example of Relib in use:

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

# 1.0 | Relib Objects
This section goes over the main `Relib` service and object, created through `Relib.new()`. This will also go over how to make your own relib template. 

## Relib Type
`Relib.new` returns the main relib item necessary for creating the ui and all of its components. Argument one is the name and argument two is the parent gui. The new `relib` is made from the template (`Template.new`) given in arg 3, which defaults to the standard template. Once a new template is made, it will expand on it, adding several signals and properties: signals `Created` and `Destroyed`, tables `Pages` and `Settings`, and properties `Type` and `Name`. Once the object is expanded, the `:RelibInit()` method (inherited from the template item) is fired. Additionally, it should be noted that the `Destroy` method should not be used on a `Relib` object; instead, call the object (shown in figure 1.2):

```lua 
-- Figure 1.1
local newRelib : {
  Created : signal,
  Destroyed : signal,
  Pages : {},
  Settings : {}, -- Only meant to be configured by the template
  Name : string,
  Type : string
} = Relib.new("Reflinders's Script", script.Parent)
```
```lua
-- Figure 1.2
--/ ... Destruction Methods
newRelib:Destroy() -- WRONG WAY
newRelib() -- CORRECT WAY
```
### Templates
A template is what the function `Relib.new` inherits from. Not supplying a template in `Relib.new` will result in the standard template being used. Templates are essentially the core of the `relib` object. It's suggested to use the standard template, but it is fully possible to create a custom template.

Templates must have the 2 methods/functions `RelibInit` and `Destroy`, and the module must have a constructor. Refer to Figure 1.3 for the basics in creating a template.
```lua
-- Figure 1.3
local template = {}
function template.new(parentUi : any, name : string) -- Parameters should always be the same: parentUi and Name.
  local newTemplate = {}; do
    function newTemplate:RelibInit() -- fired once the templateObject is expanded upon
      self.Ui = Instance.new('Frame', parentUi)
      self.Ui.Name = name
      --/ ...
      self.Events[#self.Events + 1] = self.Created:Connect(function(page) -- `Created` is fired when a new page is created through `:Create`
        page.BasePg.Parent = self.Ui
      end
    end

    function newTemplate:Destroy() -- fires when the relib object is called. just a simple clean-up function
      self.Ui:Destroy()
      for _, event in ipairs(self.Events) do
        event:Disconnect()
      end
    end
  end
  return newTemplate
end
return template
```

## Page Type
A page is what is created and returned through `relibObject:Create()`. The first argument is the title of the page, or in other words, what text will appear on the button. The `page` item has the properties `Type`, `Title`, `BasePg`, and `Button`, and the method `Destroy`. `BasePg` is the instance of the page, so essentially the ui object. Similarly, `Button` is the instance of the button. Additionally, it should be noted that when a page is created, event/signal "`Created`" of the `relib` object will be fired with the page as the first argument. 

# 2.0 | Components

In order to add buttons and functional ui to a page, you must first manage to use the components of relib. `Relib:GetComponents()` will return a dictionary of all components available. 
Practically all components follow the same practice in adding them, but some may have different parameters, properties, methods, or signals. This section will cover those differences and the general practices. Also, it should be noted that every component that has a ValueChanged signal has the property `.Value`. 

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
Buttons are more complex than Embeds, but still fairly simple. The only argument property it has is Title; and it has one signal referenced as Activated, which fires when the button is pressed.

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
Textboxes contain the basic argument properties like Title, and it's own property, ClearTextOnFocus. Textboxes have 2 signals: ValueChanged, which is a basic signal that will show up in practically every component other than Buttons and Embeds; and Focused, which is fired when the text box is focused.

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
The boolean component is a toggle (which is valued at true or false). It only has the basic argument properties and signals.

```lua
local Arguments = {
  Title = 'Inf Health',
  Hooks = {
    ValueChanged = function(value : boolean)
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

## Dropdown
Dropdowns are the most advanced of all the components. The properties of the arguments are `Title` and `Set`, and the signals are 'Descended', 'Ascended', and 'ValueChanged'. The `dropdown` also has the property 'IsAscended' and the method `:add(setOption : any)`, which allows you to externally add new options.

```lua
local Arguments = {
  Title = 'Jump Boost',
  Set = {25, 45, 65, 85}, -- Values that aren't strings will be converted to strings
  Hooks = {
    ValueChanged = function(value : any) -- in this situation, the value will be a number
       game.Players.LocalPlayer.Character.Humanoid.JumpPower = value
    end,
    Ascended = function(self)  -- First parameter for both Ascended and Descended is the `Dropdown`.
      self:add(math.random(1, 200))
    end,
  }
}
Page:Mount(Dropdown.new, Arguments)
```

## Colorpicker
Colorpickers, similarly to dropdowns, have the basic signals as well as the signals `Ascended` and `Descended`. The returned `colorpicker` does not have any additional methods or properties other `Value` and `Title`.

```lua
local Arguments = {
  Title = 'Body Color',
  Hooks = {
    ValueChanged = function(color : Color3) -- in this situation, the value will be a number
       local partsInCharacter = game.Players.LocalPlayer.Character:GetDescendants()
       for _, part in ipairs(partInCharacter) do
        part.Color = color
       end
    end
  }
}
Page:Mount(Colorpicker.new, Arguments)
```

```Version: Alpha 0.1; Note that there may be bugs and that the standard Relib Ui is subject to change```
