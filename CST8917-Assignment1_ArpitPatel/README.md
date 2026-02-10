#### Name: Arpit Patel
#### Student No: 041159097
#### Course Code: CST8917
#### Assignment 1: Serverless Computing - Critical Analysis
#### Date: 10th Feb 2026 

## Part 1: Paper Summary

In Serverless Computing: One Step Forward, Two Steps Back, Hellerstein et al. critically examine first-generation serverless platforms, arguing that while they introduce meaningful improvements in cloud usability, they also impose architectural constraints that significantly hinder data-intensive and distributed computing (Hellerstein et al., 2019).

### Main Argument

The central thesis of the paper is that current serverless platforms—primarily Functions-as-a-Service (FaaS)—represent a partial advancement in cloud programming but ultimately fall short of enabling true cloud-scale innovation. The authors describe serverless computing as “one step forward” because it introduces autoscaling, fine-grained billing, and removes the burden of server management from developers. These features make it easier to deploy and operate applications at variable scale. However, serverless also takes “two steps back” by ignoring fundamental principles of efficient data processing and distributed system design. As a result, modern workloads that depend on state, coordination, and data locality are poorly supported, limiting the cloud’s potential as a platform for innovation rather than just service outsourcing (Section 1.2).

### Key Limitations of First-Generation Serverless Platforms

One major limitation identified is execution time constraints. Serverless functions are short-lived, with hard execution limits (e.g., 15 minutes in AWS Lambda), and there is no guarantee that subsequent invocations will run on the same underlying machine. This forces developers to treat functions as stateless, making long-running or iterative workloads inefficient or impossible (Section 3).

The paper also highlights severe communication and network limitations. Serverless functions are not directly network-addressable, meaning they cannot communicate with one another through low-latency messaging. Instead, all coordination must occur through shared storage services such as object stores or key-value databases. This design introduces significant I/O bottlenecks and high latency, especially as concurrency increases, making fine-grained distributed communication impractical (Section 3; Table 1).

Closely related to this is the data shipping anti-pattern. Because functions are isolated from data storage and cannot maintain local state, data must repeatedly be moved from storage to compute. The authors argue that this “shipping data to code” approach is inefficient in terms of latency, bandwidth, and cost, especially when compared to architectures that move computation closer to data (Section 3).

Another limitation is restricted access to hardware. Current FaaS platforms expose only generic CPU resources and limited memory, with no support for GPUs or other accelerators. This prevents serverless platforms from supporting modern workloads such as machine learning, high-performance analytics, or hardware-aware database systems (Section 3).

Finally, the authors argue that serverless platforms stymie distributed and stateful computing. The lack of direct communication, persistent identity, and coordination primitives makes it extremely costly to implement distributed protocols, consensus mechanisms, or stateful services. As a result, serverless systems are largely confined to embarrassingly parallel or orchestration-focused workloads (Section 3).

### Proposed Future Directions

To move beyond these limitations, the authors propose several characteristics for future cloud programming platforms. First, they argue for fluid code and data placement, allowing computation to move closer to data when beneficial rather than enforcing strict separation. Second, they call for long-running, addressable virtual agents that retain identity and state while remaining elastic, enabling efficient communication and coordination. Third, they emphasize the need for heterogeneous hardware support, allowing cloud applications to leverage specialized accelerators such as GPUs through higher-level abstractions. Together, these proposals aim to preserve the benefits of autoscaling while unlocking the full performance and innovation potential of the cloud (Section 4).


## Part 2: Azure Durable Functions Deep Dive

### Orchestration Model


Azure Durable Functions extend basic Functions-as-a-Service by introducing an orchestration model composed of client, orchestrator, and activity functions. Client functions initiate workflows, orchestrator functions define the control flow, and activity functions perform the actual work. Unlike basic FaaS, where functions execute independently with no built-in coordination, orchestrator functions can sequence tasks, handle branching logic, and manage retries in a deterministic manner. This design directly addresses the paper’s criticism that first-generation serverless platforms stymie distributed computing by forcing developers to stitch together workflows using slow external services. Durable Functions provide a structured programming model for coordination without requiring developers to manually manage queues or state transitions. However, orchestration logic still relies on storage-backed state, meaning it improves programmability rather than eliminating the architectural separation between compute and data identified by Hellerstein et al. (2019).


### State Management

Durable Functions manage state using event sourcing, checkpointing, and replay. Each step executed by an orchestrator function is recorded as an event in durable storage, allowing the function to reconstruct its state by replaying events when restarted. This mechanism enables workflows to survive restarts, failures, and scaling events, effectively allowing stateful behavior on top of a stateless execution environment. This directly addresses the paper’s criticism that serverless functions are inherently stateless and short-lived. By persisting state automatically, Durable Functions remove the burden of manual state management from developers. However, because state is persisted to storage rather than retained in memory, communication latency and data access costs remain. As a result, Durable Functions mitigate—but do not fully eliminate—the data-shipping concerns raised in the paper (Hellerstein et al., 2019).


### Execution Timeouts

