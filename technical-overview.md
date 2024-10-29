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

Compass leverages LLMs in four ways:

1. **Conversational Engagement**: Unlike typical applications where the LLM responds to user questions, Compass reverses this interaction. It generates questions to guide a **directed** and **grounded** conversation.
2. **Natural Language Processing Tasks**: Compass uses the LLM for tasks like clustering, named entity extraction, and classification, handling user inputs efficiently without the need for costly and time-consuming model training or fine-tuning.
3. **Explainability and Traceability**: The LLM provides reasoning for specific outputs, allowing for explanations that link discovered skills back to the user’s input. This feature, which is based on a variation of Chain of Thought reasoning, is especially noteworthy—not only for the capability it offers but also because it was an unplanned outcome. It emerged while attempting to solve a different problem: improving the accuracy of the LLM's tasks
4. **Filtering of Taxonomy output**: The LLM filters relevant skills and occupations connected to the ESCO model from the conversation's output. Leveraging its advanced reasoning capabilities, the LLM efficiently processes large amounts of text, identifying the most pertinent entities. This approach combines traditional entity linking via semantic search with LLM-based filtering, creating a hybrid solution.

Compass is grounded and protected from hallucinations in multiple ways:

* **Task Decomposition**: Smaller, more manageable tasks are assigned to individual agents with specific LLM prompts.
* **State-Induced Instructions**: Agents use their internal state to generate targeted instructions during user interactions, guiding the conversation toward a specific goal. This approach reduces the size of the prompt by including only relevant segments, making it more likely that the LLM will follow the instructions accurately, thereby reducing the risk of hallucinations.
* **Guided Output**: Instructions are carefully crafted to increase the likelihood of relevant responses. Techniques include:
  * One- and few-shot learning
  * Chain of Thought
  * Retrieval Augmented Generation
  * JSON schemas with validation and retries
  * Ordering output segments to align with semantic dependencies
* **State Guardrails**: Simple, rule-based decisions are made whenever possible, reducing reliance on the LLM and minimizing potential inaccuracies.
* **Taxonomy Grounding**: By linking entities to a predefined occupations/skills taxonomy, Compass ensures that identified skills remain within a relevant and accurate domain.

### Evaluation&#x20;

For evaluating Compass, we followed the strategy outlined below:

* **Rigorous Embeddings Evaluation**: We rigorously evaluated various strategies for generating embeddings from the taxonomy entities. Our considerations included identifying which properties of the entities should be included in the embeddings generation, as well as determining the optimal number of entities to balance accuracy and precision. For the tests, we used established datasets from the literature and generated synthetic data to mimic Compass user queries.
* **Isolated Component Testing**: Each agent's tools were evaluated individually using specific inputs and expected outputs. For example, classification components were tested with known inputs to verify accurate label assignments.
* **Scripted Conversations**: Conversational agents were tested using predefined dialogues, with outputs evaluated by either automated evaluators (other LLMs) or human inspectors.
* **Simulated User Interactions**: Compass was tested in end-to-end scenarios by simulating user interactions driven by an LLM. The simulated user was given a persona based on our UX research and additional instructions to cover specific cases of interest. These conversations were then assessed for quality and relevance by automated evaluators (other LLMs) or human inspectors.
* **User Testing and Trace Analysis**: On a smaller scale, real user tests were conducted. By tracing the top skill outputs back to the user's input, human inspectors could assess the performance of specific agents within Compass.

### The Core Role of a Taxonomy

Tabiya's inclusive taxonomy plays a central role in Compass. It grounds the LLM’s tasks, but there are several additional aspects worth mentioning:

* **Standardization**: The identified skills are linked to a standard taxonomy, making interpretation and comparison easier. The concepts behind these skills are well-defined and can be explained, allowing for clarification and disambiguation.
* **Canonicalization**: Explored skills are listed with canonical names and UUIDs, enabling consistent referencing across different experiences and applications.
* **Network Structure**: The taxonomy models the labor market by associating occupations with skills, forming a knowledge graph that can provide additional insights to users.
* **Unseen Economy**: The taxonomy has been extended to include activities from the unseen economy, empowering young women and first-time job seekers to enter the job market.
* **Localization**: A taxonomy can consider the specific context of a country. This includes occupations unique to certain regions, alternative names for occupations that are region-specific, and varying skill requirements for the same occupation across different countries.
* **Work Type Classification**: All experiences are classified into four types (wage employment, self-employment, unpaid trainee work, and unseen work), which allows for a more targeted exploration of the job seeker’s skills.

### Technical Stack Overview <a href="#technical-stack-overview" id="technical-stack-overview"></a>

* **Language Models and Embeddings**: Compass utilizes the `gemini-1.5-flash-001` model for its LLM capabilities and the `textembedding-gecko version 3` model for embeddings. The `gemini-1.5-pro-preview-0409` model is used for the LLM auto-evaluator. The Gemini model was chosen for its balanced performance across task accuracy, inference speed, rate availability, and cost.
* **Backend Technologies**: Developed with `Python 3.11`, `FastAPI 0.111`, and `Pydantic 2.7` for a performant server-side environment. An asynchronous framework suited the use case well, as LLM inference endpoints can be slow. Python was chosen for its extensive AI/ML library support and because it made it easier to integrate ML scientists into the development team.
* **Frontend Technologies**: The UI, built with `React.js 19`, `TypeScript 5`, and `Material UI 5`, is optimized for mobile but performs well on tablets and desktops. Additionally, we use `Storybook 8.1` to showcase, visually inspect, and test UI components in isolation.
* **Data Persistence**: Data is securely stored using `MongoDB Atlas`, which includes vector search capabilities. Our team was already familiar with `MongoDB`, and the taxonomy was already in `MongoDB Atlas`, so it was a natural choice.
* **Deployment**: The entire application is deployed on `Google Cloud Platform (GCP)`, ensuring high availability and scalability. We use `Pulumi` to deploy nearly all the infrastructure, as it allows us to write deployment code in `Python`, aligning with the rest of the backend development. Additionally, our team was already experienced with `Pulumi`, making it a natural choice. For error tracking and application performance monitoring, we use `Sentry`.

<figure><img src=".gitbook/assets/Compass Solution Archotecture - Compass _ Cloud Architecture.svg" alt=""><figcaption><p>Cloud architecture</p></figcaption></figure>
