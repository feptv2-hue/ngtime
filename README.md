extends Control

@onready var h_box_container: HBoxContainer = $TopBar/HBoxContainer
@onready var nickname_label: Label = $TopBar/HBoxContainer/NicknameLabel
@onready var elo_label: Label = $TopBar/HBoxContainer/EloLabel
@onready var unit_slots_container: HBoxContainer = $BottomPanel/MarginContainer/UnitSlotsContainer

@onready var ru_tech_tree: Control = $MenuBlurBackground/TechTreePanel/RuTechTree
@onready var us_tech_tree: Control = $MenuBlurBackground/TechTreePanel/UsTechTree
@onready var slot_focus_scrim: ColorRect = $SlotFocusScrim
# Добавляем ссылку на панель технологий
@onready var tech_tree_panel: Control = $MenuBlurBackground
var current_nation: String = "RU"
var player_name: String = "Pro_Gamer_777"
var player_elo: int = 1450

func _ready() -> void:
	nickname_label.text = "Игрок: " + player_name
	elo_label.text = "ELO: " + str(player_elo)
	slot_focus_scrim.visible = false
	unit_slots_container.modulate = Color("FFFFFF")
	
	# На всякий случай гарантируем в коде, что при старте окно технологий закрыто
	tech_tree_panel.visible = false 

# Хранит имя юнита, который игрок ХОЧЕТ поставить в слот (например, "Т-90М" или "Abrams")
var selected_unit_to_equip: String = ""

# Режим выбора слота: true — когда мы ждем клика по нижнему слоту, false — обычный режим
var is_selecting_slot: bool = false



# Функция вызывается при нажатии на крестик «Х» внутри панели технологий
func _on_close_tech_button_pressed() -> void:
	print("Закрываем древо технологий")
	tech_tree_panel.visible = false # Снова скрываем панель

func _on_button_pressed() -> void:
	print("Запуск матча! Сохраняем деку...")
	
	var slot1_text = $BottomPanel/MarginContainer/UnitSlotsContainer/Slot1.text
	var slot2_text = $BottomPanel/MarginContainer/UnitSlotsContainer/Slot2.text
	var slot3_text = $BottomPanel/MarginContainer/UnitSlotsContainer/Slot3.text
	var slot4_text = $BottomPanel/MarginContainer/UnitSlotsContainer/Slot4.text
	var slot5_text = $BottomPanel/MarginContainer/UnitSlotsContainer/Slot5.text
	var slot6_text = $BottomPanel/MarginContainer/UnitSlotsContainer/Slot6.text
	
	# Стучимся к синглтону напрямую через корневое дерево движка:
	var global = get_node("/root/GlobalData")
	
	if global:
		global.current_deck = [slot1_text, slot2_text, slot3_text, slot4_text, slot5_text, slot6_text]
		print("Дека успешно сохранена в GlobalData: ", global.current_deck)
	else:
		print("Ошибка: Движок всё еще не создал узел GlobalData!")
		
	get_tree().change_scene_to_file("res://scenes/game_world.tscn")


#_on_techtree_pressed()
func _on_techtree_pressed() -> void:
	cancel_slot_selection()
	print("Открываем древо технологий")
	# Вместо переменной используем прямой и точный путь к панели.
	# Зажмите CTRL и перетащите узел TechTreePanel прямо сюда вместо строки ниже:
	$MenuBlurBackground.visible = true


func _on_ground_button_pressed() -> void:
	print("Вкладка: Наземная техника")
	
	if current_nation == "RU":
		# Если выбрана Россия, управляем ветками внутри RuTechTree
		$MenuBlurBackground/TechTreePanel/RuTechTree/GroundBranch.visible = true
		$MenuBlurBackground/TechTreePanel/RuTechTree/HeliBranch.visible = false
	elif current_nation == "US":
		# Если выбраны США, управляем ветками внутри UsTechTree
		$MenuBlurBackground/TechTreePanel/UsTechTree/GroundBranch.visible = true
		$MenuBlurBackground/TechTreePanel/UsTechTree/HeliBranch.visible = false


func _on_heli_button_pressed() -> void:
	print("Вкладка: Вертолёты")
	
	if current_nation == "RU":
		$MenuBlurBackground/TechTreePanel/RuTechTree/GroundBranch.visible = false
		$MenuBlurBackground/TechTreePanel/RuTechTree/HeliBranch.visible = true
	elif current_nation == "US":
		$MenuBlurBackground/TechTreePanel/UsTechTree/GroundBranch.visible = false
		$MenuBlurBackground/TechTreePanel/UsTechTree/HeliBranch.visible = true


