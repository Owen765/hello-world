require "defines"
require "story"

script.on_init(function()
  global.story = story_init()
end)

script.on_configuration_changed(function(data)
  if global.story == nil then
    global.story = story_init()
  end
end)

-- The story contents
story_table =
{{
  {
    action =
    function()
      game.show_message_dialog{text = {"msg-introduction"}}
      game.show_message_dialog{text = {"msg-ask-chest"}}
    end
  },
  {
    action =
    function()
      local player = game.get_player(1)
      if player ~= nil then
        player.gui.top.add{type = "button", name="button_use_chest", caption={"button-use-chest"}}
        player.gui.top.add{type = "button", name="button_no_chest", caption={"button-no-chest"}}
      end
    end
  },
  {
    condition =
    function(event)
      if event.name == defines.events.on_gui_click then
        local player = game.get_player(event.player_index)
        if event.element.name == "button_use_chest" then
          build_starting_chest(player)
        elseif event.element.name ~= "button_no_chest" then
          return false
        end
        player.gui.top.button_use_chest.destroy()
        player.gui.top.button_no_chest.destroy()
        return true
      end
      return false
    end,
    action = function()
      game.show_message_dialog{text = {"msg-ask-technologies"}}
    end
  },
  {
    action =
    function()
      local player = game.get_player(1)
      if player ~= nil then
        player.gui.top.add{type = "button", name = "button_technologies_researched", caption = {"button-technologies-researched"}}
        player.gui.top.add{type = "button", name = "button_technologies_normal", caption = {"button-technologies-normal"}}
      end
    end
  },
  {
    condition =
    function(event)
      if event.name == defines.events.on_gui_click then
        local player = game.get_player(event.player_index)
        if event.element.name == "button_technologies_researched" then
          player.force.research_all_technologies()
        elseif event.element.name ~= "button_technologies_normal" then
          return false
        end
        player.gui.top.button_technologies_researched.destroy()
        player.gui.top.button_technologies_normal.destroy()
        return true
      end
      return false
    end,
    action = function()
      action = function() game.show_message_dialog{text = {"msg-start-play"}} end
    end
  },
  -- start the game
  {
    -- After this point we don't need events any more, so we just disable them all
    action =
    function()
      script.on_event(defines.events, nil)
    end
  },
  {
    condition = function() return false end,
    action = function() end
  }
}}

story_init_helpers(story_table)

-- We need to update the story after every event except on_player_created, so the order
-- of the tw following definitions is important.

script.on_event(defines.events, function(event)
  story_update(global.story, event, "")
end)

script.on_event(defines.events.on_player_created, function(event)
  local player = game.get_player(event.player_index)
  local character = player.character
  player.character = nil
  if character then character.destroy() end
end)

function build_starting_chest(player)
  local chest_position = player.surface.find_non_colliding_position("wooden-chest", player.position, 0, 0.1)
  local chest = player.surface.create_entity{name="wooden-chest", position=chest_position, force = game.forces.player}
  chest.insert({name="coal", count="50"})
  chest.insert({name="stone", count="50"})
  chest.insert({name="iron-plate", count="50"})
  chest.insert({name="copper-plate", count="50"})
  chest.insert({name="iron-gear-wheel", count="50"})
  chest.insert({name="electronic-circuit", count="50"})
  chest.insert({name="offshore-pump", count="50"})
  chest.insert({name="pipe", count="50"})
  chest.insert({name="boiler", count="50"})
  chest.insert({name="basic-mining-drill", count="50"})
  chest.insert({name="steam-engine", count="50"})
  chest.insert({name="stone-furnace", count="50"})
  chest.insert({name="basic-transport-belt", count="50"})
  chest.insert({name="basic-inserter", count="50"})
  chest.insert({name="small-electric-pole", count="50"})
  chest.insert({name="assembling-machine-1", count="50"})
end

script.on_event(defines.events.on_rocket_launched, function(event)
  local force = event.rocket.force
  if event.rocket.get_item_count("satellite") > 0 then
    if global.satellite_sent == nil then
      global.satellite_sent = {}
    end
    if global.satellite_sent[force.name] == nil then
      game.set_game_state{game_finished=true, player_won=true, can_continue=true}
      global.satellite_sent[force.name] = 1
    else
      global.satellite_sent[force.name] = global.satellite_sent[force.name] + 1
    end
    for index, player in pairs(force.players) do
      if player.gui.left.rocket_score == nil then
        local frame = player.gui.left.add{name = "rocket_score", type = "frame", direction = "horizontal", caption={"score"}}
        frame.add{name="rocket_count_label", type = "label", caption={"", {"rockets-sent"}, ""}}
        frame.add{name="rocket_count", type = "label", caption="1"}
      else
        player.gui.left.rocket_score.rocket_count.caption = tostring(global.satellite_sent[force.name])
      end
    end
  else
    if (#game.players <= 1) then
      game.show_message_dialog{text = {"gui-rocket-silo.rocket-launched-without-satellite"}}
    else
      for index, player in pairs(force.players) do
        player.print({"gui-rocket-silo.rocket-launched-without-satellite"})
      end
    end
  end
end)
