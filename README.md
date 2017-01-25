# eco-fps-walker
Artificial intelligence for characters moving in a 3D terrain in Godot.

As long as there is a map with collision feature (physics body), the bot implementing the AI will be able to move in this map.

## Features
### State machine
The bots implement a state machine that allows to configure easily complex behaviours with a simple declarative code.

### Travel with dynamic waypoints
The AI can use a navigation node to calculate an optimal path to a destination point. But rather than following each point of the calculated route, it will take the first waypoint to reach and move to this destination. When the waypoint is reached, a new route is calculated. And so on until the real destination point is reached. When the AI reaches the last waypoint, it will directly go to the final destination.

This implementation allows the bot to follow a moving target, like a human player. It also take in account difficult paths, like stairs and U-turn around a wall, which can cause problems when the bot has a large body. It is also tolerant to strange paths of the navigation mesh (won't be needed if the route calculation is improved in the future).

### No need for a navigation mesh
The bot can improve its path by using a navigation node but it's not mandatory. In the absence of a navmesh the target goes directly to the target. Of course it won't be able to avoid dead ends. But if there's no obstacle that requires path finding, this mode is very functionnal and usually it's enough.

### Avoid walls and holes
Wether the bot uses a navigation node or not, it will avoid holes and walls. Thanks to a concept of watching his steps the bot can detect if the path is clear. It uses a stereo detector which tells him if the hindrance is at its left or right, or both. The bot will then turn around and try another direction.

## Types of bots
There are 3 kinds of AI:
* walker_core : It's the core of the bots. This script allows a bot to go from point A to point B, with or without the help of a Navigation node. If there is a navigation node, the bot will follow the path with waypoints and try to avoid being stuck in funny situations generated by the navigation route calculation, such as extra waypoints in opposite direction. If there's no navigation node, the bot will move directly to the target.
* simple_bot : Extended version of the walker_core with the ability to attack a target. If the target dies, the bot will sleep until it gets a new target (it won't search by itself).
* basic_guard : Extended version of the simple_bot with the feature to scan an area in order to acquire a new target. When it has no current target, it will randomly rotate but not move, until a new target is in sight. This bot has a parameter 'target_group' which tells which group of nodes to search when looking for a new target. 

## Demos

## Installation
Either install from the asset store or copy the content of the addons folder in your project root folder.
The samples folder can be deleted.

## Usage
The usual way to use the bot is to add a node in the scene from one of the available bot nodes in /addons/eco.fps.walker/. You must select the .tscn node file.
Then you add as child of the node a visual mesh or node. The mesh will automatically move and rotate with the bot.

### Animation
It is possible to animated the mesh using information from the bot, like current speed or current state, by adding signal listeners.
The walker_core has 2 signals:
* walk_speed_changed : gives the current speed of the bot. The parameter is the speed of the bot.
* action_changed : sent when the bot's state changed. The parameter is the new state.

### Communication between bot and children
When the bot needs an event from the child, the recommended way is to implement a signal in the child node and extend the bot script to handle the signal event. For instance, a simple bot that attacks a target and has an attack animation must be able to send a signal to the bot for when the animation reaches the moment where the attack reaches the target. Also, when the attack animation ends, the bot must know that the attack ended and change its state to the next one.

## Add features
Those scripts are usually too basic to be used as it is in a game. They are meant to be extended, like in demo 8 where a simple patrolling behaviour is implemented while still attacking any target at sight. You can either use the simple_bot or basic_guard as a base for your own code, or start with walker_core to implement from almost scratch your own bot.

### State machine
Thanks to the state machine implemented in the core, the proper way to implement a bot is to use states rather than variables with conditions if-then-else. But it's not mandatory if your need is too specific to be implemented in the state machine. And you can add another state machines for your own usage if you feel the need to.

#### Declaration
The standard states are declared in one field called action_states, defined like this example:
const action_states=["sleep","move","turn","wait","my_state1","my_state2","some_state"]
It's a required step, unless you are happy with the default states.

#### Configuration
After the states are declared, it is required to configure them. The scripts have methods you can override to make your own configuration.

If you use the walker_core as base, you have only the method _init_fsm to override.

If you use basic_bot or basic_guard, you don't need to override _init_fsm but rather 3 sub-methods:
* _init_fsm_states : It creates the states and organize the groups.
* _init_fsm_move : It creates the links between states when the bot is moving or idling.
* _init_fsm_attack : It creates the links between states for when the bot is attacking a target.

Check the documentation of the state machine and the source code of the scripts for further information.

#### Change the state programmatically
The state machine usually does the update of the current state automatically with the given rules. But sometimes it's much easier to set a specific state from the script rather than the state machine rules.
In that case, you call the method:
   fsm_action.set_state(new_state_name)
where the fsm_action is the instance of the state machine, as defined in walker_core.

The signal action_changed will be triggered.

Beware that a wrong state name will cause an error and make the bot unstable.

## Tips and pitfalls

