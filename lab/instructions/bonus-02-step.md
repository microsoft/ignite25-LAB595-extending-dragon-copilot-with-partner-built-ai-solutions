## Updating the Processor - Output

We will now update the output format of the sample to display an adaptive card.

1. In VS Code, open the following file
    ```
    samples\DragonCopilot\Workflow\SampleExtension.Web\Services\ProcessingService.cs

    ```
2. Add a new method called `CreateAdaptiveCardResource`
	```csharp
    private static VisualizationResource CreateAdaptiveCardResource(string summary)
    {
        // Create the body elements list
        var bodyElements = new List<object>
        {
            // Header
            new
            {
                type = "TextBlock",
                text = "Patient Friendly Summarization",
                weight = "Bolder",
                size = "Large",
                color = "Accent"
            },
		
        };

        // No entities found message
    	bodyElements.Add(new
        {
			type = "Container",
		    style = "attention",
	        items = new object[]
    		{
		        new
				{
				    type = "TextBlock",
                    text = summary,
                    wrap = true,
                    horizontalAlignment = "Center"
                }
            }
        });

        // Add footer
        bodyElements.Add(new
        {
            type = "TextBlock",
            text = $"Processed at {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC",
            size = "Small",
            horizontalAlignment = "Right",
            spacing = "Medium"
        });

        return new VisualizationResource
        {
            Id = Guid.NewGuid().ToString(),
            Type = "AdaptiveCard",
            Subtype = VisualizationSubtype.Note,
            CardTitle = "Clinical Entities Extracted",
            AdaptiveCardPayload = new
            {
                type = "AdaptiveCard",
                version = "1.3",
                body = bodyElements.ToArray()
            },
            Actions = new List<VisualizationAction>
            {
                new()
                {
                    Title = "Accept Analysis",
                    Action = VisualizationActionType.Accept,
                    ActionType = ActionButtonType.Primary
                },
                new()
                {
                    Title = "Copy to Note",
                    Action = VisualizationActionType.Copy,
                    ActionType = ActionButtonType.Secondary,
                    Code = summary
                },
                new()
                {
                    Title = "Reject Analysis",
                    Action = VisualizationActionType.Reject,
                    ActionType = ActionButtonType.Tertiary
                }
            },
            PayloadSources = new List<PayloadSource>
            {
                new()
                {
                    Identifier = Guid.NewGuid().ToString(),
                    Description = "Sample Extension Clinical Entity Extractor",
                    Url = new Uri("https://localhost/api/process")
                }
            },
            DragonCopilotCopyData = "Clinical entities extracted from note content"
        };
    }
    ```
1. Update the line that includes `text = "Patient Friendly Summarization",` to include your name such as `text = "Patient Friendly Summarization - John Doe",`
1. Navigate to the `ProcessNoteAsync` method
2. Add the following line before the `return adaptiveCardResponse;` statement
	```csharp
    adaptiveCardResponse.Resources.Add(CreateAdaptiveCardResource(response.Value.Content[0].Text));

    ```

We have now updated our processor to display our results into an adaptive card output. In our next section, we will test this locally and verify our adaptive card output.
