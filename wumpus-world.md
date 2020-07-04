# Wumpus-world - code analysis



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

2. **The world initialization** - Anish can work on expanding this section.

3. **The recursion** -  Yunfei, please see if you can talk about the **take_step** and **step_pre** recursion rules as well as the condition statement in **step_pre
   

## Agent Stepping Mechanisms


### Sensing the world and creating the Perception list (make_percept_sentence/1)

At every step, the agent will create a list bound to the Perception variable. This list contains information about the presence of smell, breeze or glitter. This information is used later to update the KB. 

Rules involved:

- **make_percept_sentence/1**
- smelly, bleezy, glittering - with yes/no variants
- isSmelly, isBleezy, isGlittering
- adjacent

**Description:**

After the evaluation of the **make_percept_sentence** in the **take_steps** rule, the Perception variable will be bound to a list containing information about the current location. This is how it happens:

- make_percept_sentence([Stench, Bleeze, Glitter]) is matched (Perception is unified with [Stench, Bleeze, Glitter]). At this moment the 3 variables are not yet bound.

  
  **Solving Smelly**

- smelly(yes) is matched (because is before smelly(no)). The AL variable is bound using the agent_location(AL) fact from the KB

- isSmelly(Ls1) is matched. The Ls1 variable is bound to AL, representing the current agent position, extracted from the KB.

- Ls2 is bound using the wumpus_location(Ls2) from the KB

- adjacent(Ls1,Ls2) is matched. If this fails, isSmelly(Ls1) fails and smelly(yes) fails. Backtracking stops as smelly(no) will be the next match. As a result the Stench variable is bound to **no**.


  **Solving Bleezy and Glitter - same pattern as Solving Smelly**

  Once make_percept_sentence returns, the Perception variable will be something like: [no, no, no]
  
  
  
### Updating the Knowledge Base from Percepts (update_KB/1)
  
  
  
  
### Asking the Knowledge Base for a recommended approach (ask_KB/2)
  
  
  
  
### Game State Updates (update_time and update_score rules)
  
  

