## RTM vs Socket

The legacy real time messaging service (RTM) used a websocket, when attached to could send and receive messages and notifications. For better security and a more up to date version its is suggested to use the HTTP version instead.
## App
Use the slack app to setup tokens used for messaging and give permissions to chosen channels for reading and writing.

Needs a re install into workspace after making changes.
## Events API

One way to use the Events API is as an alternative to opening WebSocket connections to the [RTM API](https://api.slack.com/rtm). Why choose the Events API over the RTM API? 

Instead of maintaining one or more long-lived connections for each workspace an application is connected to, you can set up one or more endpoints on your own servers to receive events atomically in near real-time. The events API gives yous more control over handling events, however RTM is easier to set up.

A new socket will connect to the setup slack server used for ingesting messages.