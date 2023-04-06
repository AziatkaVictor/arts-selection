# Arts Selection

Давным-давно, от нечего делать, я решил сделать небольшой скрипт, для селекции артефактов, основанный на GUI. Это было не легко и в итоге я не могу утверждать, что сделал все до конца, но теперь оно работает. Так как я использовать это не буду, а как-то выкидывать на помойку не хочется, то просто выложу это сюда.

![Демонстрация](url/text.png)

Реализован такой функционал:

* Проверка нахождения  в определённой зоне селекции, относительно каждого артефакта
* Вывод информации об артефакте
* Вывод информации о необходимых компонентах, их наличии или отсутствии
* Информация о зонах селекции


## Как добавить

Процесс добавения достаточно прост. Для начала наобходимо создать объект, используя который, мы сможем открывать диалог селекции. Делайте на своё усмотрени, но вот пример секции, который я создал в `configs\misc\items.ltx`:

```ini
[art_selection]:booster
$spawn 				        = "devices\art_selection"
visual				        = dynamics\devices\dev_aptechka\dev_aptechka_low.ogf
inv_name			        = st_art_selection
inv_name_short			    = st_art_selection
description			        = st_art_selection_descr
inv_weight			        = 0.5

inv_grid_width			    = 1
inv_grid_height			    = 1
inv_grid_x			        = 6
inv_grid_y			        = 14
cost				        = 2500

boost_time			        = 0
boost_health_restore		= 0
boost_radiation_restore		= 0
boost_bleeding_restore		= 0

use_sound			        = interface\inv_medkit
```

Далее нужно добавить вызов при использовании. Для этого мы открываем файл `scripts\bind_stalker.script`, ищем в нём функцию `actor_binder:use_inventory_item(obj)`. После того как нашли, мы увидим что-то такое:

```lua
function actor_binder:use_inventory_item(obj)
    if(obj) then
        local s_obj = alife():object(obj:id())
        if(s_obj) and (s_obj:section_name()=="drug_anabiotic") then
            xr_effects.disable_ui_only(db.actor, nil)
            level.add_cam_effector("camera_effects\\surge_02.anm", 10, false, "bind_stalker.anabiotic_callback")
            level.add_pp_effector("surge_fade.ppe", 11, false)
            give_info("anabiotic_in_process")
            _G.mus_vol = get_console():get_float("snd_volume_music")
            _G.amb_vol = get_console():get_float("snd_volume_eff")
            get_console():execute("snd_volume_music 0")
            get_console():execute("snd_volume_eff 0")
        end
    end
end
```

Тут уже зарегестрирован приём анабиотика. Для корректной работы нужно добавить `ui_art_selection.RunDialog()` и `alife():create()`, так как мы сделали наш предмет как `booster` и после использования он пропадает. По аналогии необходимо добавить эти строки: 

```lua
if(s_obj) and (s_obj:section_name() == "art_selection") then
    ui_art_selection.RunDialog()
    alife():create("art_selection", db.actor:position(), db.actor:level_vertex_id(), db.actor:game_vertex_id(), db.actor:id())
end
```

В итоге получаем это:

```lua
function actor_binder:use_inventory_item(obj)
    if(obj) then
        local s_obj = alife():object(obj:id())
        if(s_obj) and (s_obj:section_name()=="drug_anabiotic") then
            xr_effects.disable_ui_only(db.actor, nil)
            level.add_cam_effector("camera_effects\\surge_02.anm", 10, false, "bind_stalker.anabiotic_callback")
            level.add_pp_effector("surge_fade.ppe", 11, false)
            give_info("anabiotic_in_process")
            _G.mus_vol = get_console():get_float("snd_volume_music")
            _G.amb_vol = get_console():get_float("snd_volume_eff")
            get_console():execute("snd_volume_music 0")
            get_console():execute("snd_volume_eff 0")
        end
        if(s_obj) and (s_obj:section_name() == "art_selection") then
            ui_art_selection.RunDialog()
            alife():create("art_selection", db.actor:position(), db.actor:level_vertex_id(), db.actor:game_vertex_id(), db.actor:id())
        end
    end
end
```

## Добавление рецептов и структура

Все настраивается в `scripts\ui_art_selection.script`. Там хранится вся нужная информация.

### Регистарция рецепта

Самое важное, добавление своего рецепта. Сделано всё просто. Добавляем новый блок, и можем указать такие переменные как:

* `result` - результат селекции
* `items` - необходимые ингредиенты
* `image` - иконка *(опционально)*
* `places` - места селкции из таблицы `places_table` *(опционально)*
* `chance` - шанс селекции *(опционально)*
* `needRecipe` - необходимость в наличии инфо *(опционально)*
* `recipe` - название инфо, если `needRecipe = true`. Если не указать, то название инфо должно быть `result` + `_info`. Например: `af_medusa_info` *(опционально)*

Например, таблица рецептов может выглядеть так:

```lua
local selection_table = {
    {
        result = "af_medusa",
        items = {"wpn_pm_actor"},
        image = "ui_art_selection_items_1",
        chance = 80,
    },
    {
        result = "af_mincer_meat",
        items = {"af_fireball", "af_electra_moonlight", "af_dummy_dummy", "af_eye"},
        places = places_table.toxic,
        needRecipe = true
    },
}
```

### Регистарция зоны селекции

Перечень зон, со своими StoryID. Нужен для указания типа зоны в рецепте селекции, ради проверки на нахождение ГГ в неё.

```lua
local places_table = {
    toxic = {"StoryID_Toxic_Zone1", "StoryID_Toxic_Zone2"},
    gravi = {"StoryID_Gravi_Zone"},
    heat = {"StoryID_Heat_Zone"},
}
```