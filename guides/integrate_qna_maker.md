# Azure Bot Framework - QnA Maker Bot Integration

### This guide will help you integrate an existing QnA Maker knowledge base into your bot.

When you've completed this tutorial, you should expect to see this:
<br/><img src="../screens/embedded_web_chat_qna.jpg" /><br/><br/>


### Section 1: Modify the application settings to hold our QnA Maker knowledge base info

Within your project, you'll see a [`appsettings.json`](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-2.2#default-configuration) file. This file is used to store key/value configuration data needed by your application. This data is typically pulled in at runtime and used within your app. The `appsettings.json` file is for storing your app configuration data when locally hosted or remotely hosted if the file is deployed. Depending on the sensitivity of the data, you may want to ignore this file when commiting code to a public repository.

When your app is running in a hosted scenario, such as in Azure, you can override the values of the key/value pairs under the __Configuration__ section in your App Service in the [Azure Portal](https://portal.azure.com). This allows you to use different instances of services for different environments (i.e. tracking exceptions for staging using a different instance of Application Insights) 

Because we have a few bits of information tied to our knowledge base, we need to add this to our appsettings.json file 

1. Open the `appsettings.json` file and add the following code within the `settings` node after the `noAnswerMessage` node, then replace with your knowledge base ID, endpoint key, host URL from the previous step and then save
	```
	"qnaMaker": {
		"endpointKey": "<YOUR_QNAMAKER_ENDPOINT_KEY>",
		"knowledgeBaseId": "<YOUR_QNAMAKER_KB_ID>",
		"hostname": "<YOUR_QNAMAKER_HOST_URL>"
	},
	```

1. At this point, your `appsettings.json` file should look like this:
	```
	{
		"settings" : {
			"noAnswerMessage": "I was unable to find an answer to your question.",
			"qnaMaker": {
				"endpointKey": "7871390f-735d-422b-8fdb-680a186788dc",
				"knowledgeBaseId": "3c021717-06c7-43f7-a2ac-42f25e29853d",
				"hostname": "https://mybotqnamaker.azurewebsites.net/qnamaker"
			},
			"welcomeCard": {
				"title": "Meet Eureka!",
				"description": "Eureka is the Secretary of State’s new online search assistant. Powered by Microsoft’s artificial intelligence, Eureka knows the answers to frequently asked Business Programs Division questions.",
				"videoUrl": "https://youtu.be/YxUPIu7PL14",
				"learnMoreUrl": "https://aka.ms/EurekaBot"
			}
		}
	}
	```

1. Add the following class into the namespace in the `Settings.cs` file:
	```
	public class QnAMakerServiceSettings
	{
		public string EndpointKey { get; set; }
		public string Hostname { get; set; }
		public string KnowledgeBaseId { get; set; }
	}
	```

	This class will be used to store the QnA Maker configuration data to be used at runtime

1. Create a new property inside the `Settings` class to store our `QnAMakerServiceSettings` data
	```
	public QnAMakerServiceSettings QnAMaker { get; set; }
	```

<br/>

### Section 2: Integrate the QnAMaker .NET Core SDK

1. QnA Maker supports REST API but we'll take advantage of the SDK support by adding the following nuget package:
	- if you are using VS Code, you can add nuget packages using the terminal (Terminal > New Terminal)
		```
		dotnet add package Microsoft.Bot.Builder.AI.QnA -v 4.2.0
		```
	- if you are using Visual Studio 2017, right click on your project and select __Manage NuGet Packages...__, click the __Browse__ tab and search for:
		```
		Microsoft.Bot.Builder.AI.QnA
		```
		You should get back one result - click on it and select version 4.2.0, then click the __Install__ button

1. Open the `Bot.cs` file and create a private static variable within the `EurekaBot` class for our `QnAMaker` object
	```
	static QnAMaker _qnaMakerService;
	```

1. Add a `using` statement at the top for the QnA Maker namespace:
	```
	using Microsoft.Bot.Builder.AI.QnA;
	```

1. Add a method to create the value for `_qnaMakerService` if not already assigned
	```
	void EnsureQnAMakerService()
	{
		if(_qnaMakerService != null)
			return;

		var qnaEndpoint = new QnAMakerEndpoint()
		{
			KnowledgeBaseId = _settings.QnAMaker.KnowledgeBaseId,
			EndpointKey = _settings.QnAMaker.EndpointKey,
			Host = _settings.QnAMaker.Hostname,
		};

		_qnaMakerService = new QnAMaker(qnaEndpoint);
	}
	```

1. Add a method to execute the answer look-up in the knowledge base
	```
	async Task LookupAnswerInKnowledgeBase(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
	{
		if (string.IsNullOrEmpty(turnContext.Activity.Text))
			return;

		//Make sure we have a valid QnAMaker service to use
		EnsureQnAMakerService();

		//Get any possible answers to the question typed
		var results = await _qnaMakerService.GetAnswersAsync(turnContext);
		if (results != null && results.Any())
		{
			//Return the first result (you could also ensure the result.Score is of a minimum threshold)
			await turnContext.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
		}
		else
		{
			await turnContext.SendActivityAsync(_settings.NoAnswerMessage, cancellationToken: cancellationToken);
		}
	}	
	```

1. Now let's make the call to look up possible answers in the knowledge base by replacing the code within the `ActivityTypes.Message` switch/case statement in `OnTurnAsync` with the following and save:
	```
	await LookupAnswerInKnowledgeBase(turnContext, cancellationToken);
	break;
	```

1. That should do it, time to test it out by hitting `F5` to start the local debugger

1. Launch the Bot Framework Emulator and ensure __development__ is selected as the endpoint

1. Type a question phrased similar to a question in your knowledge base and confirm the appropriate answer was send back by the bot
<br/><img src="../screens/bot_framework_emulator_qna_local.jpg" />


### Section 3: Deploy to Azure

1. Commit your changes to git and push to your remote repository to kick off a new build and deploy
	- you can confirm the automated pipeline is working by visiting the __Deployment Center__ section of the App Service hosting your bot in the Azure Portal

1. Test your bot in the web chat and the Bot Framework Emulator to ensure your public endpoint is functioning properly
<br/><img src="../screens/bot_framework_emulator_qna_production.jpg" />

<br/>

### Section 4: Add Chat Bot to Web Page

Now that we have a working public endpoint, we can add enable one of the several bot channel options available to us, Web Chat by adding a few lines of HTML to our web page.

1. Open the `wwwroot/default.htm` file and add the following code just before the `</body>` close tag:
	```
	<img src="https://cdn0.iconfinder.com/data/icons/social-messaging-ui-color-shapes/128/chat-circle-blue-512.png"
	style="width: 60px; height: 60px; position: absolute; bottom: 30px; right: 30px; cursor: pointer"
	onclick="(function(){document.getElementById('chatFrame').style.visibility='visible'})()" />

	<iframe id="chatFrame" src='<YOUR_BOT_EMBED_CODE_SRC>'
		style='visibility: hidden; width: 500px; height:600px; position: absolute; bottom: 20px; right: 20px; background-color: #FFF; border-width: 1px'></iframe>

	```

	> Note - embedding the secret in javascript makes it easy for other developers to embed your bot in their pages. In the future, as a best practice, you should [exchange the secret for a time-based token](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0#option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed). It's a little bit more work but much more secure.

1. To get the URL that the `iframe` needs to point to, navigate to your Web App Bot in the Azure Portal

1. Click on the __Channels__ section

1. Click on the __Edit__ link under Web Chat
<br/><img src="../screens/web_chat_channel_settings.jpg" />

1. Here you'll find 2 secrets and HTML embed code - copy the value of the `src` property of the `iframe` and paste it over `<YOUR_BOT_EMBED_CODE_SRC>`

1. Back in the Azure Portal, click the __Show__ link next to one of the secret keys to expose the value, copy it and paste it over the `YOUR_SECRET_HERE` so the value of the `iframe`'s `src` property in your code looks similar to this:
	```
	https://webchat.botframework.com/embed/EurekaChatBot?s=gKVaFfVTvks.cwA.8cc.YX7eSxJDJKDfgWsqMlOEh25Jl7EOSKWKQIn-Imth9XSo
	```

1. Save the file and hit `F5` to launch the local debugger

1. In a web browser, navigate to `http://localhost:3978` and click on the chat icon in the lower right corner

1. Ask a question and verify the appropriate answer is returned by the bot
<br/><img src="../screens/embedded_web_chat_qna.jpg" />

1. Commit and push your code to your repository to trigger a build and deploy in Azure, then re-test against your public endpoint
