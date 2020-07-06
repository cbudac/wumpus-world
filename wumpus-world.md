# Wumpus World - Prolog Code Analysis

We are group 2, and our members are: Cosmin Budac, Anish Basak, Yunfei Bai 

## Language Built-in Predicates and Constructs

Throughout the code certain language specific built-in predicates are used:

**dynamic/1** - informs that the predicate definitions may change during execution.

**format** - used to write formatted information to the console (defined by Quintus Prolog).

**writeln/1** - term writing predicate, locks the  output stream.

**retractall/1** - used to remove all the KB facts matching the head.

**assert/1**- used to add a fact or rule in the KB.

**( A -> B ; C )** - if then else construct.

- used to exit the recursion in **step_pre**

 

## High Level Mechanisms

1. **The starting point**

   The code is starting by invoking the **start** rule. The **start** rule is a conjunction between the **init** and the **take_steps** rule. The **take_steps** rule launches the recursion mechanism used explore the world.
   
2. **The world initialization**

   The **init** consists of four components:
	- **init_game**: initiallize environmental variables like time, score, visited, visited_cells, and clear the isWumpus and isGold.
	- **init_land_fig72**: define the size of the map, setup 1 gold location and 3 pit locations in the map.
	- **init_agent**: initialize agent location at bottom left corner, which is [1,1] in the map, and update visited_cells.
	- **init_wumpus**: setup a wumpus location.

	It must be noted that the location of the pits, gold and wumpus are not randomly assigned with each program run, and are fixed in reference to Figure 7.2 in Russel-Norvig's AI textbook. 


3. **The recursion** 

   After initialization, control shifts to the **take_steps** rule which tracks the agent's current percepts, updates the knowledge base, issues the next logical step, and alters the world state (time, score, agent's standing, etc.). If the agent does not fall into a pit, control is passed to the **step_pre rule** which utilizes an if-then-else construct to see if the game ends (agent obtains gold or is eaten). If neither case occurs, control is passed back to **take_steps** and the process repeats.
   

## Agent Step-Taking Mechanisms


### I. Sensing the world and creating the Percepts list (make_percept_sentence/1)

At every step, the agent will create a list bound to the Perception variable. This list contains information about the presence of smell, breeze or glitter. This information is used later to update the KB. 

Rules involved:

- **make_percept_sentence/1**
- smelly/1, bleezy/1, glittering/1 - with yes/no variants
- isSmelly/1, isBleezy/1, isGlittering/1
- adjacent/2

**Description:**

After the evaluation of the **make_percept_sentence** in the **take_steps** rule, the Perception variable will be bound to a list containing information about the current location. This is how it happens:

- make_percept_sentence([Stench, Bleeze, Glitter]) is matched (Perception is unified with [Stench, Bleeze, Glitter]). At this moment the 3 variables are not yet bound.

  

**Solving Smelly**

- smelly(yes) is matched (because is before smelly(no)). The AL variable is bound using the agent_location(AL) fact from the KB

- isSmelly(Ls1) is matched. The Ls1 variable is bound to AL, representing the current agent position, extracted from the KB.

- Ls2 is bound using the wumpus_location(Ls2) from the KB

- adjacent(Ls1,Ls2) is matched. If this fails, isSmelly(Ls1) fails and smelly(yes) fails. Backtracking stops as smelly(no) will be the next match. As a result the Stench variable is bound to **no**.


**Solving Bleezy and Glitter - same pattern as Solving Smelly**

- Once make_percept_sentence returns, the Perception variable will be something like: [no, no, no]
  
  
  
### II. Updating the Knowledge Base from the Percepts list (update_KB/1)

Once the percepts list within the Perception variable is initialized at the agent's current location, **update_KB/1** would pass the list values to subsidiary rules to update the KB's world state. This process would then be entailed by **ask_KB** to help the agent step into a new location. 

Rules involved:

- **update_KB/1**
- add_wumpus_KB/1, add_pit_KB/1, add_gold_KB/1 - with their respective yes or no percept variable variants (Stench, Bleeze or Glitter) as arguments.
- assume_wumpus/2, assume_pit/2, assume_gold/2 - with their respective yes or no atom and location list variants as arguments.

**Description:**

After the evaluation of the **make_percept_sentence** in the **take_steps** rule, the Perception variable with the updated percepts list would be bound to the **update_KB** rule. Taking the starting location of [1,1] as an example, the percepts list returns [no, no, no]. These values would be matched with add_wumpus_KB(no), add_pit_KB(no) and add_gold_KB(no) rules.  


**Updating the Wumpus KB**

- From the example, add_wumpus_KB(no) is matched as Smelly percept is not observed at location [1,1]. 

- Through a step-by-step process, assume_wumpus(no, L) rule is matched for every location L adjacent to the current agent_location([X,Y]).
  - The adjacent co-ordinates are calculated using mathematical expressions and stored within new variables (Z1, Z2, etc.).
  
