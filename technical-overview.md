# Technical Overview

### AI Architecture

Compass utilizes agentic workflows to interact with users, gather information, and identify their skills.&#x20;

Compass mimics how a human would approach a conversation and the resulting tasks, acting as an overarching agent that decomposes into smaller agents, each with its own responsibilities and goals.&#x20;

Each agent within Compass has a specific responsibility and performs multiple tasks to achieve its goal. For example, an agent might converse with a user to collect specific information, process that information, and prepare it for use by another agent.

<figure><img src=".gitbook/assets/Compass Solution Archotecture - Compass _ AI Architecture.svg" alt=""><figcaption><p>Compas AI architecture overview</p></figcaption></figure>

Agents maintain an internal state that allows them to apply a strategy to accomplish their goal, and have access to the user's conversation history and use tools based on LLM prompts (or not). These tools could be used for tasks such as conversing with the user, named entity extraction, classification, or transforming user input.

Agents are guided by a combination of instructions (prompts) and their internal state. The prompts can vary based on the agent's state to help them accomplish their tasks.

Once Compass has gathered all the necessary information from the user, it processes the data to identify the user’s top skills. This identification is done through a multi-stage pipeline that employs various techniques, such as clustering, classification, and entity linking the a occupations/skills taxonomy.

<figure><img src=".gitbook/assets/Compass Solution Archotecture - Compass _ AI Pipeline.svg" alt=""><figcaption><p>Detailed AI architecture with the multi-stage skills pipeline</p></figcaption></figure>

Compass leverages LLMs in three ways:

* To converse, generate questions, and respond to user input. This is a reverse application of LLMs, as they typically respond to user questions, but in Compass, the user is prompted by Compass to provide responses.
* To perform standard NLP tasks such as clustering, classification, and named entity extraction.
* To reason and provide explanations for the output of specific tasks.

Compass is grounded and protected from hallucinations in multiple ways:

* By giving the LLMs smaller tasks to solve.
* By using state to induce specific instructions during the conversation with the user.
* Using instructions that decrease the probability of irrelevant output from the LLMs.
* With the use of the occupations/skills taxonomy that confines the discovered skills to the given space.    &#x20;

Another important feature of Compass is its ability to offer traceability of outputs back to the user’s input. Because Compass includes reasoning for each task it performs, it allows for tracing and explaining why certain skills were selected and how they relate to the user’s experience.\


### Evaluation&#x20;

For evaluating Compass, we followed the strategy outlined bellow, tailored to the specifics of each agent:

* Agent’s tools where evaluated in isolation. Each component was evaluated based on specific inputs and expected outputs. For example, for component performing a classification task, we used known inputs and expected class labels to evaluate the tool.&#x20;
* Conversational components were evaluated with scripted conversations, where the pre-conducted conversation is treated as a given input and the expected output is evaluated by another LLM (auto-evaluators) or by human inspection.
* Additionally conversational components where evaluated, by simulating users driven by an LLM. The conversation between Compass and the simulated user was captured and evaluated by an auto-evaluator or by human inspection. This technique was applied to evaluate Compass as an end-end agent.&#x20;
* On a smaller scale, we conducted user tests and, following the traces of the top skills output to the user’s input, evaluated how well specific agents performed their duties.

### Stack

Compass utilizes the gemini-1.5-flash-001 model as its LLM and the textembedding-gecko version 3 model for embeddings. For the LLM auto-evaluator, the gemini-1.5-pro-preview-0409 model was used.

The backend is built with Python 3.11, FastAPI 0.111, and Pydantic 2.7. The frontend is developed with React.js 19, TypeScript 5, and Material UI 5. It is a web-based application designed primarily for mobile use but also works well on tablets and desktops.

Data is persisted using MongoDB Atlas, and Compass is deployed on GCP.

<figure><img src=".gitbook/assets/Compass Solution Archotecture - Compass _ Cloud Architecture.svg" alt=""><figcaption><p>Cloud architecture</p></figcaption></figure>