In standard Azure Functions, execution time is limited by platform-enforced timeouts, making long-running workflows impractical. Durable Functions bypass this limitation by allowing orchestrator functions to pause execution after checkpointing state and resume later through replay. This enables workflows to run for hours, days, or even weeks without violating execution limits. This design responds directly to the paper’s criticism that strict execution time constraints prevent iterative and long-running workloads. However, activity functions within a Durable Function orchestration are still subject to standard Azure Function timeout limits. As a result, long-running tasks must be decomposed into smaller activities. While this approach improves feasibility for extended workflows, it reinforces a decomposition model rather than enabling truly long-lived compute processes, meaning it partially—but not fully—resolves the limitations identified in first-generation serverless platforms. 

### Communication Between Functions

Durable Functions enable communication between orchestrator and activity functions through a managed messaging layer backed by Azure Storage. Orchestrators schedule activities and receive their results through persisted events rather than direct network communication. This approach improves reliability and determinism compared to ad hoc use of queues or object storage. It partially addresses the paper’s criticism that serverless functions rely on slow storage intermediaries for communication by providing a more structured abstraction. However, the underlying mechanism still depends on durable storage rather than direct, low-latency messaging. As a result, while developer experience is improved, the fundamental performance limitations described by Hellerstein et al. (2019) remain. Durable Functions make coordination easier, but they do not fundamentally change the communication model of serverless computing.

### Parallel Execution (Fan-Out/Fan-In)

Durable Functions natively support the fan-out/fan-in pattern, allowing orchestrator functions to schedule multiple activity functions in parallel and wait for their completion. This simplifies parallel execution and makes it easier to express distributed workloads compared to basic FaaS, where developers must manually manage concurrency using queues or external services. This capability directly addresses the paper’s concern that serverless platforms are poorly suited for distributed computing. Fan-out/fan-in enables structured parallelism and aggregation of results. However, coordination still occurs through storage-backed orchestration rather than direct inter-function communication. Consequently, while Durable Functions significantly improve the expressiveness of parallel workflows, they do not provide the fine-grained, low-latency coordination mechanisms required for many distributed systems described in the paper.



## Part 3: Critical Evaluation

### 1. Limitations that remain unresolved

Although Azure Durable Functions address several operational limitations of first-generation serverless platforms, some of the core criticisms identified by Hellerstein et al. (2019) remain unresolved or are only partially addressed. One such limitation is the data-shipping anti-pattern, which the authors identify as a fundamental architectural flaw of Functions-as-a-Service platforms. Durable Functions simplify state handling by persisting orchestration state to durable storage, but they do not alter the physical separation between compute and data. Activity functions still retrieve data from storage services rather than executing computation closer to where data resides. As a result, data-intensive workloads continue to incur latency, bandwidth overhead, and increased costs. While Durable Functions improve programmability, they do not resolve the performance inefficiencies caused by repeatedly moving data between storage and ephemeral compute instances.

A second unresolved limitation is the lack of direct, low-latency communication between functions, which continues to hinder distributed computing. Durable Functions coordinate execution through storage-backed messaging and event logs, ensuring reliability and fault tolerance. However, this design does not provide direct network addressability or fine-grained messaging between functions. Distributed systems often rely on fast communication to implement coordination protocols such as leader election, consensus, or synchronized parallelism. As described by Hellerstein et al. (2019), routing all communication through storage introduces significant latency and cost. Durable Functions reduce developer complexity but do not fundamentally change this communication model, leaving many distributed workloads impractical in a serverless environment.


### My Verdict

Azure Durable Functions represent meaningful but incremental progress toward the future of serverless computing envisioned by Hellerstein et al. (2019). They directly address several operational criticisms raised in the paper, particularly those related to stateless execution and strict time limits. Through event sourcing, checkpointing, and replay, Durable Functions enable long-running, stateful workflows that would be infeasible in basic FaaS platforms. Additionally, built-in orchestration and fan-out/fan-in patterns improve the expressiveness of parallel and asynchronous programming in the cloud.

However, Durable Functions largely operate as a layered workaround rather than a fundamental architectural shift. The platform preserves the core design choices of first-generation serverless computing, including storage-mediated communication and strict separation between compute and data. As a result, the deeper challenges emphasized in the paper—such as fluid code–data placement, low-latency distributed coordination, and hardware-aware execution—remain unaddressed. Durable Functions improve usability and reliability but stop short of enabling the data-centric and distributed innovation the authors argue is necessary for the cloud’s full potential.

In conclusion, Azure Durable Functions move serverless computing forward in practice but not in principle. They demonstrate how thoughtful abstractions can mitigate some limitations of serverless platforms, yet they do not fully realize the transformative vision outlined by Hellerstein et al. (2019). As such, Durable Functions should be viewed as an important evolutionary step rather than the endpoint of serverless cloud programming.






### AI Disclosure Statement

AI tools were used in a limited and transparent manner to support this assignment. ChatGPT was used to assist with summarizing the assigned research paper, organizing ideas, and improving structure in written explanations. All analysis, interpretation, and final content were reviewed, edited, and validated to ensure accuracy and understanding.






















### Reference

Hellerstein, J. M., Faleiro, J., Gonzalez, J. E., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). Serverless Computing: One Step Forward, Two Steps Back. Conference on Innovative Data Systems Research (CIDR). https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf

Microsoft. (6th April 2025). Azure Durable Functions overview. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=in-process%2Cnodejs-v3%2Cv1-model&pivots=csharp


Microsoft. (11th December 2025). Azure Durable orchestrations. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations?tabs=csharp-inproc

Microsoft. (22th March 2023).Fan-out/fan-in scenario in Durable Functions.  https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-cloud-backup?tabs=csharp