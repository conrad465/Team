Hello! I am working on an experiment where I use agents to manage the e2e product development process for a utility I am building. 

The idea: is inspired by khan academy videos that illustrates concepts, WhiteBoard AI is a framework that allows an LLM to create illustrative presentations without using expensive, slow video generation models. The framework consists of a JSON-oriented language and rendering engine. An LLM defines the presentation using the language and the engine can "animate it" using text-to-speech and SceneGraph. The key value of this utility is to speed up information transfer between LLMs (lightning fast thinkers) and people (not as fast). This approach combines cheap, fast text based generation capabilities of LLMs with human preference for animated visual information exchange.

The strategy: for building this application is a simple flywheel that connects Three elements: the product vision, the system, and user feedback. A strong product vision yields a system (codebase) that delivers low latency, engaging, and visual LLM-to-human interface that attracts users and yields user feedback that enables a stronger product vision. 

The approach overview: There will be four primary systems of record. The codebase, The product spec, and the Team document. The codebase and product spec will be owned by a product development agent, the strategy document will be co-owned by the agent and me, the human. finally user feedback is given by the user but the framework and collection system is part of the codebase/system. 

The Team document will be the method that everyone on the team uses to work together. It will define the overall strategy, share key information, state of the effort, tasks, the skills for each domain. Critical to the team document is a section where anyone can post suggestions / questions, including requests for new tools or skills for the rest of the team that could help us achieve our goals more effectively. 

Current state: I alone have worked with claude code to generate a PoC of an application that is currently hosted on GitHub Pages. For context, I am a student in business school. I am the sole user of this tool - it is not good enough yet for scaling to more people. There is a basic user feedback framework right now.

Note: A file named CLAUDE.draft.md may exist in this repository. It represents the previously promoted version of this file and is retained for diffing purposes only. Do not use it as guidance — this file (CLAUDE.md) is always the authoritative source.

Branch rules: Never checkout or switch to the main or master branch. All agent work happens on the agents branch (or a branch prefixed with agents/). If you need changes on main, create a PR — do not switch to it directly.