func _on_russia_pressed() -> void:
	cancel_slot_selection() # Сначала сбрасываем старый выбор, если он был!
	
	print("Выбрана нация: Россия")
	current_nation = "RU"
	unit_slots_container.modulate = Color("FFFFFF")
	ru_tech_tree.visible = true
	us_tech_tree.visible = false


# Нажали кнопку США на главном экране
func _on_usa_pressed() -> void:
	cancel_slot_selection() # Тоже сбрасываем выбор!
	
	print("Выбрана нация: США")
	current_nation = "US"
	unit_slots_container.modulate = Color("FFFFFF")
	ru_tech_tree.visible = false
	us_tech_tree.visible = true
# Эту функцию автоматически вызовет ЛЮБАЯ карточка танка или вертолета при клике
func select_unit_for_deck(unit_name: String) -> void:
	selected_unit_to_equip = unit_name
	is_selecting_slot = true
	print("Выбран юнит для деки: ", selected_unit_to_equip)
	
	# ВКЛЮЧАЕМ затемнение экрана для фокуса на слотах
	slot_focus_scrim.visible = true
	
	# Включаем зеленую подсветку самих слотов (как делали раньше)
	unit_slots_container.modulate = Color("00ff00") 
	
	tech_tree_panel.visible = false




# --- ОБНОВЛЕННАЯ ФУНКЦИЯ УСТАНОВКИ В СЛОТ ---
func equip_unit_to_slot(slot_number: int, slot_button: Button) -> void:
	if is_selecting_slot and selected_unit_to_equip != "":
		slot_button.text = selected_unit_to_equip
		print("Юнит ", selected_unit_to_equip, " успешно установлен в Слот ", slot_number)
		
		# Ищем крестик внутри этой кнопки слота и ПОКАЗЫВАЕМ его
		if slot_button.has_node("RemoveButton"):
			slot_button.get_node("RemoveButton").visible = true
		
		# Сбрасываем режимы и гасим фокус экрана
		is_selecting_slot = false
		selected_unit_to_equip = ""
		slot_focus_scrim.visible = false
		
		# Возвращаем родной цвет фракции для контейнера слотов
		if current_nation == "RU":
			unit_slots_container.modulate = Color("FFFFFF")
		elif current_nation == "US":
			unit_slots_container.modulate = Color("FFFFFF")
	else:
		print("Слот ", slot_number, " нажат, но юнит для установки не выб")
func clear_specific_slot(slot_button: Button) -> void:
	slot_button.text = "Пусто" # Делаем текст снова пустым, как при старте игры
	
	# Ищем крестик внутри кнопки и СКРЫВАЕМ его
	if slot_button.has_node("RemoveButton"):
		slot_button.get_node("RemoveButton").visible = false
		
	print("Слот успешно очищен.")


# Эта функция мгновенно отменяет режим выбора слота и гасит подсветку
func cancel_slot_selection() -> void:
	if is_selecting_slot:
		print("Выбор слота отменен игроком.")
		is_selecting_slot = false
		selected_unit_to_equip = ""
		
		# Прячем темный экран фокуса
		slot_focus_scrim.visible = false
		
		# Возвращаем слотам правильный цвет текущей нации
		if current_nation == "RU":
			unit_slots_container.modulate = Color("FFFFFF")
		elif current_nation == "US":
			unit_slots_container.modulate = Color("FFFFFF")
# Функции самих слотов, которые вызывают нашу универсальную команду:
func _on_slot_1_pressed() -> void:
	# Передаем номер слота и ссылку на саму кнопку слота
	equip_unit_to_slot(1, $BottomPanel/MarginContainer/UnitSlotsContainer/Slot1)

func _on_slot_2_pressed() -> void:
	equip_unit_to_slot(2, $BottomPanel/MarginContainer/UnitSlotsContainer/Slot2)

func _on_slot_3_pressed() -> void:
	equip_unit_to_slot(3, $BottomPanel/MarginContainer/UnitSlotsContainer/Slot3)

func _on_slot_4_pressed() -> void:
	equip_unit_to_slot(4, $BottomPanel/MarginContainer/UnitSlotsContainer/Slot4)

func _on_slot_5_pressed() -> void:
	equip_unit_to_slot(5, $BottomPanel/MarginContainer/UnitSlotsContainer/Slot5)

func _on_slot_6_pressed() -> void:
	equip_unit_to_slot(6, $BottomPanel/MarginContainer/UnitSlotsContainer/Slot6)


func _on_remove_1_button_pressed() -> void:
	clear_specific_slot($BottomPanel/MarginContainer/UnitSlotsContainer/Slot1)
	print("Выбор слота отменен игроком.")

func _on_remove_2_remove_pressed() -> void:
	clear_specific_slot($BottomPanel/MarginContainer/UnitSlotsContainer/Slot2)

