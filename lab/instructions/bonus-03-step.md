## Local Integration Testing

We will now verify the contract along with the adaptive card output from the extension

1. Go back to your terminal and navigate to `samples\DragonCopilot\Workflow\SampleExtension.Web` 
2. Issue a +++dotnet run+++ command.
3. Open the http file `samples\DragonCopilot\Workflow\SampleExtension.Web\SampleExtension.Web.http`
4. Click `Send Request`
5. In the output, find the property `adaptive_card`
6. Copy the value of the property including the braces.
7. Open the [Microsoft Adaptive Card Designer](https://adaptivecards.microsoft.com/designer)
8. In the "Copy Payload Editor" section, replace the content with your copied content.
9. Verify that the rendered adaptive card looks correct.

You have now seen the output of your extension. If you would like to make any tweaks to what you are seeing, now is the time to do that. In the next section we setup a DevTunnel to make your extension accessible to Dragon Copilot.
