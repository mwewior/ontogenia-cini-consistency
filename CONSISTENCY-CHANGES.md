# Bench4KE &mdash; Consistency Update 

Project for Knowledge Engineering Course at UniBo. Semester - Summer 2026.

Authors:
- **Darian Severin Hach**
- **Mikołaj Wewiór**

---


## Overal description



---

## How to run

- Create a virtual enviroment and install dependencies from requirements.txt 
- Need 3 terminals, and in each, run:
    ```bash
    source .venv/bin/activate
    export OPENROUTER_API_KEY=sk-or-v1-...
    ```
- Then in the following terminals":
    1. Run Bench4KE:
        ```bash
        cd ./restapi
        uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
        ```

    1. Run external cq generator, e.g.:
        ```bash
        cd ./restapi
        python3 cq_generator_app.py 
        ```

    1. Run consistency evaluation:
        ```bash
        cd ./restapi
        python3 consistency_runner.py --submit --server-url http://127.0.0.1:8000 --config conf/consistency.yaml
        ```

---

## Changes - per functionality

Listed functionality and areas that have been changed.

1. ### Consistency evaluation

    Core functionality of this project.

    #### Files included
    1. new
        - `consistency.py`
        - `consistency_runner.py`
        - `consistency_evaluator.py`
        - `consistency_reporter.py`

    1. modified
        - `main.py`
        - `models.py`
        - `validation_pipeline.py`

1. ### Consistency evaluation configuration 

    Setting up the configuration and parameters for running the evaluation scripts.
    
    ### Files included

    1. new
        - `consistency_configuration.py`
        - `consistency.yaml`
    
    1. modified
        - `models.py`


1. ### New LLM provider &ndash; openrouter

    We added wrappers to use [Openrouter.ai](https://openrouter.ai/).
    It allows to use many of models from different authors easily, by many ways, such as OpenAI SDK.

    #### Files included

    1. modified
        - `cq_generator_app.py`
        - `cq_validator.py`
        - `hit_rate_evaluator.py`
        - `llm_clients.py`

1. ### Reusage of functions moved to external file

    Moved `run_validation_pipeline` outside of `cq_validation.py` to a module outside (`validation_pipeline.py`) that can be imported in other scripts.

    #### Files included

    1. new
        - `validation_pipeline.py`

    1. modified
        - `cq_validation.py`

---


## Changes - per file

Listed files with change description


### [*new*] consistency_runner.py
[./restapi/]

- Script to run the consistency evaluation compact and easily.


### [*new*] consistency.py
[./restapi/app/routers]

- A router to run the consistency evaluation with Bench4KE. Works on the same logic as `cq_validation.py`, but calls `consistency_evaluator.py` script.
- Separates complex consistency request to smaller subtasks, which are single Bench4KE validation runs.


### [*new*] consistency_evaluator.py
[./restapi/app/services]

- key component covering: reading configuration, collecting tasks, generating CQs (allowed in parallel), running validation, evaluation and plots generation.
- Both CQs Generation and Validation are prepared to run in parallel. As generation has no issues, then validation can meet sometimes HF/PyTorch errors related to reading/loading same resources while accessing then concurently. We runned the tests with `max_generation_workers` set to 15, and `max_validation_workers` set to 1. 


### [*new*] consistency_reporter.py
[./restapi/app/services]

- Calculates statistics as written in *CoLLM* paper.
- Examines if test is stable/consistent, minor variation, or unstable/inconsistent.
- Plots the results and saves them to the file.




### [*new*] consistency_config.py
[./restapi/app/services]

- Works as a script and calss to handle and pass configuration parameters properly. Written for Hydra-style configuration.


### [*new*] consistency.yaml
[./restapi/conf/]

- Added a file to configure key parameters for the consistency evaluation, such as: *n_runs*, *models*, *collm tests*, and others.


### main.py
[./restapi/app/]

- Added router to `consistency` service.


### models.py
[./restapi/app/]

- Added new classes for setting up the configuration
- Added new classes for customized Responses related to consistency testing






### cq_generator_app.py
[./restapi/]

- Openrouter wrapper added (new llm provider)
    - Exponential backoffs implemented when occured errors during generation
    - Added verification if selected models support temperature selection  
- Minor fixes and features: logger instances added, missing exceptions added

We also founded that way of generating CQs may be improved. We see that for each **single** question in selected project this app calls LLM with same prompt to generate **all** questions. So there is a lot of redundacy which is costly (each llm call costs real money) and takes some time.


### cq_validator.py
[./restapi/app/services]

- Added `provider` attribute in CQValidator class and in `get_llm_client` functino


### hit_rate_evaluator.py
[./restapi/app/services]

- Added `provider` parameter in `_call_llm` function and other related.


### llm_clients.py
[./restapi/app/utils]

- Added Openrouter wrapper for llm calls to this provider. It uses OpenAI SDK (via OpenAIAdapter class)





### cq_validation.py
[./restapi/app/routers/]

- Moved `run_validation_pipeline` and `normalize_cq_columns` functions outside of a script (to `services` directory), to be able to use it also from other files.


### [*new*] validation_pipeline.py
[./restapi/app/services]

- Moved the `run_validation_pipeline` from cq_validation to this file for allowing reusability of the code.
- Added double-checked locking (for parallel validation runs)





### requirements.txt
[./]

- updated libraries versions for `python-3.10`

---
