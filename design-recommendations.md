# Design recommendations for user interfaces to interact with digital twins

Facilitating interoperability of open research data from experiments and simulation requires a FAIR treatment of physical samples in the digital domain. Samples and experimental protocols that one may perform on them should be defined with sufficient, semantically annotated metadata to ensure reproducibility and interoperability. Furthermore, the history of a sample throughout its lifetime must be carefully tracked to ensure consistency between experimental and computational processes. The present document provides guidelines and best-practices for designing user interfaces to interact with the digital representations of physical samples and protocols in experimental and computational platforms. These guidelines are developed under the PREMISE project in establishing the infrastructure for experimental and simulation platform interoperability.

Throughout the document, a dedicated UI for the Aurora autonomous battery assembly and testing platform designed in the [AiiDAlab](https://www.aiidalab.net/) GUI companion of the [AiiDA](https://www.aiida.net/) WFMS as part of the [BIG-MAP Stakeholder Initiative](https://www.big-map.eu/) will be used to demonstrate the relevant recommendations and guidelines discussed in the document. The [Aurora-AiiDAlab](https://big-map.github.io/big-map-registry/apps/aiidalab-aurora.html) app has been used at the [Materials for Energy Conversion Laboratory](https://www.empa.ch/web/s501) at Empa to build, submit, and analyze results from coin-cell cycling experiments. The app is documented [online](https://aiidalab-aurora.readthedocs.io/en/latest/). Ongoing efforts are underway to modify the app’s interface towards interoperability with the [openBIS](https://openbis.ch/) ELN in a platform-independent manner following the guidelines of PREMISE’s deliverable [D1.1](https://github.com/ord-premise/interoperability-guidelines). These developments will further drive UI design guidelines and as such, will impact and accordingly update this document.

Where appropriate, a second example pertaining to [microscopy and spectroscopy experiments of molecules on surfaces](https://www.empa.ch/web/s205/atomistic-simulations) carried out at Empa may also be used as an example to highlight the challenges revolving the use of digital twins. Similarly, added insight from this use case will impact the guidelines discussed herein. Note that when discussing digital twins, the definition provided by [NASA](https://www.emacromall.com/reference/NASA-Modeling-Simulation-IT-Processing-Roadmap.pdf) is considered, with its three requirements: i. that the physical object is represented sufficiently and semantically in the digital domain; ii. that process outcomes are integrated into the digital twin (e.g. via experiment/process/dataset hierarchy); and iii. that the digital twin evolves in a closed loop by its processes.

With the above in mind, the following recommendations are based on the experience and insight gained in the first year of the PREMISE project. Note that the document is a living one, and as such, will be updated throughout the life of the project as its guidelines and recommendations are put to the test.

## Single source of truth

Ensuring reproducibility requires data integrity and trust. Due to the possibility of so-called out-of-band processes not tracked by a WFMS, it is recommended that an ELN be used as a single source of truth. As such, any modifications made to samples and protocols must be recorded accordingly in the ELN, regardless of the source of mutation.

## Recording digital twins in an ELN

The design and layouts of ELNs are likely to vary in implementation. However, at least in the field of materials science, some recommendations can be made regarding how a user should choose to layout their personal ELN instance. Though comments are made here with respect to the implementation of openBIS, the recommendations are meant more generally.

Samples and protocols are typically considered as parts in an inventory when constructing experiments. As such, when recording an experiment and its processes, the inventory is leveraged to associate samples and experiments. However, the specifics of the layout in which ELN nodes are attached with respect to an experiment is often left arbitrary. Here, it is recommended to consider the following:

- If a process is non-destructive, non-mutating, it may be attached as is If a process is non-destructive, but mutating, the mutation is applied directly to the sample as long as the history of an object is tracked internally by the ELN
- If a process is destructive in the sense that it transforms a sample beyond what is recordable, anew sample should be created. This is also true in the previous case if no history tracking is implemented internally

The concept of digital twin mutation is addressed further in the following section.

## Digital twin construction and mutation

In working with digital twins, a first step is often constructing the digital representation of the physical models and process involved in an experiment. When such digital representations are used in automated workflows involving both experimental and simulation platforms, it is crucial that the user interface provides room for sufficient input, both machine and user generated, to ensure reproducibility at both ends.

In the case of Aurora, where coin cell assembly is robot-driven, the UI becomes an integrated component of the assembly process. As such, a UI may be designed on either end to select from available coin cell components in preparation for assembly. If developed for an external (e.g. AiiDAlab), sample designs are to be recorded in the ELN upon assembly submission per the principle of the single source of truth. Note that due to technical limitations of the present instrumentation, AiiDAlab has yet to be connected with the assembly robot for this purpose. Future developments on this end will be mentioned here.

Other than the physical samples required for an experiment, one or more protocols must be defined to perform on the samples as an experiment. These could be predefined in the ELN in a manner consistent with community standards in the relevant field. Once a protocol is associated with at least one experiment, it must be frozen to ensure reproducibility. Note, however, that this is not necessarily required in the external control API (e.g. AiiDAlab), as long as the associated WFMS does not cache the protocol. In other words, protocols may be edited if and only if they are decoupled from the stored protocols associated with previous experiments. For example, if experiment 1 used protocol 1 in state A, a mutation of protocol 1 to state B may be performed if and only if experiment 1 is linked to protocol 1 in state A in the stored provenance of the WFMS.

Note that the above recommendation with respect to protocol mutation/freezing is offered after careful consideration of the consequence of mutation of protocols in an ELN. Consider an ELN implementation allowing for the mutation of protocols while keeping previously recorded experiments consistent by linking to historical states of a protocol. Though a fair approach, it can quickly lead to confusion after repeated mutations that may reach a protocol state with no overlapping properties of the initial state. As such, it is recommended to avoid the mutation entirely and maintain unique protocols per property set.

## Recording process outcomes (sample/process history)

Following the NASA definition requirement that a digital twin should evolve by its processes, a process should not create a new sample, but rather provide an outcome in the form of a dataset containing the raw data results of the process, and the new state to which the process evolved the sample. These are in turn communicated to the ELN for record keeping. If the experiment managed by the WFMS is multi-process, the WFMS is required to track sample state changes internally and report to the ELN the full experiment provenance.

On the ELN side, sample changes are recorded as mutations, each taking the sample to a new state. This forms a closed system with respect to sample mutation, in the sense that requested samples (and protocols) at the WFMS UI side are responded by the ELN with the latest state, ensuring consistency. However, as a further measure, it is crucial that on, or rather directly prior to submitting an experiment, the WFMS UI validates its components (samples and protocols) against the ELN as a final consistency check. Digital sample mutation constitutes a closed loop between samples and processes, as required by the NASA specification.

The last requirement of the digital twin specification is covered by the integration of process outcomes (datasets) as children of the sample. In the openBIS ELN, for example, a WFMS-managed experiment is unpacked and mapped onto a child hierarchy attached to the sample root node.

To illustrate the above, consider a typical use case of Aurora, where an experiment is performed on a sample consisting of one sample/electrolyte interface formation protocol, and one long-term cycling protocol. The user would construct said experiment from a sample and protocols acquired from an ELN (e.g. openBIS) in a platform-agnostic manner (see [D1.1](https://github.com/ord-premise/interoperability-guidelines)). The sample will contain the last known state, as will each protocol. Immediately prior to submission, a check is performed to validate state consistency. During the experiment, each protocol results in a state change, which is recorded and transferred to the next protocol. Upon completion, the experiment provenance is shipped to the ELN, which in turn unpacks and maps the experiment as a child of the sample. Each protocol is attached to the experiment (in order), and its associated dataset is attached as a child of the protocol. Finally, each resultant state change is recorded as a mutation to the associated sample.

## Designing the graphical user interface of the workflow manager

The above sections focus on the integration of experimental results into a digital twin and its evolution by these results, as stated in the NASA requirements for digital twins. This section and its subsections will focus on the design of a graphical user interface for building experiments and exploring and analyzing their results.

#### 1. The inventory

Experiments in materials science require at the least one sample and one process to apply to the sample. An experiment workflow may consist of several of these processes. As such, it is recommended to provide a section in the UI to manage samples and processes. The section should be carefully integrated and coordinated against the associated ELN to ensure data consistency.

It is recommended to include the following for samples:

- A detailed viewer – at least text-based for differentiation, though graphical viewers/editors, if feasible, could provide sample design and construction functionality
- An importer to upload existing samples from raw files of arbitrary format
- Sorting/filtering functionality for improved user experience
- Sample grouping to enhance sorting/filtering capabilities
- User-input widgets to modify samples (see above section on [digital twin mutations](#digital-twin-construction-and-mutation))

For experimental protocols, the following is recommended:

- A detailed viewer
- Sorting/filtering functionality for improved user experience
- A protocol editor

For both, it is crucial to ensure consistency against the associated ELN. As such, any modifications must be synchronized with the ELN in a way that protects the integrity of the provenances of both the ELN and the WFMS. See above sections on [digital twin mutations](#digital-twin-construction-and-mutation) for more detail.

#### 2. The constructor

With samples and protocols made available in the inventory, the user may proceed to construct experiments. For this purpose, it is recommended to design a wizard-like experiment builder to walk the user through the steps of constructing the experiment. These include: (i) sample and protocol selection steps, where sorting/filtering functionality is highly advised to enhance user experience; (ii) a configuration section for system-dependent setup (e.g. instrument settings, live monitoring, etc.); (iii) an input preview section for the user to verify their selection, where it is recommended to implement automatic validations; and (iv) submission controls.

#### 3. The analyzer

Users often wish to inspect and analyze the results of completed experiments. Though such features maybe available on the ELN side, it is recommended to integrate analysis tools into the same interface used for submission to improve user experience. In the Aurora AiiDAlab app, such a tool was implemented allowing users to quickly filter through previously submitted experiments (including custom experiment groups). Here, the user can view both completed and running experiments.

For completed experiments, the user can visualize post-processed data, both by series, and statistically aggregated results. Multiple plots may be shown at any given time, each providing on top of the visual representation a summary a text-based summary of the experimental results, and a set of plot controls to modify or download the plot. For running experiments where live monitoring was assigned during experiment construction, the user may view partial data and choose to terminate an experiment if deemed faulty.

#### 4. The explorer

Lastly, a user may wish to search for one or more previously submitted experiments. It is then recommended to include in the user interface an explorer section following the common design of web-based advanced search tools. These tools are often storage-system-dependent. For example, an explorer tool is currently being designed to work against AiiDA’s QueryBuilder tool. However, the UI of said tool regardless should follow a modern-day advanced search pattern to facilitate user experience.

## Conclusions

Designing user interfaces for digital twins involves careful integration and synchronization of both the ELN and the WFMS user interfaces to facilitate data integration into and mutation of digital twins while ensuring the integrity of the overall provenance. The present document provides guidelines for such ventures from the gained insights and experience of the PREMISE project and will self-update as the project continues to apply its findings in practice.
