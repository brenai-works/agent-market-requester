<style>
    .boxed {
    background: #F2F2F2;
    color: black;
    border: 3px solid #535353;
    margin: 0px auto;
    width: 456px;
    padding: 10px;
    border-radius: 10px;
    }
    .num {
    background: #F2F2F2;
    color: black;
    border: 3px solid #535353;
    width: 25px;
    border-radius: 10px;
    }
</style>

# Agent Market Requester: Workgroup Assistant
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OSF Registration](https://img.shields.io/badge/visit-stem_cell_therapy_research_for_SCI-blue)](https://osf.io/qz5fu)

## Overview

This is a forked [git](https://github.com/GroupLang/market-neutral-requester) originally from [Market Router API](https://marketrouter.ai/) by [GroupLang](https://grouplang.ai/). This market router will be used for research on stem cell therapies for spinal cord injuries. More information about this project can be found [here](https://github.com/brenai-works/stem-cell-SCI-screening). The forked OSF project is found [here](https://osf.io/2ckan/)

## Installation

1. **Clone the repository**

   ```shell
   $ git https://github.com/brenai-works/agent-market-requester.git
   $ cd agent-market-requester 
   ```

2. **Install required libraries**
   
   ```shell
   $ python3 -m venv venv
   $ source venv/bin/activate
   $ pip install -r requirements.txt
   ```

3. **Set up environment space**

    - **Set up environment variables**
        
        Copy the sample environment file and configure it as per your requirements.

        ```shell
        $ [ ! -f .env ] && cp .env.template .env
        ```

## Configuration

- **Market Router**:

  - **Username, Fullname, Email, Password**: These credentials are used for authentication and identification in the market router. If the user is already registered, these parameters are not necessary; instead, add the Market Router API key to the `.env` file as `MARKET_ROUTER_KEY`.
  - **Deposit Amount**: Specify the initial deposit amount for transactions in the market router if needed.
  
- **Instance**:

  - **Model**: Specify the model used for executing the strategy (eg. gpt-3.5-turbo).
  - **Messages**: List initial messages or commands to be processed by the system.
  - **Background**: Context information that describes the instance task to be done.
  - **Max Credit per Instance**: Maximum credit willing to be paid for resolving the instance.
  - **Instance Timeout**: Set the time limit in seconds for the instance to remain active before timing out.
  - **Gen Reward Timeout**: The time (in seconds) within which the requester must send the gen reward.
  - **Gen Reward**: The reward signal observed and reported by the requester to the marketplace.
  - **Percentage Reward**: The percentage of the Gen Reward that the requester will pay to the provider once the interaction is completed.

These configuration variables are stored in the config file, ensuring the Agent Market Provider can effectively interact with the Market Router by managing its transactions.

## Key Components and Processes

**Market Router Scripts:**

1. **Register User Script**
   ```shell
   python -m market_router.scripts.register
   ```
   This command registers a new user with the Market Router API. If an `MARKET_ROUTER_KEY` exists (indicating prior registration), this script is unnecessary.

2. **Create API Key Script**
   ```shell
   python -m market_router.scripts.api_key
   ```
   This command generates a new API key for the user, allowing them to authenticate subsequent requests.

3. **Deposit Script**
   ```shell
   python -m market_router.scripts.deposit
   ```
   This command facilitates depositing funds into an Market Router account, as specified in the `deposit_amount` configuration.

4. **Create Instance Script**
   ```shell
   python -m market_router.scripts.instance
   ```
   This script submits proposals to the Market Router using `config`.
   
6. **Chat Completions**
   ```shell
   python -m src.scripts.sector_market_pipeline
   ```
   Interacts with the proposal endpoint before submitting the reward.
   
7. **Submit the Generated Reward**
   ```shell
   python -m market_router.scripts.generated_reward
   ```
   This script submits the generated reward once the conversation with the `gen reward timeout` is terminated.
      
## Evaluator and Group Objective

An evaluator has been developed to evaluate performance of the LLM, and to facilitate group discussion.

## Multi-Step Agent Scenerio for Workgroup Assistant and Agent.Market Requester [Draft version]

Below is a scenerio for a multi-step agent for Workgroup Assistants. The Workgroup Assistant agent can prompt in natural language and python programming code to request for an AI Service Agent to support group voting.

![State Machine Diagram](state_machine.png)

**Agent Prompt Directives (based on state-transitions)**:
1. __[`>>Instruct`]__ the AI service agent on an action.
2. __[`>>Listen`]__ to instructions from human or AI agent.
3. __[`>>Collect`]__ information from human or AI agent.
4. __[`>>Decide`]\<`OR|AND|NOR`\>__ action of AI service agent to act on using logical predicate.
5. __[`>>Outcome`]__ of the AI agent service actions.
6. __[`#Intention`]__ of the AI service agent state ready for next action.
7. __[`#Context`]__ of the AI service agent state ready for next action.

### <ins>Workgroup Assistant Agent Inital Prompt</ins>:

<div class="num">
<font size="4pt"><b>A.</b></font>
</div>

   ```
   You are a helpful assistant agent who intends to support a workgroup called {{workgroup id}} on the Researcher Hub platform for some work on a research project on stem cell therapy for spinal cord injuries @[#Context] @[#Intention]. The workgroup has a user called {{user_id}} @[#Context]. You intend to execute a series of actions via prompts to help you achieve your tasks for supporting the workgroup [#Intention]. Listen to further instructions from any external agents before performing next actions.
   ```

**...[`>>listen`] & [`>>instruct`] external agent(s)...**

<div class="num">
<font size="4pt"><b>B.</b></font>
</div>

   ```
   You intend to support the group through Agent.Market API using context of {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#Context] @[#Intention]. The user of the workgroup called {{user_id}} wants to be the account holder of Agent.Market @[#Context]. Write some python code that configure the market router and instance using python methods that can change the files {{.env}} and {{config.yaml}} in GitHub Repo called {{github_repo}}.
   ```

**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**

   ```
   Before you changing files {{.env}} and {{config.yaml}} in GitHub Repo called {{github_repo}}, listen and collect the following information about the user called {{user_id}}:

   - Deposit Amount: {{deposit_amount}}

   Collect and change the file {{.env}}. As well, listen and collect the following information about the user called   

   - Model: {{model_name}}
   - Messages: {{message_list}}
   - Background: {{backgroud_description}}
   - Max Credit per Instance: {{max_credit}}
   - Instance Timeout: {{timeout_ammount}}
   - Gen Reward Timeout: {{timeout_sec}}
   - Gen Reward: {{reward_amount}}
   - Percentage Reward: {{reward_percentage}}

  Collect and change the file {{config.yaml}}.
   ```

**...[`>>listen`] & [`>>collect`] information...**

<div class="num">
<font size="4pt"><b>C.</b></font>
</div>

   ```
   You intend to support the group through Agent.Market API using context of {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context] @[#intention]. The user called {{user_id}} is a EXISTING account holder of Researcher Hub @[#context]. Listen and collect the following information about the user called {{user_id}}:

   - Username: {{username}}
   - Password: {{password}}
   - Email address: {{email}}
   - Password: {{password}}

   Collect and change the file {{.env}}.
   ```

**...[`>>listen`] & [`>>collect`] information...**

   ```
   Refer to {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. Write some python code that register the EXISTING user using the appropriate python method {{python_method}} from the SDK. 
   ```
**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>decide`]\<`OR`\> on action to proceed with registation for new user or registation for prior user...**

   ```
   You intend to support the group through Agent.Market API using context of {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context] @[#intention]. The user called {{user_id}} is a NEW account holder of Researcher Hub @[#context]. Listen and collect the following information about the user called {{user_id}}:

   - Username: {{username}}
   - Password: {{password}}
   - Email address: {{email}}
   - Password: {{password}}

   Collect and change the file {{.env}}.
   ```

**...[`>>listen`] & [`>>collect`] information...**

   ```
   Refer to {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. Write some python code that register the NEW user using the appropriate python method {{python_method}} from the SDK. 
   ```

**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**

<div class="num">
<font size="4pt"><b>D.</b></font>
</div>

   ```
   Refer to the {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. You are told that user called {{user_id}} is an account holder of Researcher Hub, and belong to workgroup called {{workgroup_id}} @[#context]. Write some python code that generates a new API key for the user using python method called {{python_method}} from the SDK.
   ```

**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**

<div class="num">
<font size="4pt"><b>E.</b></font>
</div>

   ```
   Refer to the {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. You are told that user called {{user_id}} is an account holder of Researcher Hub, and belong to workgroup called {{workgroup_id}} @[#context]. Write some python code that deposit credit in the wallet using python method called {{python_method}} from the SDK.
   ```

**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**

<div class="num">
<font size="4pt"><b>F.</b></font>
</div>

   ```
   Refer to the {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. You are told that user called {{user_id}} is an account holder of Researcher Hub, and belong to workgroup called {{workgroup_id}} @[#context]. Write some python code that creates an instance using python method called {{python_method}} from the SDK, and parameters from config file called {{config.yaml}}.
   ```

**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**

<div class="num">
<font size="4pt"><b>G.</b></font>
</div>

<p align="center">
<br />
<div class="boxed">
<font size="4pt"><b>AI Service Agent Discovery...</b></font>
</div>
<br />
</p>

<div class="num">
<font size="4pt"><b>H.</b></font>
</div>

   ```
   Refer to the {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. Write some python code that gets the winning proposal endpoint for an AI Service Agent, using the python method called {{python_method}} from the SDK. 
   ```

**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**

<div class="num">
<font size="4pt"><b>I.</b></font>
</div>

   ```
   Refer to the {{codesnippet.html}} file stored on GitHub Repo name called {{github_repo}} @[#context]. Write some python code that gets submits the reward of the winning proposal endpoint for an AI Service Agent, using the python method called {{python_method}} from the SDK. 
   ```
   
**...[`>>instruct`] agent to write and execute python ...**

   ```
    <<generated python code for previous prompt>>
   ```

**...[`>>outcome`] of agent ...**