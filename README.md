
# CM30174/50206

Template project for Rover coursework (2019/20 session).



__<a name="bugs"></a>NOTE: You are encouraged to use this repository's issues tab to report bugs and requests new features where necessary.__



## What you need to get started 

1. Install [Eclipse](https://wiki.eclipse.org/Eclipse/Installation).
2. Install Jason [ V2.4](https://sourceforge.net/projects/jason/files/jason/version%202.4/jason-2.4.zip/download) & [Jason Eclipse plugin](http://jason.sourceforge.net/mini-tutorial/eclipse-plugin/).




## Contents
 - [The Environment](#env)
 - The Agent
   - [Agents actions within the environment](#awe)
     - [Scan](#scan)
     - [Move](#move)
     - [Collect](#collect)
     - [Deposit](#deposit)
  - [Action Feedback](#act_feed)
  	 - All Actions
  	 	 - [action\_completed\_feedback](#a_c_feed) 
  	 	 - [insufficent\_energy\_feedback](#i_e_feed) 
  	 	 - [invalid\_action\_feedback](#i_a_feed) 
  	 - Specific Feedback: Scanning
  	 	 - [resource\_not\_found\_feedback](#r_n_f_feed) 
  	 	 - [resource\_found\_feedback](#r_f_feed) 
  	 - Specific Feedback: Moving
  	 	 - [movement\_obstructed\_feedback](#m_o_feed) 
   - [Extending agents capabilities using Internal actions](#eac)
      -  [get\_remaining\_energy](#gre)
      -  [Important note & provisional internal actions](#pia)
 - [Updates](#updates)
 - Troubleshooting
   - [Ant is not properly configured](#ant)
   - [Error in Centralised MAS environment creation java.lang.ClassNotFoundException: rover.RoverWorld](#cnf)



## <a name="env"></a>The Environment

The RoverWorld environment is provided as pat of the `rover.jar` file located at `/libs/rover.jar`.
To set up the envirnment, the following line needs to be present in your mas2j file(s);

__`environment: rover.RoverWorld("mas2j_file=<mas2j_file_name>",  "scenario=<scenario_id>")`__

where;

**_mas2j\_file_**: Name of mas2j file or its path if it's located elsewhere.

**_scenario_**: An integer representing the id of the scenario to load.


example;
__`environment:	rover.RoverWorld("mas2j_file=getting_started.mas2j", "scenario=7")`__


_**NOTE**: A working example is available in the getting\_started.mas2j file_

## <a name="ag"></a>Agents
Agents are loaded via the mas2j file and each agent is expected to be configured before connecting to the environment.
Configuring an agent involves specifying its `capacity`, `scan_range`, `speed` and where necessary `resource_type`.

You have **9** points to share between `capacity`, `scan_range` and `speed`.

i.e. `speed` can not have a value below 0 or above 9 and where `speed` is 9, both `capacity` and `scan_range` must be 0.


The environment would confirm all agent configurations are valid. Where a violation is detected, an adequate error message would be displayed and the affected agent(s) would be reconfigured as follows: 

`capacity = 3, scan_range = 3 and speed = 3`

Adding an Agent to the environment requires the following line to be present in your coursework's mas2j file(s);

__`agents: <agent_name> [capacity=< val >, scan_range=< val >, speed=< val >"] #< instances >;`__

where;

__*agent_name*__: Name of agent to add. This is the same name as the Agent's `.asl` file

__*instances*__:  A positive non-zero integer that specifies the instances of that agent to add to the environment. 

example;
__`dummy_bot [capacity=3, scan_range=3, speed=3] #1;`__

In this example, one instance of the agent `dummy_bot` defined in the file `dummy_bot.asl` would be added into the environment. It would have a capacity of 3, a scan range of 3 and its speed will be 3.


In certain scenarios, there would be multiple resource types available on the map (Gold & Diamond). Agents may only collect resources of a particular type and this can be specified in the mas2j file. Where unspecified, the type of the first resource collected by the agent would be the only type it can collect.

To specify resource type, add the `resource_type` attribute to the agent declaration as shown below;

example;
__`dummy_bot [capacity=3, ange=3, speed=3, resource_type=Gold] #1;`__

The available options are `Gold` and `Diamond`. 



### <a name="awe"></a>Agents actions within the environment
Within the environment, an Agent would be able to `scan` the map, `move` around the map, `collect` resources and `deposit` collected resources. 

All actions  cost energy and would result in a feedback indicating the success or failure fo the action execution which would be delivered to agents in the form of `percepts`.

__Note:__ Staying idle also costs energy.



Below are the syntax for the above stated actions;

<a name="scan"></a>`scan:  scan(<scan_range>)`

	Example: scan(3) 
	Example note: This would allow an Agent scan the map at a range of 3

<a name="move"></a>`move: move(<x-displacement>, <y-displacement>, <speed>)`

	Example:  move(3, 3, 1)
	Example note: This would allow an agent to move 3 steps away from its current possition along the x-axis and 3 steps away from its current position along the y-axis at a speed of 1
	
	More info:
	x-displacement: - Distance away from current position to travel along the x-axis
	y-diaplacement:- Distance away from current position to travel along the y-axis
	speed:- travel speed
	
<a name="deposit"></a>`deposit: deposit(X)`

	Example: deposit(X)
	Example note: Deposits a single resource on the map. The value X denotes nothing. It is merely for syntax conformity. An agent would always collect one resource at a time.
	
<a name="collect"></a>`collect: collect(X)`

	Example: collect(X)
	Example note: Collects a single resource from the map. The value X denotes nothing. It is merely for syntax conformity. An agent would always deposit one resource at a time. 


### <a name="act_feed"></a> Action Feedback

#### <a name="a_act_feed"></a> All actions
Performing any of the allowed actions (`scan`, `move`, `collect` and `deposit`) may result in any of the following feedback

##### <a name="a_c_feed"></a> 1. action completed feedback:
When an Agent successfully performs an action, a feedback is returned to inform it of the success.

	Format: action_completed(<action_name>);
	Example: action_completed(move);
	Example note: Informs the agent that it has successfully moved.

##### <a name="i_e_feed"></a> 2. insufficient energy feedback:
	
Insufficent energy feedback is returned when an Agent did not have sufficient energy to complete the action it just attempted. An insufficient energy feedback is a sign of action execution failure.

	Format:insufficient_energy(<action_name>, <remaining_energy>);
	Example: insufficient_energy(move, 1.0);
	Example note: Infroms the agent that it was unable to move and has 
					 a current energy level of 1.0.
	
##### <a name="i_a_feed"></a> 3. invalid action feedback:
When an Agent attempts to perform an action that has not been properly specified or an action that is not permitted within the environment, a failure feedback is returned indicting the invalid action. 

	Format: invalid_action(<action_name>, <reason>);
	Example: invalid_action(jump, UNPERMITTED);
	Example note: Informs the agent that the action jump is not permitted in the environment. 
	
The three possible reasons are `UNPERMITTED`, `INVALID_PARAM` and `UNMET_REQUIREMENT`

The `UNPERMITTED`reason informs us that the an Agent is trying to perform an action that is not permitted in the environment. 

The `INVALID_PARAM` reason informs us that an invalid parameter was passed to an action. For instance trying to move with a negative speed. 

The `UNMET_REQUIREMENT` reason informs us that a requirement to perform the stated action has not been met. For instance, an agent trying to deposit a resource when it does not have any or when it has not be configured to carry a resource.
	
__Note:__ An invalid action feedback would be raised for known action types in the following events;

##### __- scan__: If the Agent does not have the ability to scan or tries to scan an invalid range such as ` a negative range` or `a non numeric range`.

##### __- move__: If the Agent does not have the ability to move or tries to move to an invalid location (`non numeric coordinate`) or tries to move at an invalid speed (`a negative speed` or `a non numeric speed`).

##### __- collect__: If the Agent tries to collect resource from a tile that has no resoure or if the Agent has no space to carry another resource.

##### __- deposit__: If the Agent tries to deposit when it is not carrying a resource. 


#### Specific Feedback: Scanning

##### <a name="r_n_f_feed"></a> 1. resource\_not\_found feedback:
If an Agent scans and doesn't find any resource, a resource\_not\_found feedback is returned. 

	Format: resource_not_found
	Example: resurce_not_found
	Example note: An agent scanned a portion of the map and found nothing.

##### <a name="r_f_feed"></a> 2. resource\_found feedback:
If an Agent scans a portion of the map and finds one or more resources.

		Format:  resource_found(<artefact_type>, <artefact_qty>, <x_dist>, <y_dist>)
		
where;

__*artefact_type*__: Item found on map such as `gold`, `diamond` or `obstacle`.

__*artefact_qty*__: The qty found at that location.

__*x_dist*__: Artefact distance away from agent on the x-axis.

__*y_dist*__: Artefact diatance away from agent on the y-axis.

		Example: resource_found(gold,10,1,1)
		Example note: The Agent found 10 deposits of gold 1 step away on the x-axis and 1 step away on the y-axis.

#### Specific Feedback: Moving

##### <a name="m_o_feed"></a> 1. movement_obstructed feedback:
If an Agent runs into another Agent on it's way, this would result in a movement_obstructed feedback 

	Format: movement_obstructed(<x_travelled>,<y_travelled>,<x_left>, <y_left>)

where;

__*x_travelled*__: Distance travelled along the x-axis before the collission.
	
__*y_travelled*__: Distance travelled along the y-axis before the collission.

__*x_left*__: Distance from destination along the x-axis.

__*y_left*__: Distance from destination along the y-axis.


	Example: movement_obstructed(1,1,3,4)
	Example note: The Agent successfully travelled 1 step on both its x and y axis and the distance away from its initial destination is 3 steps away on the x-axis and 4 steps away on the y-axis.

   
   
### <a name="eac"></a>Extending the capability of Agents

Apart from the actions listed above, an Agent may extend its capability via internal actions.
Some internal actions have been provided for you such as;

#### <a name="gre"></a>*get\_remaining\_energy*: 
- usage:  __*get\_remaining\_energy(X)*__

This would store the Agent's current energy of type double in the variable X. 

__<a name="pia"></a>NOTE__: For this coursework, you are limited to create internal actions for navigation purposes only. A very basic implementation of this has been made available in the `libs/rover.jar` jar file under the package `rover.ia`. This implementation is made available to get you started and is not optimised __(unsuitable for submission)__. These internal actions would be demonstrated during the lab sessions and are briefly discussed below;

- __*add_journey*__: Keeps track of movement made by the Agent.
	`usage: rover.ia.add_journey (X, Y)`
- __*clear_map*__: Clears the map the Agent built.
        `usage: rover.ia.clear_map`
- __*get_base_location*__: Retrieves the location of the base.
       `usage: rover.ia.get_base_location(X,Y)`
- __*get_config*__: Retrieves the agent's configuration (capacity, range, speed and preferred resource type)
        `usage: rover.ia.get_config(C,R,S,P)`
- __*random*__: Generates a random number between 0 and 9.
      `usage: rover.ia.random(N)`


##  <a name="update"></a>Updates

Updates to the environment would result in  a new release which can be found in the `releases` tab of this repository. 









## Troubleshooting

**If you run into any issue that has not been documented here contact the course tutors ASAP.**
