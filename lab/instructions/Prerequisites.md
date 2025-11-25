# Prerequisites

In this lab, we will be updating a sample Dragon Copilot extension to make use of a deployed Open AI service.

## Before you begin

The lab has the following prerequisites that should be met before starting. If you are using the provided lab environment, these prerequisites should already be met. If you are using your own environment, please ensure you have the following:

### Software
The machine you are using for this lab should have the following prerequisites installed:

* Visual Studio Code
  * C# Language Extension
  * REST Http Extension
* DotNet 9
* Node 22.20.0
* npm 10.9.3
* Visual Studio 2022 Community Edition
* Az CLI
* DevTunnel

### Accounts and Resources
The following resources should be pre-provisioned for you:

* An Azure AI Foundry OpenAI resource running the gpt-5-mini model.
* Dragon Copilot Lab environment with pre-configured access to Dragon Copilot.

### Tokens
Tokens are used through the lab to simplify configuration and setup. The following tokens will need to be replaced if you are not using the provided lab environment.

| Token Name                      | Sample Value                         |
|---------------------------------|--------------------------------------|
| @lab.LabInstance.Id             | Instance123                          |
| @lab.CloudSubscription.TenantId | 12345678-90ab-cdef-1234-567890abcdef |
| @lab.CloudPortal.Link           | https://portal.azure.com/            |
