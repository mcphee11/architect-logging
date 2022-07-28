# architect-logging

This is an example on how to get debugging information from architect flows including digital flows more accessible in realtime to assist in debugging when building out flows. There are several ways to do this, this is what I personally find a quick handy way to do this task. 

Basically this method will gather the error messages that you want to see and sends them into a Genesys Cloud Chat group via a WebHook API request. One of the advantages of this approach is that you can control who has access to the chat group as well as this can receive error messages even when your not online and you can come back and see them with time stamps.

![](/docs/images/loggingExample1.png?raw=true)

# Create a Group

First you will need a "Group" in this case we are going to create a new one and only assign it to our own user. Ensure that when you create the group you se the visability to "Members Only" that way other users will not be shown the chat group in their UI.

![](/docs/images/createGroup.png?raw=true)

Once you create the group you will be assigned as a member by default

# Install the Generic Chat Notification Integration

There are 2x integrations you will need to install. The first one will be the "Generic Chat Notifications"

![](/docs/images/webhookIntegration.png?raw=true)

Once installed you will be given a "WebHook Notification" URL copy this value it will be something like: 

    https://apps.mypurecloud.com.au:443/webhooks/api/v1/webhook/<YOUR_WEBHOOK_ID>

Go to the "Mappings" Tab create a new mapping, select the group that was created above and ensure that the starts with is "logging". This is also where you can create different mappings to send different messages to different groups if you would like. You can also use the MetaData value in future but at first lets start simple with just the one mapping.

![](/docs/images/mapping.png?raw=true)

Save the changes and ensure that the Integration is "Active".

# Install the Generic Web Services Integration

![](/docs/images/webServicesIntegration.png?raw=true)

Once you have installed this integration you can import the dataAction below:

[Logging to GC Chat Group](/docs/dataAction/Logging-to-GC-Chat-Group.json)

You will need to replace the URL with your own from the above step where you are given the WebHook URL. Now save and Publish the Action as well as ensuring that the Integration is "Active". Now all the pre setup is done we can get to using this new DataAction in the Architect flow.

# Using the logger

Inside Architect if you get to a place where you want to log any errors for example if a DataAction calls a 3rd party you want to know what happens and why if it goes down the "Failed" path. To do this first of all click on the dataAction block and assign a variable to the userMsg output:

    State.varErrorMsg

![](/docs/images/varErrorMsg.png?raw=true)

You can no put the logging dataAction in the failed path of the original dataAction as well as pass in the State.varErrorMsg into the string being sent to the chat group. Personally I also add in the block number so if i have several items logging I can easily see at which point this error happened.

![](/docs/images/sendMessage.png?raw=true)

Now if an interaction goes through the Failed path in that dataAction it will be sent to our Logging Chat room as we can see it in realtime appear.

# Considerations

It is worth noting that each request does add slight latency to the overall conversation so I would not be adding these logging blocks EVERYWHERE especially in production, but having them in the failed paths is a very nice way to capture items even when your not actively monitoring the flow as the chat room will keep the history even if it happens at 12am on Sunday.

I also often use this to see the value of a variable at a point in time of the flow. Do be aware though that you are converting the output as a "String" and often when i see people run into issues its due to a int being used not a String etc.