## Updating the Processor

The next step is to update the processor to make use of an OpenAI service. We are going to be updating the sample to make a call to the OpenAI service to generate a patient friendly explanation of the note. Our work will be split into two different steps:
* Processing of the note
* Generating the adaptive card output

### Updating the processing of the note

#### Obtaining the OpenAI sample code
An OpenAI resource running the gpt-5-mini model has been pre-provisioned for you.

1. In a browser, open the Azure AI Foundrary +++https://ai.azure.com/+++
2. Click "Sign In" and use credentials found in the "Resources" tab above.
3. In the top table, click the resource that starts with "ignite25ai"
5. In the left navigation select `Models + Endpoints`
7. In the table select `gpt-5-mini`
8. In the right side of the screen update the language to `C#`

#### Adding required dependencies to our solution
1. In VS Code, open the terminal and navigate to `samples\DragonCopilot\Workflow\SampleExtension.Web`
2. Issue a `dotnet restore`
3. Issue a `dotnet add package Azure.AI.OpenAI --version 2.2.0-beta.4`
4. Issue a `dotnet add package Azure.Core`

#### Updating the processing service

1. In VS Code, open the following file
    ```
    samples\DragonCopilot\Workflow\SampleExtension.Web\Services\ProcessingService.cs

    ```
1. Navigate to the `ProcessNoteAsync` method.
1. Remove the contents of the method.
1. Update the method signature to the following
	```csharp
	private static async Task<DspResponse> ProcessNoteAsync(
        Note note,
        SessionData sessionData,
        CancellationToken cancellationToken)

	```
1. Back in your browser, in the Open AI Deployment "Get Started" area, scroll to Section 3 "Run a basic code sample"
1. Click the "copy" icon in the top right of the box.
1. Paste the sample into the `ProcessNoteAsync` method.
1. Move the copied "using" statements to the top of the file.
1. Back in the browser, in the top of the left section, copy the key.
1. Paste the key into the variable `apiKey` in the `ProcessNoteAsync` method.
1. Update the `var response = chatClient.CompleteChat(messages, requestOptions);` as follows
    ```csharp
	var response = await chatClient.CompleteChatAsync(messages, requestOptions, cancellationToken).ConfigureAwait(false);

	```
1. Add the following code to the bottom of the `ProcessNoteAsync` method
	```csharp
	// Create adaptive card version
    var adaptiveCardResponse = new DspResponse
	{
        SchemaVersion = "0.1",
        Document = note.Document,
    };

    return adaptiveCardResponse;
	```
1. Before the `messages` variable is defined, add the following code which concatenates all the note sections to a single variable:
	```csharp
    StringBuilder promptBuilder = new StringBuilder();
    foreach (var section in note.Resources ?? Array.Empty<ClinicalDocumentSection>())
    {
        if (string.IsNullOrWhiteSpace(section.Content))
            continue;

        promptBuilder.AppendLine(section.Context?.Definition);
        promptBuilder.AppendLine(section.Content);
        promptBuilder.AppendLine("---");
    }
	```
1. Add a using statetment at the top of the file for `using System.Text;`
1. Update the `SystemChatMessage` to the following
	```csharp
    new SystemChatMessage(@"You are a compassionate healthcare communication specialist. Your role is to translate complex medical notes into clear, patient-friendly explanations that are easy to understand and accurate. This should be in a document format that can be printed and given to the patient when leaving the office.

	GUIDELINES:
    1. Use simple, everyday language - avoid medical jargon
    2. Explain medical terms when you must use them
    3. Structure information logically (what happened → what it means → what's next)
    4. Be warm and empathetic in tone
    5. Focus on what matters most to the patient
    6. Include reassuring context when appropriate
    7. Highlight important actions or follow-ups
    8. Use clear headings to organize information

	FORMAT YOUR RESPONSE WITH:
	• **What We Found:** Brief summary of key findings
	• **What This Means:** Explanation in simple terms
	• **Important Points:** Key things to know or remember
	• **Next Steps:** Any follow-up actions or appointments

	EXAMPLE STYLE:
	Instead of: 'Patient presents with acute upper respiratory infection'
	Say: '**What We Found:** You have a cold or upper respiratory infection - this means the virus has affected your nose, throat, and upper breathing passages.'

	Remember: Patients want to understand their health, feel informed, and know what to expect. Be thorough but not overwhelming."),

	```
1. Update the `UserChatMessage` to the following
	```csharp
    new UserChatMessage(promptBuilder.ToString()),

    ```
1. Navigate to the `ProcessAsync` method.
2. Remove the line
	```csharp
    processResponse.Payload["sample-entities"] = noteResponse.SampleEntities;

    ```
3. Update the following line to read:
	```
	processResponse.Payload["adaptive-card"] = noteResponse;

    ```

We should now be able to successfully call the OpenAI Resource. In the next section, we will update the output to display an adaptive card
