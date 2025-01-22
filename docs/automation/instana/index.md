- [Instana Installation on OCP](https://pages.github.ibm.com/DACH-TECH-AUTOMATION/dach-automation-waiops-tech-mini-jam/chapters/instana_cpwaiops_onocp/instana/)
- [step-by-step](https://github.ibm.com/Lai-Mee-Lok/OCP-step-by-step-Instana-Backend/tree/power)

# What you need
- `<agent key>`
- `<sales key>`

# BeeInstana Operator
BeeInstana can be installed as an OC Operator.

!!! warning
    Installation of BeeInstana is optional at installation time and it can be added later to an existing installation. However, adding BeeInstana to an existing Instana installation will result in a database migration to BeeInstana. The duration of this migration is dependent on the amount of already existing data.

# Install Instana

One needs `agent_key` to be able to access Helm repo. To install Instana properly, these are the steps you need to do:
1. install `instana-kubectl` plugin
2. using the plugin that installed in step 1, install Instana Operator
3. create Instana License File
4. create `secret` and collect them
5. create **CORE** config file and create `secret` out of it
6. create **UNIT** config file and create `secret` out of it
7. create routes for both **core** and **unit**

# Questions

1. what does beeinstana do? The data store operators, are they are mandatary to install
