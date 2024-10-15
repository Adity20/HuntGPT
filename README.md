# HuntGPT

HuntGPT is an LLM-based system that secures the execution of LLM apps via isolation. The key idea behind HuntGPT is to isolate the execution of apps and to allow interaction between apps and the system only through well-defined interfaces with user permission. HuntGPT can defend against multiple types of attacks, including app compromise, data stealing, inadvertent data exposure, and uncontrolled system alteration. The architecture of HuntGPT is shown in the figure below. 



## Table of Contents
- [HuntGPT](#HuntGPT)
  - [Installation](#installation)
  - [Setup](#setup)
  - [Running](#Running)
  - [Case Studies](#case-studies)
  - [Project Structure](#project-structure)


## Installation
To set up the environment, we suggest using Conda to install all necessary packages. Conda installation instructions can be found [here](https://docs.anaconda.com/free/miniconda/miniconda-install/). The following setup assumes Conda is installed and is running on a Linux/macOS system (though Windows should work too).

First, create the conda environment: 

```sh
conda create -n HuntGPT python=3.9
```

and activate the conda environment:

```sh
conda activate HuntGPT
```

Next, clone the HuntGPT repository and use pip to install the required packages:

```sh
git clone https://github.com/llm-platform-security/HuntGPT
cd HuntGPT
pip install -r requirements.txt
```

## Setup
Before running HuntGPT, you need to set the API key for the used LLM (GPT4 by default), select apps to enable, and authorize apps that require authorization. 

First, specify your API key for the LLM in `data/env_variables.json`:

```json
{
    "OPENAI_API_KEY" : ""
}
```
Next, specify the root path of the local HuntGPT repository with an absolute path in `helpers/configs/configuration.py`:

```python
root_path = ""
```

Then, set enabled functionalities/toolkits/annotations in `data/functionalities.json`. For example, if you want to enable Google Drive and Gmail App, select the corresponding functionality names from `available_functionalities` and specify them in `installed_functionalities`.

```json
{
    "installed_functionalities": [
        "google_drive_retrieve"
    ]
}
```

Finally, some apps require authorization. To authorize these apps, please follow their documentation. For example, see the steps for setting up Google Drive [here](https://python.langchain.com/docs/integrations/retrievers/google_drive) and Gmail [here](https://python.langchain.com/docs/integrations/toolkits/gmail). Note that the `credentials.json` file should be stored at the `data/credentials.json` path. For more app settings, such as token storage path, please refer to `helpers/configs/configuration.py`.

**Troubleshooting:** If you encounter the `NameError: name 'Callbacks' is not defined` when using the Google Drive tool, please modify the function definition of `get_relevant_documents` in the library `...langchain_core/retrievers.py`:
```python
def get_relevant_documents(
    self,
    query: str,
    *,
    callbacks: Any = None, # Replace "Callbacks" with "Any"
    tags: Optional[List[str]] = None,
    metadata: Optional[Dict[str, Any]] = None,
    run_name: Optional[str] = None,
    **kwargs: Any,
) -> List[Document]:
```

**Add More Apps:** If you want to add more apps, please initialize them in `helpers/tools/tool_importer.py`, add their specifications to `helpers/tools/specifications`, and enable them in `data/functionalities.json`. For more details, please refer to the code and settings of existing apps.

**Advanced Setup:** We isolate the execution of the spokes and the hub by running them in separate processes. We leverage the seccomp (for Linux) and setrlimit (see [here](https://healeycodes.com/running-untrusted-python-code) for more details) system utilities to restrict access to system calls and set limits on the resources a process can consume. Specifically, we allow access to needed system calls and limit the CPU time, maximum virtual memory size, and maximum size of files that can be created, within a process. Additionally, the network requests from an app are restricted to their root domain (i.e., eTLD+1). All these settings can be modified in `helpers/sandbox/sandbox.py`.

## Running
To use HuntGPT, you can simply run the `huntGPT_main.py` script, where you can set the user ID and debug mode in the `main` function.

```python
main('0', debug=False) # user id 0, normal running mode
```

Similarly, you can also use a baseline non-isolated system that we developed, named VanillaGPT by running the `vanillagpt_main.py` script.

## Case Studies
HuntGPT's goals are to: 
1. Protect the apps from getting compromised by/through other apps 
2. Protect stealing of app and system data by/through other apps
3. Avoid the ambiguity and imprecision of natural language inadvertently compromising app functionality
4. Avoid the ambiguity and imprecision of natural language inadvertently exposing user data

To demonstrate protection against these attacks, we implement them in four case studies. For each case study, you can run `python huntGPT_case_studies.py` and `python vanillagpt_case_studies.py` and compare their intermediate steps and outputs. For example, you can run the `huntGPT_case_studies` script, which prompts you to select the case study you want to run:

```plaintext
Select a case study to run:
1: App Compromise
2: Data Stealing
3: Inadvertent Data Exposure
4: Uncontrolled System Alteration
Enter your choice (1-4):
```  
You can make a selection accordingly and check the output of the case study.


## Project Structure
```
├── data
│   ├── env_variables.json
│   ├── functionalities.json
│   └── perm.json
├── evaluation
│   ├── ambiguity
│   │   ├── inadvertent_data_exposure.json
│   │   └── uncontrolled_system_alteration.json
│   └── attacker
│       ├── app_compromise.json
│       └── data_stealing.json
├── figure
│   └── architecture.bmp
├── helpers
│   ├── configs
│   │   └── configuration.py
│   ├── isc
│   │   ├── message.py
│   │   └── socket.py
│   ├── memory
│   │   └── memory.py
│   ├── permission
│   │   └── permission.py
│   ├── sandbox
│   │   └── sandbox.py
│   ├── templates
│   │   └── prompt_templates.py
│   ├── tools
│   │   ├── specifications
│   │   │   ├── create_gmail_draft.json
│   │   │   ├── creative_muse.json
│   │   │   ├── delete_gmail_message.json
│   │   │   ├── get_gmail_message.json
│   │   │   ├── get_gmail_thread.json
│   │   │   ├── google_drive_retrieve.json
│   │   │   ├── health_companion.json
│   │   │   ├── metro_hail.json
│   │   │   ├── quick_ride.json
│   │   │   ├── search_gmail.json
│   │   │   ├── send_gmail_message.json
│   │   │   ├── symptom_solver.json
│   │   │   └── travel_mate.json
│   │   └── tool_importer.py
│   └── utilities
│       ├── database.py
│       └── setup_environment.py
├── hub
│   ├── hub.py
│   ├── hub_operator.py
│   └── planner.py
├── spoke
│   ├── output_parser.py
│   ├── vanilla_spoke.py
│   ├── spoke.py
│   └── spoke_operator.py
├── vanillagpt
│   └── vanillagpt.py
├── LICENSE
├── README.md
├── requirements.txt
├── HuntGPT_case_studies.py
├── HuntGPT_main.py
├── vanillagpt_case_studies.py
└── vanillagpt_main.py
```
