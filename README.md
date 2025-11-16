# State Machine

A small library to define finite state machines in Godot 4.

## Issues

- Defining states as different objects encourages an unhealthy amount of
  coupling since states want to operate on the FSM's parent's level.

## Usage

1. Add an instance of `StateMachine` to your root node.

```gdscript
@export_group("Internal")
@export var state_machine: StateMachine
```

2. Basic `State` provides functions for `update` and `fixed_update` to be
   called from `_process` and `_physics_process` respectively (different names
   are intentional, I didn't want update functions to blend with built-in
   signals). If you want to define any extra actions or to give states some
   information (e. g. a reference to the machine's parent to apply changes to),
   make a custom class to extend `State`.

```gdscript
class_name LCDState
extends State

@export_group("Internal")
@export var lcd: LCD

func setup(opts: Dictionary[String, Variant]) -> void:
    super(opts)

    if opts.has("lcd"):
        lcd = opts.lcd


func on_turn_on_animation_complete() -> void:
    pass


func on_turned_off() -> void:
    pass
```

3. For each state create a script describing the logic of this state. To
   transition to a different state, use the `transition` function.

```gdscript
class_name TurningOnLCDState
extends LCDState

@export var idle_state: LCDState

func setup(opts: Dictionary[String, Variant]) -> void:
    super(opts)

    if opts.has("next_state"):
        idle_state = opts.next_state


func enter() -> void:
    super()
    lcd.play_turn_on_animation()


func on_turn_on_animation_complete() -> void:
    super()
    transition(idle_state)
```

4. Initialize states in your root node:

```gdscript
func _ready() -> void:
    var turning_on_state := TurningOnLCDState.new()
    var idle_state := IdleLCDState.new()

    turning_on_state.setup({
        name = "TurningOn",
        lcd = self,
        next_state = idle_state
    })

    idle_state.setup({
        name = "Idle",
        lcd = self,
        on_state = turning_on_state
    })

    state_machine.add_state(turning_on_state)
    state_machine.add_state(idle_state)

    state_machine.transition(idle_state)
```

5. Add the necessary calls corresponding to actions to root node's code. Note
   that `update` and `fixed_update` calls are handled by `StateMachine`.

```gdscript
func on_turn_on_animation_complete() -> void:
   state_machine.current_state.on_turn_on_animation_complete()
```
