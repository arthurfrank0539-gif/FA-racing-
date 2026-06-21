extends VehicleBody3D

# --- Vehicle Settings ---
@export var max_engine_force: float = 300.0
@export var max_steer_angle: float = 0.6
@export var brake_force: float = 50.0

# --- Booster Settings (Your Feature #3) ---
@export var boost_multiplier: float = 2.0
@export var max_boost_time: float = 1.5 # Booster lasts 1.5 seconds

var is_boosting: bool = false
var boost_timer: float = 0.0

func _physics_process(delta: float) -> void:
	# 1. Handle Regular Driving Inputs
	var engine_input = Input.get_axis("ui_down", "ui_up") # Arrow keys or WASD
	var steer_input = Input.get_axis("ui_right", "ui_left")
	
	# 2. Check for Booster Input (Spacebar)
	if Input.is_action_just_pressed("ui_accept") and not is_boosting:
		activate_booster()
		
	# 3. Manage Booster Timer
	if is_boosting:
		boost_timer -= delta
		if boost_timer <= 0:
			deactivate_booster()

	# 4. Apply Forces to the Wheels
	# If boosting, multiply the engine power!
	var current_force = max_engine_force
	if is_boosting:
		current_force *= boost_multiplier
		
	engine_force = engine_input * current_force
	steering = steer_input * max_steer_angle
	
	# Handbrake
	if Input.is_key_pressed(KEY_SPACE):
		brake = brake_force
	else:
		brake = 0.0

# --- Booster Functions ---
func activate_booster():
	is_boosting = true
	boost_timer = max_boost_time
	print("BOOST ACTIVATED!") 
	# Here you would also trigger a screen blur or tailpipe particle effect

func deactivate_booster():
	is_boosting = false
	print("Boost over.")
