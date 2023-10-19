# __Customized `Mephisto` with the Timeout Functionality for Empathic Conversation 2.0__

## __What's `Mephisto`__

[`Mephisto`](https://github.com/facebookresearch/Mephisto) is a flexible and extensible crowdsourcing framework developed by FAIR. Designed to streamline the process of collecting human data at scale, `Mephisto` provides the tools necessary to manage complex data collection tasks across multiple crowdsourcing platforms. In the Empathic Conversation 2.0 project, we use `Mephisto` to collect (two-party) conversations between two human workers about a recent news. During the collection process, there is a 20% chance that one of the two parties from a single chat session may not be able to connect to the session. Therefore, we developed a new timeout feature that sets the maximum wait time for the first connected worker. Afterward, we connect the worker with a chatbot (based on [`ParlAI`](https://parl.ai/docs/tutorial_basic.html)). And we archive this customized version used in our data collection process within this repository.

## __Important `Mephisto` Concepts__

`Mephisto` provides a plethora of APIs to incorporate various data collection scenarios, and, here, we only introduce the concepts required to understand our implementation of the timeout feature. Please refer to their [official documentation](https://mephisto.ai/docs/explanations/architecture_overview/) for the full architecture.

In this study, we consider conversations between two parties. In `Mephisto`'s language, a single chat session can be considered as an [`Assignment`](https://mephisto.ai/docs/explanations/architecture_overview/#assignment) composed of two [`Unit`'s](https://mephisto.ai/docs/explanations/architecture_overview/#unit). Each unit is the collection of work to be done (i.e., utterances) by one [`Worker`](https://mephisto.ai/docs/explanations/architecture_overview/#worker). When a worker is paired to the assigned unit, we may call it [`Agent`](https://mephisto.ai/docs/explanations/architecture_overview/#agent). So, to launch a conversation session, the out-of-the-box `Mephisto` requires every unit to be filled with a worker, and we make changes to this launching process.

## __Timeout Feature Implementation Explained__

### __How `Mephisto` Launches Assignments__

All we refer code snippets from this repository, __which is NOT the latest `Mephisto` implementation, and the logic is subject to change.__ The `_assign_unit_to_agent()` function on L192 of [`worker_pool.py`](./packages/Mephisto/mephisto/operations/worker_pool.py) is the core of this assignment launching process. Every time a worker gets onboard, this function try to pair the new worker with a free unit. Then, it checks whether all units are filled. If so, the assignment launches; otherwise, the function simply returns (with no return value).

### __The Annoyingly Simple Timeout Implementation__

The launching mechanism implies that the last worker is responsible for launching the assignment. So, we simply give the first joined worker a fallback control mechanism with `await asyncio.sleep(max_launch_timeout)` on L321. Once the agent awakes and finds any unfilled unit, it will invalidate the units and launch with a reduced list of agents. See other implementation details within the local function `_launch_assignment_after_timeout()` starting from L309. To set the maximum timeout seconds, we introduce a new parameter on L150 of [`task_run.py`](./packages/Mephisto/mephisto/data_model/task_run.py).

### __Who is Handling the Reduced Agent List__

For `ParlAI` chat task, the `run_assignment()` method on L213 from [`parlai_chat_task_runner.py`](./packages/Mephisto/mephisto/abstractions/blueprints/parlai_chat/parlai_chat_task_runner.py) is the driver of the task. It takes the list passed from `_assign_unit_to_agent()` and initialize a [`World`](https://parl.ai/docs/tutorial_worlds.html) object using the user-defined `ParlAIChatTaskRunner.parlai_world_module.make_world()` method on L223 or L228. You can find an example implementation on L157 of [`demo_worlds.py`](./tasks/parlai_chat_task_demo/demo_worlds.py) in the `ParlAI` chat example task, and this function is the actual implementation of `ParlAIChatTaskRunner.parlai_world_module.make_world()`.

## __Example `Mephisto` Timeout Chat Session with `ParlAI`__

As stated above, `make_world()` on L157 of [`demo_worlds.py`](./tasks/parlai_chat_task_demo/demo_worlds.py) defines how we handle the reduced list of agents. In this example, we instantiated a [`ParlAI Agent`](https://parl.ai/docs/tutorial_basic.html#agents) that forwards all messages to a remote chatbot hosted at `34.71.159.56:35496` (a GCP instance). Please refer to [this repository](https://github.com/Empathic-Conversations/ParlAI-Chat) for more information about how we host a chatbot for the EC2 project using `ParlAI`. Next, one sets the maximum timeout seconds as the L14 of [`base.yaml`](./tasks/parlai_chat_task_demo/hydra_configs/conf/base.yaml) config file. Finally, one launch the demo chat task via `python parlai_test_script.py conf=YOUR_CONF_DEFINED_UNDER_HYDRA_CONFIGS_WITH_NO_SUFFIX`.
