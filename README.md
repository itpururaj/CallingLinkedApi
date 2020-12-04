# Calling Linked Api
How to call one api from another api


private async Task<List<FxRates>> GetDataFromLinkedApi(string parameters)
        {
            var externalData = new List<MyDataModel>();
            var baseUrl = _configuration["ExternalApiBaseUrl"];
            var externalMethodUrl = _configuration["ExternalMethodUrl"];
  
            // AzureServiceTokenProvider will help us to get the Service Managed token.
            var azureServiceTokenProvider = new AzureServiceTokenProvider();
            
            // Authenticate to the Azure Resource Manager to get the Service Managed token.
            string accessToken = await azureServiceTokenProvider.GetAccessTokenAsync("https://management.azure.com/");
            
            // Setup an HTTP Client and add the access token.
            HttpClient client = new HttpClient() { BaseAddress = new Uri(baseUrl) };
            client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
            client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            
            // HTTP GET
            _logger.LogDebug($"Calling internal/external api to get the data: {externalMethodUrl}?{parameters}");
            HttpResponseMessage response = client.GetAsync($"{externalMethodUrl}?{parameters}").Result;
            if (!response.IsSuccessStatusCode)
            {
                var errorMessage = $"Error occured while fetching data from {response.RequestMessage.RequestUri.AbsoluteUri}. Status Code: {response.StatusCode}";
                _logger.LogError(new Exception(errorMessage), errorMessage);
                throw new Exception(errorMessage);
            }
            var dataSourceResult = await response.Content.ReadAsAsync<List<MyDataSourceModel>>();
            dataSourceResult?.ForEach(data =>
            {
                var formattedDate = DateTime.ParseExact(data.Date, "yyyyMMdd", CultureInfo.InvariantCulture);
                externalData.Add((new MyDataModel()
                {
                    Property1 = data.property1,
                    Property2 = data.property2,
                    Property3 = data.property3,
                }));
            });
            return externalData;
        }


Application Configuration:

"ExternalApiBaseUrl": "https://webapp-dev01.az-ase-ms-gen01.appserviceenvironment.net/",
"ExternalMethodUrl": "api/MyExternalMethod"
