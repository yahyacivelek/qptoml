# sctoml

**sctoml** - State Chart TOML is used to represent state charts textually using TOML.

## Dining Philosopher Problem (Dpp) Example

I will explain the **sctoml** by taking classical dpp as an example which is used in qp in examples.

Below is the visual diagram of dpp state machines taken from qm tool:

![Table Active Object](img/Table.png)
![Philo Active Object](img/Philo.png)

First one Table and the second one is Philo active objects.

Here is textual representations of these objects in sctoml:

```
[[object]]
  name = "Table"
  initial = "serving"

  [object.states]
    active.events = [
      {sig = "TEST", target = "active"},
      {sig = "EAT", target = "active"}
    ]

    active.serving.enter = ""
    active.serving.events = [
      {sig = "HUNGARY", target = ["active", "active"], conditions = "bothfree"},
      {sig = "DONE", target = "active"},
      {sig = "EAT", target = "active"},
      {sig = "PAUSE", target = "paused"}
    ]

    active.paused.enter = ""
    active.paused.exit = ""
    active.paused.events = [
      {sig = "SERVE", target = "serving"},
      {sig = "HUNGARY", target = "paused"},
      {sig = "DONE", target = "paused"}
    ]

[[object]]
  name = "Philo"
  initial = "thinking"

  [object.states]
    thinking.initial = ""
    thinking.enter = ""
    thinking.exit = ""
    thinking.events = [
      {sig = "TIMEOUT", target = "hungary"},
      {sig = ["EAT", "DONE"], target = "thinking"},
    ]

    hungary.enter = ""
    hungary.events = [
      {
        sig = "EAT",
        target = ["eating", "hungary"],
        condition = ["Q_EVT_CAST(TableEvt)->philoNum == PHILO_ID(me)"]
      },
      {sig = "DONE", target = "hungary"}
    ]

    eating.enter = "BSP_displayPaused(1U);"
    eating.exit = "BSP_displayPaused(0U);"
    eating.events = [
      {sig = "TIMEOUT", target = "thinking"},
      {sig = ["EAT", "DONE"], target = "eating"}
    ]
```

Before starting to define each active object, you will need to type `[[objects]]` at the beginning.

```
name = "Table"
initial = "serving"
```

`name` is the name of the active object.
`initial` is the initial state of the active object.

```
[object.states]
     .
     .
active.serving.initial = ""
    active.serving.events = [
      {sig = "HUNGARY", target = ["active", "active"], conditions = "bothfree"},
      {sig = "DONE", target = "active"},
      {sig = "EAT", target = "active"},
      {sig = "PAUSE", target = "paused"}
    ]
```

Before starting to define state structure in an object you will need to type `[object.states]`.

`active.serving.enter = ""`

You can define enter/exit actions for each state.

`active.serving.initial = ""` assigns initial state to the *serving* state in *active* state. As it is '""', it has no initial state. This is optional if there is no initial state.

`active.serving.events = [...]` assigns an array of events that *serving* state in *active* state is waiting for. Right side of the expression **must** be an array type.

`{sig = "HUNGARY", target = ["active", "active"], conditions = "bothfree"}`:
Event signal name is *HUNGARY*. `target` is used to define states to be transitioned as a result of triggering *HUNGARY* event. `conditions` is used as guards to define which state to be transitioned. In this example, no transition occurs because the name of the target states (*active*) is the same as the name of the state.

If `target` has only one element, there is no need to assign an array. You are free to use an array, though.

If `conditions` has multiple elements, you **must** use array notation.

If `target` has only one element, you will not need to define `conditions`.

Number of elements in `condition` **must** be one element less than `target`.
