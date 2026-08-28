---
name: telco-network-configuration
title: "AI Agent for Telecom Network Configuration Planning"
description: "Automate and optimize the configuration of radio access network (RAN) parameters using agentic AI and a large language model (LLM)-driven framework."
publisher: "nvidia"
type: blueprint
updated: "2026-02-17T19:33:44.952Z"
canonical: https://build.nvidia.com/nvidia/telco-network-configuration
source_url: https://build.nvidia.com/qc69jvmznzxy/telco-network-configuration.md
retrieved_at: 2026-08-20T06:20:07Z
card_format: build-source-backed-v1
---

# AI Agent for Telecom Network Configuration Planning

## Blueprint Card

| Field | Value |
| --- | --- |
| Blueprint ID | `telco-network-configuration` |
| Publisher | nvidia |
| Official page | [https://build.nvidia.com/nvidia/telco-network-configuration](https://build.nvidia.com/nvidia/telco-network-configuration) |
| Official Markdown source | [https://build.nvidia.com/qc69jvmznzxy/telco-network-configuration.md](https://build.nvidia.com/qc69jvmznzxy/telco-network-configuration.md) |
| Source snapshot | [../raw/telco-network-configuration.md](../raw/telco-network-configuration.md) |
| Last updated by publisher | 2026-02-17T19:33:44.952Z |

**Summary:** Automate and optimize the configuration of radio access network (RAN) parameters using agentic AI and a large language model (LLM)-driven framework.

## Public Blueprint Documentation

The content below is preserved from the official Build NVIDIA Blueprint Markdown
source retrieved on 2026-08-20T06:20:07Z. Build-relative links have been made absolute.

In a telco network, configuration and optimization involve managing a vast number of interdependent parameters—like TDD patterns, tx/rx gain, scheduling algorithms, MCS selection, handover thresholds, and power control—all of which directly affect network performance, user experience, and spectrum efficiency. These settings need constant tuning based on time of day, user behavior, mobility, interference, and service types.  

Given all the above, NVIDIA is developing an agentic AI solution to bring autonomy into this dynamic environment by observing real-time KPIs (like SNR, MCS, DL/UL bitrate, or LDPC decoder iterations), making data-driven decisions, and automatically adjusting parameters. Unlike traditional rule-based SON, an agentic AI can reason through complex trade-offs (e.g., higher DL bitrate vs. SNR degradation), learn from feedback loops, and adapt to new conditions without human intervention. It can also orchestrate changes across multiple layers (PHY/MAC to RRC) and multiple vendors, enabling coordinated actions like load balancing, inter-cell interference coordination, or power saving in lightly loaded areas. This level of autonomous control not only improves efficiency and QoS but also reduces operational complexity and time-to-resolution for issues in dense, high-demand RAN environments. 

##### **What Is an AI Blueprint, and How Does It Interact With the LTMs (Large Telco Models)?** 

NVIDIA AI Blueprints are customizable AI workflow examples that equip enterprise developers with NVIDIA NIM™ microservices, reference code, documentation, and a Helm chart for deployment. 

With the AI Blueprints, developers can now build and deploy custom AI agents. These AI agents act like “knowledge robots” that can reason, plan, and take action to quickly analyze large quantities of data, summarize information, and distill real-time insights. 

In an agentic AI solution, the core LLM (in this case, for network configuration planning) functions as the central intelligence, capable of interpreting natural language, reasoning over information, making decisions, and generating human-like responses. However, on its own, the LLM lacks memory, awareness of its environment, the ability to interact with external systems, and the capacity to persist actions over time. That’s where the agentic solution AI Blueprint comes in—it is the full technical and operational architecture that turns a raw LLM into an autonomous, goal-driven software agent. This AI Blueprint includes several critical layers. The memory module (often using vector databases like Pinecone or Redis) allows the agent to store and retrieve context, facts, or past interactions. The tooling and action layer connects the agent to real-world APIs, software systems (e.g., CRM, OSS/BSS), data sources, or command-line interfaces, enabling it to execute actions like triggering network reconfigurations or fetching metrics. A planner and orchestrator layer (built with frameworks like LangGraph, AutoGen, or CrewAI) allows the agent to break down complex tasks into sub-steps, reason about dependencies, and determine when to use a tool, call another agent, or loop through decisions. The observation and reflection loop gives the agent feedback about the outcomes of its actions, enabling it to retry, revise its approach, or escalate decisions. The AI Blueprint also includes a user interface layer (such as a chatbot, voice interface, or dashboard); a guardrail and governance layer for access control, output filtering, compliance, and monitoring; and a logging and observability stack for performance tracking and debugging. In total, while the LLM acts as the “brain,” the AI Blueprint defines the “body, sensors, memory, hands, and instincts”—everything needed to operationalize an autonomous agent in a complex enterprise like a telco. Together, they enable intelligent automation of tasks like network optimization, fault prediction, customer support, and service orchestration in a secure, scalable, and human-aligned way.

[Watch the Blueprint Tutorial](https://youtu.be/H5T6tkLCfTQ?si=0uJDlyfZ5e8-KcGK)

[Learn More About Large Telco Models](https://blogs.nvidia.com/blog/agentic-ai-blueprints/%20)

## Use Case Description
The goal of this NVIDIA AI Blueprint for telco network configuration is to develop an agent that provides insight to the end user to set optimal configuration values for specific RAN parameters for a cell-site, with the ability to extend to other areas beyond RAN, by utilizing historical (for insight) and real-time data (for monitoring) from cells with similar classifications.  

This AI Blueprint will ingest configuration data and historical KPIs from the network, and when the engineer (end-user) asks about the optimal value of a specific parameter, applies the logic via LLM to the aggregated data and returns a table. 

Traditionally, the optimal value for each configurable parameter is figured out by trials and experimentation over a long period of time, requiring a lot of skills that are hard to acquire. 

The LLM looks through large datasets and aggregates key performance KPIs of cells with similar classifiers and displays in a table how KPIs change by changing a parameter to different values.  

Once the user decides what value to configure based on these insights, the AI Blueprint runs the new config for the cell-site in the validation engine and compares the new real-time KPIs with existing ones by a weighted average method. It will monitor the new config for a given validation period (configurable), and if the new set of KPIs results in degradation (reduction in weighted average), it will revert to the original values. If there is no degradation (the weighted average of pre/post KPIs are above 0), the values will be kept. 

## Target Users
* RAN Operators: Utilize the framework to optimize network performance efficiently.   
* Network Engineers/Installers: Reduce manual effort and use data-driven configuration recommendations.   
* Telecom Companies: Improve network performance and customer satisfaction. Extend this use case for additional telco-related services.   
* Blueprint Users: Show the effectiveness of NIM-based LangGraph frameworks in addressing the domain-specific data and configurations. 

## Architecture Diagram
![Architecture Diagram](https://assets.ngc.nvidia.com/products/api-catalog/telco-network-configuration/diagram.jpg)

## Input Data
The datasets used in the NVIDIA AI Blueprint for telco network configuration are generated by BubbleRAN based on their [MX-PDK](https://bubbleran.com/products/mx-pdk/) 5G O-RAN platform deployed in the laboratory indoor environment. BubbleRAN’s MX-5G, OpenAirInterface-based 5G stack for RAN and CN, and MX-RIC and dataset collection xApp were used with NI USRP B210 connected to a commercial 5G UE. The dataset generation scenario attributes are summarized in the table below. 

| Attribute ID  | Attribute Name  | Description  |
| :---- | :---- | :---- |
| ATT-1  | Environment  | Laboratory, indoor  |
| ATT-2  | Interference  | Multi-cell interference present  |
| ATT-3  | User Equipment (UE)  | Single UE \- Google Pixel 5 and Quectel RM5xxG  |
| ATT-4  | Line-of-Sight (LoS)  | 1-3 m between UE and gNB  |
| ATT-5  | Mobility  | UE is stationary (no mobility)  |
| ATT-6  | Traffic Model  | Downlink TCP traffic is generated using iperf, with the Cubic congestion-control algorithm.  |

We have selected a set of 5G RAN parameters—some candidates that could reflect the trade-off between two KPIs, one of which (DL bitrate) is the goal and the other, the consequence (snr); i.e., to show how different values of a parameter can impact these two KPIs, offering users statistical insight to choose the sweet spot between improving one KPI vs degrading the other. 

The table below shows the 10 tunable gNB/DU configuration parameters chosen to generate datasets. 

| Config ID  | Name  | Description  | Impact on  SNR  |
| :---- | :---- | :---- | :---- |
| CONF-1  | P0\_Nominal  | Nominal power offset  | YES  |
| CONF-2  | DL Carrier Bandwidth  | Number of Physical Resource Blocks (PRBs) in downlink  | YES  |
| CONF-3  | UL Carrier Bandwidth  | Number of Physical Resource Blocks (PRBs) in uplink  | YES  |
| CONF-4  | TX\_gain  | Transmit gain  | YES  |
| CONF-5  | RX\_gain  | Receive gain  | YES  |
| CONF-6  | Power Ramping Step  | Power increase step in random access procedure  | NO  |
| CONF-7  | LDPC Iterations  | Number of iterations in the LDPC decoder  | NO  |
| CONF-8  | Center Frequency  | Center frequency of the NR carrier (Hz)  | NO  |
| CONF-9  | PMAX  | Maximum UE transmit power (dBm)  | NO  |
| CONF-10  | TDD Pattern  | TDD UL/DL slot configuration  | YES  |

The considered key performance indicators (KPIs) are summarized in the table below.  

| KPI ID  | KPI Name   | Description  |
| :---- | :---- | :---- |
| KPI-1  | DL\_NACK  | Number of DL retransmissions (retx) requested by the UE (due to failed DL reception; i.e., DL NACK count).  |
| KPI-2  | DL\_ACK  | Number of successful DL transmissions (i.e., DL transport blocks acknowledged by the UE).  |
| KPI-3  | DL Bitrate  | Average downlink bitrate measured at the MAC layer (bits per second).  |
| KPI-4  | DL MCS  | Modulation and coding scheme used for downlink transmissions.  |
| KPI-5  | UL SNR  | Measured UL signal-to-noise ratio.  |
| KPI-6  | UL MCS  | Modulation and coding scheme used for uplink transmissions.  |
| KPI-7  | UL\_NACK  | Number of retransmissions requested by the gNB (due to failed UL reception; i.e., UL NACK count).  |
| KPI-8  | UL\_ACK  | Number of UL transport blocks received without error (i.e., UL ACK count).  |
| KPI-9  | UL Bitrate  | Uplink bitrate measured at the MAC layer (bits per second).  |
| KPI-10  | LDPC\_iter  | Number of iterations used in the LDPC decoder for each decoding attempt.  |

## Details About LLM Implementation
In this NVIDIA AI Blueprint for telco network configuration, we leverage NVIDIA NIM microservices to deploy our agentic LLM-driven framework. After careful evaluation, we selected Llama 3.1-70B-Instruct as the foundational model, due to its robust performance in natural language understanding, reasoning, and tool calling. 

Customers have the flexibility to deploy this AI Blueprint via: 

* NVIDIA’s hosted NIM API endpoints at [build.nvidia.com](https://build.nvidia.com/) or 

* On-premises NIM microservices to meet privacy and latency requirements. 

End-users interact through a Streamlit-based user interface (UI) to submit their queries or initiate network operations. These queries are processed by a LangGraph Agentic Framework, which orchestrates the specialized LLM agents. 

The LLM agents are equipped with specialized tools that allow them to generate and execute SQL queries on both real-time and historical KPI data, calculate weighted average gains of the collected data, apply configuration changes, and handle the BubbleRAN environment. 

We leverage prompt-tuning to inject contextual knowledge about the BubbleRAN network architecture, including the setup details and the interdependencies between various KPIs and the logic for balancing trade-offs to optimize weighted average gains. 

The LangGraph-powered agentic framework orchestrates three specialized agents, each with distinct responsibilities that work together to close the loop of monitoring, configuration, and validation. Once the user initializes the network with selected parameters, they can choose between a monitoring session with a Monitoring Agent or directly query the Configuration Agent to understand parameter impacts and network status. 

Below is a breakdown of each agent and their functionality: 

1\. Monitoring Agent 

This agent continuously tracks the average weighted gain of pre-selected parameters in user-defined time intervals (default: 10 seconds) on real-time BubbleRAN KPI database. When it detects performance degradation due to reduction in weighted average gain of a specific parameter, it raises the issue to the user for authorization of the next step. 

2\. Configuration Agent 

The Configuration Agent can be activated by the Monitoring Agent’s hand-off or direct user queries about parameter optimization or network health. It analyses historical data and reasons through the analyzed trends and domain-specific knowledge of parameter interdependencies and trade-offs. Based on its analysis, it suggests improved parameter values to the user and waits for user confirmation. 

3\. Validation Agent 

Once parameter adjustments are confirmed, the Validation Agent restarts the network with the new parameter configuration. It evaluates the updated parameters over a user-configurable validation period and calculates the resulting average weighted gain. If the real-time average weighted gain deteriorates further, it automatically rolls back to the previous stable configuration. Otherwise, it confirms success and updates the UI with the new settings. 

In summary, our framework enables continuous, intelligent network optimization through an agentic loop, where specialized LLM agents work together to monitor, analyze, and validate parameter changes in real time. Equipped with tools to analyze real-time and historical KPI data, and with domain-specific knowledge of network parameters and trade-offs, these agents provide data-backed recommendations and explainable reasoning. This closed-loop design ensures that network performance remains autonomous yet user-controllable, empowering users to maintain optimal performance while retaining control on every decision point. 

## BubbleRAN 5G O-RAN Software
To simplify the deployment and make it available to everyone, we make use of Docker Compose to deploy an end-to-end 5G O-RAN network, including MX-5G and MX-RIC, with a single command. 

The deployment file features MX-5G, comprising the core network and radio access network, and MX-RIC, a radio intelligent controller. This powerful combination empowers end-users, including innovators like NVIDIA, to implement AI-driven solutions to perform comprehensive actions of the MX-5G infrastructure. 

We provide two Docker Compose configurations to suit different needs: 

1. Simulation Environment: This setup simulates user equipment (e.g., smartphones) connecting to the MX-5G network, ideal for development and testing. 

2. Real Device Connectivity: This configuration allows MX-5G to interact with actual user equipment, such as mobile phones or Quectel modules. Please note that this option necessitates coordination with BubbleRAN for environment provisioning, due to the requirement of radio hardware and specific modules. Alsonote that this option needs a USRP/RU and real UEs.  

The diagram below illustrates the network functions and xApp deployment workflow. In both configurations, all network functions come up automatically, and the xApp begins streaming KPI data into the database. 

![Image 1](https://assets.ngc.nvidia.com/products/api-catalog/telco-network-configuration/image1.jpg)

![Image 2](https://assets.ngc.nvidia.com/products/api-catalog/telco-network-configuration/image2.jpg)

For more information about the BubbleRAN turnkey 5G O-RAN solutions, please refer [here](https://bubbleran.com/products/mx-pdk/).

## Minimum System Requirements

### Licenses

* Artifacts and software are released under 4-clause BSD license 

* Documentation and datasets are released under CC BY 4.0 

### Hardware Requirements

* CPU: 12+ cores @ 3,8 GHz; AVX-512 is a must-have 

* RAM: 32 GB

* Tested on AMD Ryzen 9 9950X 16-Core Processor 

* Optional: NI USRP B210 

* Optional: QUECTEL RM520n 

* imsi \= "001010000000001" 

* key \= "fec86ba6eb707ed08905757b1bb44b8f" 

* opc \= "C42449363BBAD02B66D16BC975D77CC1" 

* dnn \= "oai" 

### OS Requirements

* Modern Linux (e.g., Ubuntu 22.04)

### Deployment Options

* Docker and Docker Compose (latest stable)

### Software Used in This Blueprint
#### NVIDIA Technology

* [Llama 3.1 70B Instruct NIM](https://build.nvidia.com/meta/llama-3_1-70b-instruct)

#### Third-Party Software

* [BubbleRAN software](https://bubbleran.com/) 

* [LangGraph](https://www.langchain.com/langgraph) 

* [SQLite3](https://www.sqlite.org/)

## Ethical Considerations
NVIDIA believes Trustworthy AI is a shared responsibility, and we have established policies and practices to enable development for a wide array of AI applications. When downloading or using models in accordance with our terms of service, developers should work with their supporting model team to ensure the models meet requirements for the relevant industry and use case and address unforeseen product misuse. For more detailed information on ethical considerations for the models, please see the Model Card++ Explainability, Bias, Safety & Security, and Privacy Subcards. Please report security vulnerabilities or NVIDIA AI concerns [here](https://www.nvidia.com/en-us/support/submit-security-vulnerability/).

## License
Use of the models in this AI Blueprint is governed by the [NVIDIA AI Foundation Models Community License](https://docs.nvidia.com/ai-foundation-models-community-license.pdf).

## Terms of Use
The software and materials are governed by the [NVIDIA Software License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-software-license-agreement/) and the [Product-Specific Terms for NVIDIA AI Products](https://www.nvidia.com/en-us/agreements/enterprise-software/product-specific-terms-for-ai-products/) , except that models are governed by the [AI Foundation Models Community License Agreement](https://www.nvidia.com/en-us/agreements/enterprise-software/nvidia-community-models-license/) and the [NVIDIA RAG dataset](https://github.com/NVIDIA-AI-Blueprints/rag/blob/main/data/LICENSE.DATA) is governed by the NVIDIA Asset License Agreement. 

Additional Information: for Meta/llama-3.1-70b-instruct model the Llama 3.1 Community License Agreement, for nvidia/llama-3.2-nv-embedqa-1b-v2model the Llama 3.2 Community License Agreement, and for nvidia/llama-3.2-nv-embedqa-1b-v2 model the Llama 3.2 Community License Agreement. Built with Llama.
