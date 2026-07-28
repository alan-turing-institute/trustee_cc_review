(sec-rats)=
# RATS Roles

:::{glossary}
attester
: The party (usually a machine) that that is being tested to prove it is trustworthy.
The attester provides evidence to be assessed to the {term}`verifier`.

verifier
: The party (or service) which assesses the evidence produced by the attester.
The verifier may consult {term}`endorsers <endorser>`, delegating the assessment of pieces of evidence.
The verifier does not itself decide whether the attester is trustworthy or not, but instead produces a report.

endorser
: A party (or service) that is consulted by the {term}`verifier` to assess pieces of evidence from the {term}`attester`.
Hardware manufacturers are an important class of endorser, holding secrets needed to verify the unique keys of genuine hardware.

relying party
: The party (perhaps a piece of software) which relies on the {term}`verifiers <verifier>` report in order to make a decision, such as whether to use a TEE for a workflow.

relying party owner
: The party (for example an organisation, or system administrator) which has the authority to control how the {term}`relying party` responds to an attestation report.
:::
