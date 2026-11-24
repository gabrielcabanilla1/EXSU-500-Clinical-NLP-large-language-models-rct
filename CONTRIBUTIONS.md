# Contributions and Use of Generative AI

## Accountable roles

Data lead: Nyah Woller-Li. Writing lead: Neda Oghbaei. Interface lead: Gabriel Cabanilla. Modelling is
shared, with each member accountable for one model family: Nyah Woller-Li (position baseline), Neda Oghbaei
(logistic regression) and Gabriel Cabanilla (PubMedBERT and shared evaluation). Every member contributed to
the code and to the writing; the roles establish who is answerable for each area.

## CRediT author statement

| Member | Roles (CRediT taxonomy) |
|---|---|
| Gabriel Cabanilla | Conceptualization (choice of dataset); Software (03c PubMedBERT; shared data preparation and evaluation code; 01, 04 and 05); Validation (review and standardisation of 03a and 03b); Writing – review & editing |
| Neda Oghbaei | Investigation (literature review); Software (03b logistic regression); Writing – original draft; Writing – review & editing |
| Nyah Woller-Li | Project administration (team coordination); Software (03a position baseline); Writing – review & editing |

All three members prepared and recorded the presentation video.

## Use of Generative AI statement

Claude (Anthropic) was the only generative AI tool used in this project. It was used to brainstorm ideas, 
to guide the coding process for the model notebooks, to help identify relevant literature, and to correct 
grammar in the manuscript. Every source it suggested was located and checked against the original publication 
before being cited. All code was written or reviewed, run and validated by the team, and the reported results 
come from our own experiments. The study design, analysis, interpretation and final text are the authors’ own, 
and the authors take full responsibility for the content of this paper.

_N. Oghbaei: state your own use of generative AI here, or "N. Oghbaei did not use generative AI."_

Disclosure on the test split: while checking code with the assistant, test-split scores were computed
outside the planned single evaluation (once on the "@" version of the data while verifying a reference
notebook, and once while testing the code of `05_final_test` on a local copy). No model, setting or
analysis was changed after these checks; all choices were made on the validation split.

## Acknowledgements

Because no team member has a computer science background, the team also consulted people outside the team
for advice on the code.
