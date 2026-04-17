
## Fundamentals

Threat hunting is an active, human-led, and often hypothesis-driven practice that systematically combs through network data to identify stealthy, advanced threats that evade existing security solutions.
## Threat Hunting Process

***Set the stage*** - Lay out clear targets and stay informed about the most recent cyber threats

***Formulate a hypothesis*** - Make educated predictions to guide the process. Using professional intuition, threat data, industry updates, and alerts from tools. Make sure this is testable.

***Design the hunt*** - Using your hypothesis, develop a strategy for hunting. Determine what data sources you need, the methods and tools that must be used, and the patterns you're looking for.

***Data gathering and Examination*** - Collect data and look for evidence that proves or refutes your process

***Evaluate findings*** - interpret your results

***Mitigate threats*** - If we confirm a threat, mitigate it appropriately and take the appropriate response actions

***Post-Hunt*** - Document everything and share your findings, methods, and outcomes. Learn from each threat hunt.

***Continuous learning*** - Each hunt should build off of and feed into the next



## Threat Hunting Glossary

Here are some common terms in threat hunting and their exact definitions:



***Adversary*** - An entity driven to compromise your business to meet their own goals

***APT (Advanced Persistent Threat)*** - Highly organized groups or nation states that process extensive resources, a much stronger adversary than normal

***TTPS:***

***T - Tactics -*** Stategic objectives and high level concepts employed by adversaries, example: Exfiltration

***T - Techniques -*** Specific methods utilized by an adversary to accomplish tactical objs, "how"

***P - procedures -*** Granular step by step instructions for doing a technique



Analyzing TTPs offers deep insights into how an adversary penetrates a network, moves laterally within it, and achieves their objectives. Understanding TTPs allows for the creation of Indicators of Compromise (IOCs), which can help detect and thwart future attacks.



***CTI -***  Cyber Threat Intelligence

***Indicator -*** An indicator, when analyzed in CTI, encompasses both technical data and contextual information. Data + Context = Indicator

***Threat -*** A multifaceted concept, consisting of 3 fundamental factors. Intent - the driving rationale to compromise your business. Capability - The tools, resources, and financial backing adversaries have. Opportunity - conditions or events  which provide favorable circumstances for an adversary to operate against you.



***Network Artifacts: These are residual traces of an attacker's activities within the network infrastructure.***

***Host Artifacts: On the other hand, host artifacts refer to remnants of malicious activity left on individual systems or endpoints.***



## Threat Intelligence Fundamentals



Cyber Threat Intelligence (CTI) represents a vital asset in our arsenal, providing essential insights to fortify our defenses against cyberattacks. They contribute crucial insights to our Security Operations Center (SOC).



The four fundamental principles that make up CTI are:



***Relevance*** - Information passed on must be relevant. Passing on an exploit for a piece of software we don't have makes no sense.

***Timeliness -*** Quick comms of intelligence is key, aged indicators lose relevance

***Actionability -*** Intelligence should be actionable. Unactionable intelligence is just un-productive.

***Accuracy -*** Before sending out intelligence, it should be verified for accuracy. Any concern for accuracy should come with a labeled confidence indicator.



When all 4 come together, it creates additional insights into adversary operations, enriches analyst data, uncovers adversary TTPs, and informs decision makers within our org



There are 3 types of intelligence, all three technically have slight overlap:

***Strategic:*** Consumed by C-suite executives, this intelligence helps drive bigger decision making initiatives

***Operational:*** Provides mid-level information on adversarial campaigns. Answers how? and where?

***Tactical:*** Technical very actionable intelligence containing IOC's for immediate action to be taken.


