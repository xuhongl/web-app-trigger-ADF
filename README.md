# web-app-trigger-ADF

        public async Task mainsimilar()
        {
            // Set variables
            string tenantID = "72f988bf-86f1-41af-91ab-2d7cd011db47";
            string applicationId = "b1a11100-26a6-4fab-b1c0-7aaaf62226a0";
            string authenticationKey = " ";
            string subscriptionId = "4a9ef912-a2e9-4795-b0ce-55b456fd2f6e";
            string resourceGroup = "myADF";
            string region = "eastus";
            string dataFactoryName =
                "myadf-xuhliu";
            string storageAccount = "adfsaxuhliu";
            string storageKey = " ";
            // specify the container and input folder from which all files 
            // need to be copied to the output folder. 
            string inputBlobPath =
                "adftutorial/input";
            //specify the contains and output folder where the files are copied
            string outputBlobPath =
                "adftutorial/output";

            // name of the Azure Storage linked service, blob dataset, and the pipeline
            string storageLinkedServiceName = "AzureStorageLinkedService";
            string blobDatasetName = "BlobDataset";
            string pipelineName = "Adfv2QuickStartPipeline";

            // Authenticate and create a data factory management client
            IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(applicationId)
             .WithAuthority("https://login.microsoftonline.com/" + tenantID)
             .WithClientSecret(authenticationKey)
             .WithLegacyCacheCompatibility(false)
             .WithCacheOptions(CacheOptions.EnableSharedCacheOptions)
             .Build();

            AuthenticationResult result = await app.AcquireTokenForClient(
              new string[] { "https://management.azure.com//.default" })
               .ExecuteAsync();
            ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
            var client = new DataFactoryManagementClient(cred)
            {
                SubscriptionId = subscriptionId
            };

            // Create a pipeline run
            Console.WriteLine("Creating pipeline run...");
            Dictionary<string, object> parameters = new Dictionary<string, object>
{
    { "inputPath", inputBlobPath },
    { "outputPath", outputBlobPath }
};
            CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(
                resourceGroup, dataFactoryName, pipelineName, parameters: parameters
            ).Result.Body;
            Console.WriteLine("Pipeline run ID: " + runResponse.RunId);

            // Monitor the pipeline run
            Console.WriteLine("Checking pipeline run status...");
            PipelineRun pipelineRun;
            while (true)
            {
                pipelineRun = client.PipelineRuns.Get(
                    resourceGroup, dataFactoryName, runResponse.RunId);
                Console.WriteLine("Status: " + pipelineRun.Status);
                if (pipelineRun.Status == "InProgress" || pipelineRun.Status == "Queued")
                    System.Threading.Thread.Sleep(15000);
                else
                    break;
            }

            // Check the copy activity run details
            Console.WriteLine("Checking copy activity run details...");

            RunFilterParameters filterParams = new RunFilterParameters(
                DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10));
            ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
                resourceGroup, dataFactoryName, runResponse.RunId, filterParams);
            if (pipelineRun.Status == "Succeeded")
                Console.WriteLine(queryResponse.Value.First().Output);
            else
                Console.WriteLine(queryResponse.Value.First().Error);
            Console.WriteLine("\nPress any key to exit...");
            Console.ReadKey();
        }
    }
