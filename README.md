# __Customized `Mephisto` with the Timeout Functionality for Empathic Conversation 2.0__

## __What's `Mephisto`__

[`Mephisto`](https://github.com/facebookresearch/Mephisto) is a flexible and extensible crowdsourcing framework developed by FAIR. Designed to streamline the process of collecting human data at scale, `Mephisto` provides the tools necessary to manage complex data collection tasks across multiple crowdsourcing platforms. In the Empathic Conversation 2.0 project, we use `Mephisto` to collect (two-party) conversations between two human workers about a recent news. During the collection process, there is a 20% chance that one of the two parties from a single chat session may not be able to connect to the session. Therefore, we developed a new timeout feature that sets the maximum wait time for the first connected worker. Afterward, we connect the worker with a chatbot (based on [`ParlAI`](https://parl.ai/docs/tutorial_basic.html)). And we archive this customized version used in our data collection process within this repository.

## __Important `Mephisto` Concepts__

`Mephisto` provides a plethora of APIs to incorporate various data collection scenarios, and, here, we only introduce the concepts required to understand our implementation of the timeout feature. Please refer to their [official documentation](https://mephisto.ai/docs/explanations/architecture_overview/) for the full architecture.

In this study, we consider conversations between two parties. In `Mephisto`'s language, a single chat session can be considered as an [`Assignment`](https://mephisto.ai/docs/explanations/architecture_overview/#assignment) composed of two [`Unit`'s](https://mephisto.ai/docs/explanations/architecture_overview/#unit). Each unit is the collection of work to be done (i.e., utterances) by one [`Worker`](https://mephisto.ai/docs/explanations/architecture_overview/#worker). When a worker is paired to the assigned unit, we may call it [`Agent`](https://mephisto.ai/docs/explanations/architecture_overview/#agent). So, to launch a conversation session, the out-of-the-box `Mephisto` requires every unit to be filled with a worker, and we make changes to this launching process.

## __Timeout Feature Implementation Explained__

### __How `Mephisto` Launches Assignments__

All we refer code snippets from this repository, __which is NOT the latest `Mephisto` implementation, and the logic is subject to change.__ The `_assign_unit_to_agent()` function on [L192]() is the core of this assignment launching process.

### __The Annoyingly Simple Timeout Implementation__

## __Example `Mephisto` Timeout Chat Session with `ParlAI`__
