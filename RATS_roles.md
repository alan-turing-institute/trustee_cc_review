(sec-rats)=
# RATS Roles

:::{glossary}
Attester
: The party (usually a machine) that that is being tested to prove it is trustworthy.
  The attester provides evidence to be assessed to the {term}`verifier`.

Endorser
: A party that is consulted by the {term}`verifier` to assess pieces of evidence from the {term}`attester`.
  Hardware manufacturers are an important class of endorser, holding secrets needed to verify the unique keys of genuine hardware and vouch for the hardware's capabilities.

Reference Value Provider
: A party which provides known good values to the {term}`verifier` to assist in the assessment of the {term}`attester's <attester>` evidence.
  Reference values are compared to measurements in attestation evidence.

Relying Party
: The party (perhaps a piece of software) which relies on the {term}`verifier's <verifier>` report in order to make a decision, such as whether to use a TEE for a workflow.

Relying Party Owner
: The party (for example an organisation, or system administrator) which has the authority to control how the {term}`relying party` responds to an attestation report.

Verifier
: The party (or service) which assesses the evidence produced by the attester.
  The verifier may consult {term}`endorsers <endorser>` and {term}`reference value providers <reference value provider>` for support in assessing pieces of evidence.
  The verifier does not itself decide whether the attester is trustworthy or not but instead produces a report.

Verifier Owner
: The party (for example an organisation, or system administrator) which is able to set how the {term}`verifier` evaluates the evidence it receives from the {term}`attestor`.
:::