- For each L adjacent location, assume_wumpus(no, L) retracts and asserts the dynamic predicate is_wumpus(no,L) within the KB. 

- A written confirmation on the KB update is provided using the pre-defined format predicate. 


**Updating the Pit KB**

- Similar to the pattern observed by Updating the Wumpus Location, add_pit_KB(no) rule is matched as Bleeze percept is not observed at location [1,1].

- assume_pit(no, L) rule is matched for every location L adjacent to the current agent_location([X,Y]) using the same mathematical expressions.

- For each L adjacent location, assume_pit(no, L) rule retracts and asserts the dynamic predicate is_pit(no, L) within the KB.

- A written confirmation on the KB update is provided using the format predicate. 
  
  

**Updating the Gold KB**

- add_gold_KB(no) is matched as Glitter percept is not observed at location [1,1].

- assume_gold(no, L) rule is matched, where L is the location of the agent (i.e [1,1]). This is different from the original code since in the original code it uses gold location as L and this is wrong to update. (Please refer to **Code Improvements** section at the bottom of this document)

- is_gold(no, L) is asserted and a written confirmation is provided. 

In the case where the agent is within a location where a percept(s) is indeed observed, the yes variants of the respective rules would be matched and a similar process is to the analysis above is followed. 

### III. Asking the Knowledge Base for a recommended action (ask_KB/2)

The **update_KB** rule has populated the KB with information about the existence of danger (Pit or Wumpus) and Gold in the adjacent cells. Practically the KB will contain facts like **isPit(no, [1,2])**, etc.

The **ask_KB** rule accepts two variables, the **VisitedList** that is already bound and contains the the visited locations and the **Action**  that will be bound in this rule and will be used to send information about the next move to the console:   format('I\'m going to: ~p~n', [Action]).

Essentially, this rule tries to bind a variable L to a position for which all of the following evaluate to true: isWumpus(no, L), isPit(no, L), permitted(L), not_member(L, VisitedList). The backtracking approach is used repeatedly, until such a L value is found. 

**Facts:**

isWumpus and isPit are simple facts and their simple presence in the KB will suffice.

**Solving permitted(L)**

- the WS variable is bound to the initialized size of the world (in our case, 4) through a query
- the next several rules will fail if we get a location beyond the world's boundary

**Solving not_member(L, VisitedList)**

- this rule has to variants that allow to recursively visit the list and break (through an if-then-else construct) when no match is found 

- the VisitedList is recursively matched against the location L
- the recursion is exited when **not_member(_, [])** is matched

**update_agent_location(L)**

- when this rule is evaluated, the secondary effect will be that the KB is augmented with the new agent location

So when this rule evaluates to true, the KB will end up containing the new agent location. The Action variable is bound to the new agent location and is used to send information to the console.


### IV. Game State Updates (update_time and update_score rules)
Game time and score are updated by 5 rules: **update_time/0, update_score/0, update_score/1, update_score/3 (1), update_score/3 (2)**.

**update_time** rule will fetch time value from the dynamic predicate time_taken/1 and then update the time by adding 1 in value. 

**update_score/0** rule will fetch agent location, gold location and wumpus location and then try to unify the update_score(AL, GL, WL) with two **update_score/3** rules in sequence. If it unifies with **update_score(AL, AL, _)**, it means agent is now on the gold, and 1000 points rewarded, else it will unify with **update_score(_,_,_)** and -1 point is rewarded. Finally, **update_score/1** is unified, to fetch the previous score from dynamic predicate score/1 and then to update it by either adding -1 or 1000 points as the new score.

After the time and score update, the action location is fetched from the agent_location predicate and then combined with past visited list to put it at the front of the list. Then the combined visited list will be passed to **step_pre/1** for the next iteration of recursion.


### V. Standing

The **standing**  rule allows the agent to recover when it falls into a pit. When this happens, the rule fails, forcing the parent  **take_steps** to try another location. **standing** uses three other rules - the **stnd**. These **stnd** rules should match the specific cases when the agent location is the same as the wumpus location or the gold location. However it seems that the implementation is incorrect as the more generic match is written first. The more concrete rules will never be matched and the recursion will be ended in the **step_pre** rule.

## Code Improvements

1. The **add_wumpus_KB(yes)** is missing, so even when you stand at [3,1] and smell the Stench, the KB won't be updated to indicate that surrounding might exists wumpus, and it will even Fail the **update_KB** rule and backtrack to change the Stench from "yes" to "no". To solve this, just to add **add_wumpus_KB(yes)**. This update also appears in our "wumpus_updated.pl" code submitted.
2. In the KB update, in the **add_gold_KB(no)** rule, the Body was fetching the gold location and use it to update the KB. It is wrong. Instead we should use agent location to update KB to let KB learn that current agent location is not gold location by **assume_gold(no, AL)**, and then the correct written confirmation can now be provided. (ex."KB learn [1,1] - there's no gold here!"). This update also appears in our "wumpus_updated.pl" code submitted.

