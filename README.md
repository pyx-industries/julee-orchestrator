# Julee Orchestrator

This is a component of the julee architecture
that manages the composition of the other services.

The key concepts are "Pipeline" and "Analysis".

A julee Pipeline processes an event through a series of 4 generic stages:

1. Capture - inbound resource (data) is received and processed, resulting in it being stored and analysed.
2. Extract - various peices of information are extracted from the captured resource.
3. Assemble - the extracted information is assembled into a new artefact
4. Publish - one or more things are done with the new assembled artefact

It does these by orchestrating interactions between the other julee components

## Action Service
The orchestrator ensures that the action service is configured receive inbound resources, as part of the capture process. Alternatively, it may configure the cation service to "go and get" resources on a schedule, etc. The action service might support multiple protocols for recieving resources. Regardless of the protocol, if its a capture protocol, then the last step will be for the action service to store the resource in the appropriate collection (within a Knowlege Service).

The orchestrator also ensures that the action service is configured to perform the publishing actions. These action protocols start with the assembler pushing their new artefact to the action service, which will then execute the appropriate publishing protocols.

A combination of capture and publish protocols, configured in the action service by the orchestrator, constitutes the "julee pipeline integration surface". It's how system integrators can incorporate julee into an enterprise.

The action service is not involved in dynamic analysis, only pipeline interactions with the outside world.


## Knowledge Service

The orchestrator maintains the configuration of collections in the knowledge service, to support both pipelines and analysis.

For pipelines:
- the orchestrator manages the knowledge service configuration so that newly captured resources can be stored in the appropriate place
- the orchestrator routes resource-level queries to collections, to implement the pipeline extraction steps. In other words, "information extraction" is the result of a query applied to a resource.

And for analysis, the orchestrator routes collectiion-level queries to the knowledge service. It does slightly more than that though, it interprets the collection metadata to determine the query semantics that are possible for the collection, and uses this to help applications form valid queries.

# Policy Service

For pipeline processes, the orchestrator creates the relevant assemblies (there may be multiple) once the knowledge service has finished processing the resource. It first retrieves the appropriate collection of extractions from the knowledge service, then it uses a template to form the "raw" assembly, then it sends the raw assembly to zero or more Policy Service (in a deterministic, sequential order) to create a "final" assembly.

The Policy Service performs two kinds of work on the raw assembly:
- in-place transformations
- scorecard appendages

The in-place transformations might apply a gramatical/spelling policy (fix typos, use British spelling, etc), or apply an editorial voice, etc. The scorecard appendages might rate aspects of language tonality (sarcasm, positivity, etc) or flag violations (ribaldry, agression, abusiveness). Transformation policies are essentially for the purpose of refining assemblies to increase their quality.

The scorecards (list of metrics) apply to a specific version of the assembly, and may be different after the application of a transformative policy. So the same scorecards may be assessed multiple times in the policy service.

For pipeline processes, the orchestrator applies the policy services (per configuration) to the raw assembly until the final assembly is achieved, and then passes the final assembly to the action service for publishing protocols. Note, the orchestrator may have conditional rules to implement guard-rails or routing to different publishing protocols, based on scorecard values.

For analytic processes, the orchestrator may also apply rules based on scorecard values. This way, the policy service can be configured to implement an "AI Safety Strategy" through a series of guard-rails policies that assess scorecard values.