func _on_remove_3_remove_pressed() -> void:
	clear_specific_slot($BottomPanel/MarginContainer/UnitSlotsContainer/Slot3)

func _on_remove_4_remove_pressed() -> void:
	clear_specific_slot($BottomPanel/MarginContainer/UnitSlotsContainer/Slot4)

func _on_remove_5_remove_pressed() -> void:
	clear_specific_slot($BottomPanel/MarginContainer/UnitSlotsContainer/Slot5)

func _on_remove_6_remove_pressed() -> void:
	clear_specific_slot($BottomPanel/MarginContainer/UnitSlotsContainer/Slot6)
extends Node2D

# Проверь, чтобы эти три пути точно вели к твоим узлам (можно перетащить с Ctrl)
@onready var battle_slots_container: HBoxContainer = $CanvasLayer/UnitSpawnPanel/BattleSlotsContainer
@onready var camera: Camera2D = $Camera2D

var tank_scene = preload("res://scenes/tank.tscn")

# Перечень ВСЕХ вертолетов в игре (для точной фильтрации HUD)
var helicopter_list: Array[String] = [
	"Ка-52 \"Аллигатор\"", 
	"Ми-28Н \"Ночной охотник\"", 
	"AH-64D Apache", 
	"AH-1Z Viper"
]

func _ready() -> void:
	print("Матч начался! Загружаем HUD боевого интерфейса...")
	# При старте матча сразу показываем наземку из деки
	refresh_spawn_hud("GROUND")


func refresh_spawn_hud(category: String) -> void:
	print("Обновляем HUD для категории: ", category)
	
	# 1. Полностью очищаем нижнюю панель от старых кнопок
	for child in battle_slots_container.get_children():
		child.queue_free()
		
	# Ждем один кадр, чтобы Godot физически удалил старые узлы из памяти
	await get_tree().process_frame
	
	# Безопасно достаем синглтон через корень
	var global = get_node("/root/GlobalData")
	if not global:
		print("Ошибка: GlobalData не найден в системе!")
		return
		
	print("Текущая дека в памяти: ", global.current_deck)
	
	# 2. Пробегаемся по сохраненной деке и создаем новые кнопки нового дизайна
		# 2. Пробегаемся по сохраненной деке и создаем новые кнопки нового дизайна
	for unit_name in global.current_deck:
		# ИГНОРИРУЕМ пустые слоты и слоты-заглушки со словом "Пусто"
		if unit_name == "" or unit_name == "Пусто" or unit_name == "[ Пусто ]":
			continue 

			
		# Проверяем, есть ли имя юнита в нашем списке вертолетов
		var is_heli: bool = unit_name in helicopter_list
		
		# Фильтруем кнопки под выбранную вкладку HUD
		if category == "GROUND" and is_heli:
			continue # Если открыта наземка, вертолеты не создаем
		if category == "HELI" and not is_heli:
			continue # Если открыты вертолеты, танки не создаем
			
		# 3. Создаем НОВУЮ кнопку вызова (Твой новый дизайн плашек боя!)
		var new_btn = Button.new()
		new_btn.text = unit_name
		new_btn.custom_minimum_size = Vector2(150, 100) # Пропорции кнопки в бою
		
		# Применяем Oswald, если он лежит в папке
		if ResourceLoader.exists("res://assets/fonts/Oswald-Bold.ttf"):
			var font = load("res://assets/fonts/Oswald-Bold.ttf")
			new_btn.add_theme_font_override("font", font)
			
		# Настраиваем стиль кнопки нового дизайна (делаем её плоской или кастомной)
		new_btn.flat = false # Можно сделать true, если хочешь плоские кнопки в бою
		
		# Связываем нажатие с физическим спавном на карте
		new_btn.pressed.connect(func(): spawn_unit(unit_name))
		
		# Закидываем кнопку в боевой контейнер
		battle_slots_container.add_child(new_btn)
		print("Успешно добавлена кнопка в HUD боя: ", unit_name)


# --- СИГНАЛЫ ПЕРЕКЛЮЧЕНИЯ ВКЛАДОК ЛЕВОЙ ПАНЕЛИ ---

func _on_ground_cat_button_pressed() -> void:
	refresh_spawn_hud("GROUND")

func _on_heli_cat_button_pressed() -> void:
	refresh_spawn_hud("HELI")


# --- СПАВН ЮНИТА ПО ЦЕНТРУ КАМЕРЫ ---
func spawn_unit(unit_type: String) -> void:
	print("Вызываем на поле боя: ", unit_type)
	var new_unit = tank_scene.instantiate()
	new_unit.position = camera.position
	add_child(new_unit)

