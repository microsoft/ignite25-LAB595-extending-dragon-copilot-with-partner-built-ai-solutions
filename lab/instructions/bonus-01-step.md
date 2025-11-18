## Updating the Processor

The next step is to update the processor to make use of an OpenAI service. We are going to be updating the sample to make a call to the OpenAI service to generate a patient friendly explanation of the note. Our work will be split into two different steps:
* Processing of the note
* Generating the adaptive card output

### Updating the processing of the note

#### Adding required dependencies to our solution
1. In VS Code, open the terminal and navigate to `samples\DragonCopilot\Workflow\SampleExtension.Web`
2. Issue a `dotnet add package Azure.AI.OpenAI --prerelease`
3. Issue a `dotnet add package Azure.Core`

#### Updating the processing service

1. In VS Code, open the following file and replace with the following code.
    ```
    samples\DragonCopilot\Workflow\SampleExtension.Web\Services\ProcessingService.cs

    ```
```C#

    // Copyright (c) Microsoft Corporation.
    // Licensed under the MIT License.

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Text;
    using Microsoft.Extensions.Logging;
    using Dragon.Copilot.Models;
    using SampleExtension.Web.Extensions;
    using Azure;
    using Azure.AI.OpenAI;
    using Azure.Identity;
    using OpenAI.Chat;

    using static System.Environment;
    using System.Text.Json;

    namespace SampleExtension.Web.Services;

    /// <summary>
    /// Implementation of the processing service for Dragon Standard payloads
    /// </summary>
    public class ProcessingService : IProcessingService
    {
        private readonly ILogger<ProcessingService> _logger;

        const string endpoint = "https://ignite25aiaiservices-@lab.LabIntance.Id.openai.azure.com/";
        const string key = "<< ENTER API KEY HERE>>";

        /// <summary>
        /// Constructor for the processing service
        /// </summary>
        /// <param name="logger">The logger</param>
        public ProcessingService(ILogger<ProcessingService> logger)
        {
            _logger = logger;
        }

        /// <inheritdoc />
        public async Task<ProcessResponse> ProcessAsync(
            DragonStandardPayload payload,
            string? requestId = null,
            string? correlationId = null,
            CancellationToken cancellationToken = default)
        {
            ArgumentNullException.ThrowIfNull(payload, nameof(payload));

            try
            {
                _logger.LogProcessingStart(requestId, correlationId);

                var processResponse = new ProcessResponse();

                // Process Note if present
                if (payload.Note != null)
                {
                    var noteResponse = await ProcessNoteAsync(payload.Note, payload.SessionData, cancellationToken).ConfigureAwait(false);

                    processResponse.Payload["adaptive-card"] = noteResponse;
                }

                // TODO: Add processing for other payload types (Transcript, IterativeTranscript, IterativeAudio)

                processResponse.Success = true;
                processResponse.Message = "Payload processed successfully";

                return processResponse;
            }
    #pragma warning disable CA1031
            catch (Exception ex)
    #pragma warning restore CA1031
            {
                _logger.LogProcessingException(ex, requestId);

                return new ProcessResponse
                {
                    Success = false,
                    Message = "An error occurred while processing the payload",
                };
            }
        }

        private static async Task<DspResponse> ProcessNoteAsync(
            Note note,
            SessionData sessionData,
            CancellationToken cancellationToken)
        {
            AzureKeyCredential credential = new AzureKeyCredential(key);

            // Initialize the AzureOpenAIClient
            var azureClient = new AzureOpenAIClient(new Uri(endpoint), credential);

            // Initialize the ChatClient with the specified deployment name
            ChatClient chatClient = azureClient.GetChatClient("gpt-5-mini");

            StringBuilder promptBuilder = new StringBuilder();
            foreach (var section in note.Resources ?? Array.Empty<ClinicalDocumentSection>())
            {
                if (string.IsNullOrWhiteSpace(section.Content))
                    continue;

                promptBuilder.AppendLine(section.Context?.Definition);
                promptBuilder.AppendLine(section.Content);
                promptBuilder.AppendLine("---");
            }
            
            // Create a list of chat messages
            var messages = new List<ChatMessage>
            {
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
            new UserChatMessage(promptBuilder.ToString())
            };

            // Create chat completion options
            var options = new ChatCompletionOptions();
            
            try
            {
                // Create the chat completion request
    #pragma warning disable CA2007 // Consider calling ConfigureAwait on the awaited task
                ChatCompletion response = await chatClient.CompleteChatAsync(messages, options, cancellationToken);
    #pragma warning restore CA2007

                // Print the response
                if (response != null)
                {
    #pragma warning disable CA1869 // Cache and reuse JsonSerializerOptions
                    Console.WriteLine(JsonSerializer.Serialize(response, new JsonSerializerOptions() { WriteIndented = true }));
    #pragma warning restore CA1869
                }
                else
                {
    #pragma warning disable CA1303 // Do not pass literals as localized parameters
                    Console.WriteLine("No response received.");
    #pragma warning restore CA1303
                }

                // Create adaptive card version
                var adaptiveCardResponse = new DspResponse
                {
                    SchemaVersion = "0.1",
                    Document = note.Document,
                };

                var responseText = response?.Content.FirstOrDefault()?.Text ?? "No response content";
                adaptiveCardResponse.Resources.Add(CreateAdaptiveCardResource(responseText));
                return adaptiveCardResponse;
            }
            catch (Exception ex)
            {
                Console.WriteLine($"An error occurred: {ex.Message}");
                throw;
            }
        }

    private static VisualizationResource CreateAdaptiveCardResource(string summary)
    {
        // Create the body elements list
        var bodyElements = new List<object>
        {
            // Header
            new
            {
                type = "TextBlock",
                text = "Patient Friendly Summarization - John Doe",
                weight = "bolder"
            },

        };

        // No entities found message
        bodyElements.Add(new
        {
            type = "Container",
            items = new object[]
            {
                new
                {
                    type = "TextBlock",
                    text = summary,
                    wrap = true
                }
            }
        });

        // Add footer
        bodyElements.Add(new
        {
            type = "TextBlock",
            text = $"Processed at {DateTime.UtcNow:yyyy-MM-dd HH:mm:ss} UTC",
            spacing = "medium"
        });

        return new VisualizationResource
        {
            Id = Guid.NewGuid().ToString(),
            Type = "AdaptiveCard",
            Subtype = VisualizationSubtype.Timeline,
            CardTitle = "Clinical Entities Extracted",
            PartnerLogo = "https://example.com/assets/sample-extension-logo.png",
            AdaptiveCardPayload = new AdaptiveCardPayload
            {
                Body = bodyElements
            },
            Actions = new List<VisualizationAction>
            {
                new()
                {
                    Title = "Accept Analysis",
                    Action = VisualizationActionType.Accept,
                    ActionType = ActionButtonType.Accept,
                    Code = "Accept"
                },
                new()
                {
                    Title = "Copy to Note",
                    Action = VisualizationActionType.Copy,
                    ActionType = ActionButtonType.Copy,
                    Code = summary
                },
                new()
                {
                    Title = "Reject Analysis",
                    Action = VisualizationActionType.Reject,
                    ActionType = ActionButtonType.Reject,
                    Code = "Reject"
                }
            },
            References = new List<VisualizationReference>(),
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
  }

```

2. Copy Key using following steps:
    1. Open `https://ai.azure.com/?tid=4cfe372a-37a4-44f8-91b2-5faf34253c62`
    2. Click the AI resource
    3. Copy API Key

3. Paste the key into line
```C#
const string key = "<< ENTER API KEY HERE>>";
```
4. Execute `dotnet run`
5. Repeat the recording to observe clinical entities extracted by AI in the Dragon Copilot Timeline.