# Relib
A robust and flexible object-orientated UI library made by Reflinders (@Mlgisbetter on Roblox). Inspired by other UI libraries, such as Orion and Rayfield. 
It should be noted that understanding how Relib works requires experience/understanding in Luau.

Development of Relib began 5/16.
Relib will be publicly released on 6/21, and the documentation will be updated by then.

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
`Relib.new` returns the main relib item necessary for creating the ui and all of its components. Argument one is the name and argument two is the parent gui. Argument 3 is a table of settings. The new `relib` is made from the template (`Template.new`) given in arg 4, which defaults to the standard template. Once a new template is made, it will expand on it, adding several signals and properties: signals `Created` and `Destroyed`, tables `Pages` and `Settings`, and properties `Type` and `Name`. Once the object is expanded, the `:RelibInit()` method (inherited from the template item) is fired. Additionally, it should be noted that the `Destroy` method should not be used on a `Relib` object; instead, call the object (shown in figure 1.2):

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

Templates must have the 2 methods/functions `RelibInit` and `Destroy`, and the module must have a constructor. Refer to Figure 1.3.1 for the basics in creating a template.
```lua
-- Figure 1.3.1
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

### Standard Template
As mentioned previously, if the template is not supplied when `Relib.new` is called, it will proceed with the standard (default) template. Refer to figure 1.3.2 to see the few methods/properties available with the standard template.

```lua
-- Figure 1.3.2
local newRelib = Relib.new("Reflinders's Script", script.Parent, {
  Title = "Reflinder's Script" -- for the intro
  Description = "The best script eva"
})

newRelib:Notify(imageId, title, description)
-- Planning to add:
newRelib:ChangeTheme(newRelib.Theme.Violet)
```

## Page Type
A page is what is created and returned through `relibObject:Create()`. The first argument is the title of the page, or in other words, what text will appear on the button. The `page` item has the properties `Type`, `Title`, `BasePg`, and `Button`, and the method `Destroy`. `BasePg` is the instance of the page, so essentially the ui object. Similarly, `Button` is the instance of the button. Additionally, it should be noted that when a page is created, event/signal "`Created`" of the `relib` object will be fired with the page as the first argument. 

```lua
-- Figure 1.4
local Page : {
  Title : string,
  Type : string,
  BasePg : ScrollingFrame,
  Button : Frame,
  Destroy : signal,
  Created : signal
} = relibObject:Create('Regular')
```
 
# 2.0 | Components

In order to add buttons and functional ui to a page, you must first manage to use the components of relib. `Relib:GetComponents()` will return a dictionary of all components available. 
Practically all components follow the same practice in adding them, but some may have different parameters, properties, methods, or signals. This section will cover those differences and the general practices. Also, it should be noted that every component that has a ValueChanged signal has the property `.Value`. Additionally, all components that have values can have the default value set through adding the property `Value` in the `EmbedData` (table of argument given when creating a component). It is advised to use the method `Set` on the components when you are changing the value rather than directly changing it through its property `Value`.

```lua
-- Figure 2.1
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
--
-- example for value usage --
local Boolean = Components.Boolean
Page:Mount(Boolean.new, {
  Title = 'Kill Aura',
  Value = false
  Hooks = {
    ValueChanged = function(bool)
      if bool then
        coroutine.wrap(function()
          _G.killAuraActive = true
          while _G.killAuraActive do
            task.wait(1)
            local plrs = game.Players:GetPlayers()
            for _, x in ipairs(plrs) do
              local c = x.Character; if c then
                local h = c:FindFirstChild('HumanoidRootPart')
                if h then
                  if (h.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude < 10 then
                    task.wait(); c.Humanoid:TakeDamage(25)
                  end
                end
              end
            end
          end
          coroutine.yield()
        end)()
      else
        _G.killAuraActive = false
      end
    end
  }
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
Dropdowns are the most advanced of all the components. The properties of the arguments are `Title` and `Set`, and the signals are 'Descended', 'Ascended', and 'ValueChanged'. The `dropdown` also has the property 'IsAscended' and the method `:add(setOption : any)` and `:Shift(newOptions:{any})`, which allow you to externally add new options. Also, it should be noted that dropdowns do not have the method `:Set` like the other components.

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
local newDropdown = Page:Mount(Dropdown.new, Arguments)
newDropdown.ValueChanged:Connect(function()
  newDropdown:Shift{25, 45, 65, 85, 105, 125, 145}
end)
```

## Colorpicker
Colorpickers, similarly to dropdowns, have the basic signals as well as the signals `Ascended` and `Descended`. The returned `colorpicker` does not have any additional methods or properties other than `Value` and `Title`.

```lua
local Arguments = {
  Title = 'Body Color',
  Hooks = {
    ValueChanged = function(color : Color3)
       local partsInCharacter = game.Players.LocalPlayer.Character:GetDescendants()
       for _, part in ipairs(partsInCharacter) do
        part.Color = color
       end
    end
  }
}
Page:Mount(Colorpicker.new, Arguments)
```
## Alternative to `Mount`
Since some may find the `Mount` method to be too extensive, there is an alternative function that you can use to use less space/lines: `WrapMount`. Refer to Figure 2.1 to figure out how to set the method up and how to use it.

```lua
-- Figure 2.1
function Relib:WrapMount(...)
  assert(self.Type == 'Page', 'Relib: Attempt to WrapMount outside of a `Page` item.'
  local Arguments = {...}
  local Components = Relib:GetComponents(); if not Components[arguments[1]] then
    return
  end
  --/ ...
  local Base = {
      Title = Arguments[2],
      Set = Arguments[3],
      Min = Arguments[3][1],
      Max = Arguments[3][2],
      ClearTextOnFocus = Arguments[3],
      Description = Arguments[3] 
   }; return function(hooks)
    Base.Hooks = hooks
    return Page:Mount(Components[arguments[1]].new, Base)
   end
end
--
local newDropdown = Page:WrapMount('Dropdown', 'WalkSpeed', {10, 50, 100})()-- Is less expansive through the use of simple arguments rather than a table of arguments
local newButton = Page:WrapMount('Button', 'Print "Hello, world!"'){ -- The function returned is used to hook functions; in a nutshell, the first argument is the table of hooks. Regardless, it must be executed to actually create the relib component.
  Activated = function()
    warn("Hello, world!")
  end
}
```
# Alternative to Standard Relib Page and Component Creation
Another way to create a new `Relib` is by wrapping the new relib in a table of creations. I won't go into too much detail about this, but Refer to Figure 3.1 to see how.

```lua
-- Figure 3.1
local Expand, Avant, Element = Relib.Expand, Relib.Avant, Relib.Element -- Avant is for pages; element is for components.
local newRelib = Relib.new("Reflinders's Script", script.Parent){
  [Expand] = {
    Avant 'New Page' {
      Element 'Button' {
        Title = 'Create Part',
        Hooks = {
         Activated = function()
            Instance.new('Part', workspace)
         end
        }
      }
    }
  }
}
newRelib.Memory[1].UIComponents[1].Activated:Connect(function() -- new table `Memory` is created when 
  newRelib() -- destroys the relib obj
end)
```

```Version: Alpha 0.1; Note that there may be bugs and that the standard Relib Ui is subject to change```

``Roadmap``:
V.0.2 --
- Light-Mode and custom themes for standard template
- New colorpicker  (circular colorpicker, revamped features)
- Changes in Standard Ui (revamped ui, sleeker features, better tweens)
- Changes in Embeds (adaptive length changing) [FINISHED]
