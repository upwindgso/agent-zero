Agent KB style changes:
    => change core behaviour of save + retrieve successful solutions to
        [for each uniqu task in trajectory...may be multiple]
        - agent that experienced it (name and summary persona)
        - pi : p[roblem pattern (task type, strucutre, contraints)
        - goal/obj : inc success criteria + expected outputs
        - sequence: of reasoning and execution steps
        - context: domain + difficulty level
        - page 54 of https://arxiv.org/pdf/2507.06229
    => retreival bias towards recency X relevance
    => really want the objective to ahve more context
        => how to help meta search (beyond semantic)?????
        => There was the RCR-Router thing and MEM1 that seem to be converging on a simialr approach

=> probalby want to stop the auto memory and let/encourage the agent to make its own solution call after reflecting on task

Memory:
    => needs a speed up + granularity improvement...its slow and probably bloated
    => plug and play FAISS for local docker Mem0 with graph???
    => Doc ingestion: Docling for better structured outputs
        => looks like its a plug and play of the python.helpers.document_query.py function

? What does behaviour adjustment tool do? self modifying?
    

1. What is the optimal tuple for experience representation?
    - Agent KB talks about:
        problem pattern / goal/objective / sequence / context
    - There must be others
    - I feel like in a multi agent system where the orchestrator spins up dynamic agents to solve tasks...having some memory/context of the agent that was trying to perform the task is relevant?

2. Blending granularities is interesting:
    - RCR + MEM1 seem to imply less is more when agentic agents are operating
    - Yet memP seems to buck this trend by saying that more + less combined is better?
    - Is that because of the scope difference: ie performing vs strategising?

3. Knowledge organisation
    - how does strucural indexing work? give me a concrete ELI5 example

4. MetaAgent approach
    - (one of) MetaAgents core insight is to break the planning and the performing => Ie Have a reasoning + tool calling/delegating as seperate sequential agents
    - this mirrors things like openOperator / browserUse that have a specialist agent that operates the browser...the trick is in learning how to instruct the browser what you want
    - with the metagent approach the self relfeciton (i think) then reflects seperately on reasoning improvement and again on delegating
    - is this helpful/superior?

5. Context management
    - RCR approach of supplying context based on token budget
    - vs MEM1 constant compression
    - Use both or one or ? Recommendation

6. Reasoning/refinement
    - I think LAG relies on a finetuned model => Teacher-Student is more implementable..what does that look like incorporated with the MetaAgent approach?

7. I dont like the static procedural memory approach...its too brittle in practice / doesnt help generalisability



!!!! Robust implemetation of todo:
https://github.com/agent0ai/agent-zero/pull/670/files#
=> same as me except it forces when passing to subagent

Ok. Lets base our algorithm on MetaAgent with:
 - the reasoning step seems like it can be improved with 
    - LAG for initial plan (or at least cognitive load directing splits)
    - Teacher critic for ???review post execution???
 - Rather than tool calling we'll instantiate subagents dynamically (ie. create the system prompt for the agent and delegate the task to it to figure it out)
    - RCR-Router for initial context of invoked agents
    - MEM1 for the agent context management
        => but this req RL finetuning
        => important: IS = summarise past information + reasons about subsequent actions (ie dont forget information that might ebuseful in future)
 - this 'delegation' step is where our procedual memory generation process is important
    - unknown...whats the best way to ensure that the delegator has:
        - the best quality abstration of past experience it can draw on?
        - the best algorithm for dynamically translating the 'natural language task' into specific instructions for its worker sub agents?
        - => probably memP ....simplified (i think but confirm): combine trajectory + meta summary?
        - => running on agent KB tuples (above)
 
 So:
   Reasoner  =>  Delegator
   Reflect       no reflect
   via           jsut storage + memp adjustment strategy
   Teacher       defacto reflect next time proc is called
    
More deeply understand student/teacher model: apparently teacher monitors and intervenes real time (not at end)


+ Implement default vs specific agents for profiles (ie. Agent0 = Gemini 2.5 pro => Planner  = ...)
    - Default subagent for executing => GPT OSS 120b : seems comparable to Gemini for tool execution but for a fraction of the cost....if delegator does its job the default execution doesnt need deep reasoning


# Learning books
    - Its going to be some process that humans replicate
    - When learning model above is established with appropriate KG memory
    - FOCUS : Curriculum model scan book to extract lessons / things that can be done with the information => simple: by chapter
    - For each 'lesson':
        - CONCEPTUALISE : Student model reads book to apply knowledge : ok to bring in external knowledge but stick to book as primary
        - GROUNDING + COUNTER FACTUAL : Teacher model challenges to generalise learning
            - Grounding : (though storing in semantic?) Tether conceptualised outputs to real reason-act-observation experiences in episodic
            - Counter - Factual : Helps generalise + construct world model vs static knowledge "What would change if X?"
    