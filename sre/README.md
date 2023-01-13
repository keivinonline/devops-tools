# sre 
- keypoints from the google sre books

# sre-book
- https://sre.google/sre-book/table-of-contents/
# 1. Introduction
## SRE
- reliability is the most fundamental property of any product/system
- SRE is a broad discipline 
- SRE can be defined as when you hire software engineers to design an operations team
- SRE teams are focused on constant engineering
- Google places 50% cap upper bound on "ops" for al SREs (i.e. tickets, on-call, manual tasks etc)
- want systems that are automatic, not just automated
- DevOps can be viewed as a generalization of several core SRE principles to a wider range of organizations
- One could view SRE as a specific implementation of DevOps
## Error Budgets` 
    - 100% is the wrong reliability target for anything as User can't tell 100% and 99.999% 
- more of a product question than an engineering question
- questions to ask
    - what level of availability is acceptable for your product?
    - what alternatives are there when users are dissatisfied with the service?
    - what happens to users' usage of product at different availability levels? 
- error budget can be spent on taking risks on new features or improving reliability
- phase rollouts and 1% experiments
- the error budget can be spent to get maximum feature velocity 
- outages are no longer a "bad" thing and it is an expected part of a process of innovation 

## Monitoring
- if a monitoring system requires a human to read an email and decide what to do, then it is flawed 
### Valid monitoring output
1. Alerts - signifies need for action
2. tickets - signifies need for action but not immediately
3. logging - recorded for diagnostic and forensic purposes. expectation is that no one reads logs unless required
## Emergency Response
- reliability is a function of `MTTF(mean time to failure)` and `MTTR(mean time to recovery)`
- on-call playbooks and "wheel of misfortune" exercises help to prepare for emergencies and improve MTTR
## Change management
- implement progressive rollouts
- rollback changes safely and quickly
# Production environment at Google, from SRE point of view
- google uses `borg` a cluster manager 
# sre workbook
- https://sre.google/workbook/table-of-contents/

