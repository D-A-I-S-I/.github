# Distributed Artificial Intelligence System for Intrusion detection (D.A.I.S.I)

**Authors:** [Victor Annell](https://github.com/VictorAnnell), [Johannes Liljedahl](https://github.com/johanneslil), [Gustav Pettersson](https://github.com/PetterGustavsson), [Kathrin Thenhausen](https://github.com/kathrinthenhausen)

**where ya wanna go?**
- [Abstract](#abstract)
- [Repositories](#repositories)
  - [Compute Node](#compute-node)
  - [Collector Node](#collector-node)
  - [ML Models](#ml-models)
- [Usage](#usage)
- [Acknowledgements](#acknowledgements)
- [Project Journey](#project-journey)
<br>

## Abstract

Intrusion detection system at the operating system and network level that integrates AI/ML for real-time threat detection and response across distributed compute nodes. The system can monitor system calls and network activities to identify and mitigate security breaches.

> **Disclaimer:**
This project was part of a university course at Uppsala University and comes with absolutely zero warranty. Despite the cutting edge nature of this project; Only use this system for cyber defence if you want false intrusion alerts and undetected intrusions :dancer:.<br>
Oh and also, this system was designed to work on **Linux** machines.


## Repositories

### Compute Node

A Compute node receives data from Collector node(s) and runs modules for different ml-models. The current models are *Autoencoder* (neural network) for the syscall module, and *Logistic Regressor* for network module. Incming data is written to each module's buffer, and then classified by the model, in a pipelined manner. The batched used for classification works as a sliding window in the buffer - pop some old data, append some new. See [`compute-node`](https://github.com/D-A-I-S-I/compute-node) for further details.

### Collector Node

A Collector node is a node which desires protection and therefore collects and sends network and syscall data to a Compute node (via Nats) for analysis. See [`collector-node`](https://github.com/D-A-I-S-I/collector-node) for further details.

### ML Models

This repository is used for model training. See readme in [`ml-models`](https://github.com/D-A-I-S-I/ml-models) for further details.

## Usage

See readme in [`compute-node`](https://github.com/D-A-I-S-I/compute-node) for complete installation and running instructions. For instructions on how to train the ML-models, see readme in [`ml-models`](https://github.com/D-A-I-S-I/ml-models).

## Acknowledgements

Syscall dataset from UNSW Sydney:

https://research.unsw.edu.au/projects/adfa-ids-datasets

Network model, flowmeter, and dataset:

https://github.com/deepfence/FlowMeter

## Behind the Scenes


### Project Journey
In the DAISI project's early phase, we focused on infrastructure setup, including selecting datasets, establishing GitHub repositories, and setting up communication channels. We then moved into implementing syscall and network ML models, setting up Docker, and facilitating node communication via NATS.

Subsequently, we enhanced our machine learning models, particularly focusing on an autoencoder neural network for syscall attacks, employing parallelized grid search for hyperparameter optimization, and leveraging FlowMeter for network data collection and classification. 

<!-- Week before christmas -->
The best performing trained models from the ML repository were moved in to the Compute repository, and loaded into the model instance for each module respectively. All components were fused together such that collecting and classifying data could be done live and continuously. 

### Obliterated Obstacles

- **NATS data transfer limit:**
NATS has a 64MB data-size limit, causing issues when sending large .pcap files to the network module. Flowmeter requires >100,000 packets for effective transformation into flow data, making the original .pcap files too large for NATS. To overcome this limitation, we implemented a solution where the collector sends smaller .pcap files to the compute node's network module, which stores them in a buffer. Once the buffer reaches its capacity (>100,000 packets), the files are merged into a single .pcap file and transformed into a .csv for the model. This approach also enables the use of a sliding window for more continuous network traffic analysis.


<!-- - **Concurrent subroutines - asyncio?** -->

### Future Work

- A system that actually works :eyes: 
- System/application logs module
- Pipeline the modules (look at the corresponding syscalls and network traffic)
- Better alert system/counter-measure (e.g. quarantine intrusive processes, disable network traffic of infected node)
- Multiple PID (syscalls) monitoring. <!--  Send PID to compute node to write to unique buffer for each PID. Then Syscall model can batch process from one buffer at a time (to separate which PID caused intrusion alert) in an interleaving manner. -->
