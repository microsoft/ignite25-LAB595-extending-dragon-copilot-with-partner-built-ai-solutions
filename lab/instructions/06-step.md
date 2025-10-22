## Packaging your Extension

We are now going to package our extension using the dragon-extension CLI tool. The CLI tool has been pre-registered using the [instructions on the GitHub repository](https://github.com/microsoft/dragon-copilot-extension-samples?tab=readme-ov-file#dragon-extension-cli). 

1. Open a new terminal window
2. Create a new folder called `extension-payload`
3. Change directory to the newly created directory
4. Issue a `dragon-extension init` command
    1. Give your extension a name and include your name in lowercase characters (e.g. `my-extension-@lab.User.FirstName-@lab.User.LastName`)
    2. For Entra Tenant ID you can use: `@lab.CloudSubscription.TenantId`
    3. In the tools section, select "Note" as the input type
    4. For API Endpoint, use the dev tunnel URL you created in the last step with the addition of "v1/process" as the route. (i.e. https://{devtunnel-url}/v1/process)
    5. The rest of the details can be taken as default.
5. Issue a `dragon-extension package` command.

You now have a valid zip file that represents your extension.