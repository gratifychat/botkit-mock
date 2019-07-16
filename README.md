# Botkit-Mock - Write tests for [Botkit](https://github.com/howdyai/botkit).
[![npm](https://img.shields.io/npm/l/botkit.svg)](https://spdx.org/licenses/MIT)
[![Build Status](https://travis-ci.org/gratifyguy/botkit-mock.svg?branch=master)](https://travis-ci.org/gratifyguy/botkit-mock)


## Setup ##

1. `npm install --save botkit-mock`
2. Require `botkit-mock` in your test: `const { Botmock } = require('botkit-mock');`
3. Require your controller in your test: `const fileBeingTested = require("./indexController")`
4. Follow test case examples seen [here](/examples)

## Basic Usage ##

### Testing Controllers ###

Assuming you have a controller written like below:

```javascript
module.exports = function(controller) {
    // simple answer
    controller.hears(['help'], 'direct_message', function (bot, message) {
        bot.reply(message, 'help message');
    });
}
```

You can write a describe block to test your controller:

```javascript
const { Botmock, SlackApiMock } = require('botkit-mock');
const {SlackAdapter, SlackMessageTypeMiddleware, SlackEventMiddleware} = require('botbuilder-adapter-slack');

const yourController = require("./yourController");

describe("controller tests",()=>{
    beforeEach(()=>{
       const adapter = new SlackAdapter({
            clientSigningSecret: "secret",
            botToken: "token",
            debug: true
        });
   
        adapter.use(new SlackEventMiddleware());
        adapter.use(new SlackMessageTypeMiddleware());
   
        this.controller = new Botmock({
            adapter: adapter,
            disable_webserver: true
        });
   
        SlackApiMock.bindMockApi(this.controller);
        yourController(this.controller);
    });
});
```

In your `it` statement, use the `bot.usersInput` method to define the conversation.

```javascript
it('should return `help message` if user types `help`', async () => {
    const message = await this.bot.usersInput(
        [
            {
                user: 'someUserId',
                channel: 'someChannel',
                messages: [
                    {
                        text: 'help', isAssertion: true
                    }
                ]
            }
        ]
    );

    /* example of botkit response
    { type: 'message',
      text: 'help message',
      attachmentLayout: undefined,
      attachments: undefined,
      suggestedActions: undefined,
      speak: undefined,
      inputHint: undefined,
      summary: undefined,
      textFormat: undefined,
      importance: undefined,
      deliveryMode: undefined,
      expiration: undefined,
      value: undefined,
      channelData:
       { channelId: 'slack',
         serviceUrl: '',
         conversation: { id: 'someChannel', thread_ts: null },
         from: { id: '' },
         recipient: { id: 'someUserId' } },
      channelId: 'slack',
      serviceUrl: undefined,
      conversation: { id: 'someChannel', thread_ts: null },
      from: { id: undefined },
      recipient: { id: 'someUserId' } 
      }
     */

    return assert.equal(message.text, 'help message');
});
```

## Additional Information 
Botkit depends on adapters (Slack, Facebook, MS Teams, etc).
Botkit-Mock is an extension of Botkit that provides an interface for accepting user messages through `.usersInput`. You can connect any valid Botkit adapters to Botkit-Mock to extend Botkit-Mock, although currently, we have only provided an extension for [Slack](lib/Slack).

## Slack Adapter Information
`SlackApiMock` - Binds the following properties to the Botkit-Mock `controller`.
* `controller.axiosMockAdapter` - [Axios mock](https://github.com/ctimmerm/axios-mock-adapter) helps to mock requests to the Slack API.  [Here](/test/updateApiResponsesSpec.js) are examples of this being used.
* `controller.apiLogByKey` - This contains information about results of requests through `bot.api`.
* `controller.httpBodyLog` - This contains an array of Botkit responses to Slack usually set through `httpBody()`.

## .usersInput options
1. `user` user slackId (required) (string)
2. `channel` is a channel where user sends messages (required) (string)
3. `type` specify botkit message type. ie `direct_message`, `message_received`, `interactive_message_callback`. (defaults to `direct_message`) (string)
4. `messages` (array) that includes:
    - `isAssertion` indicates which conversation response array to return in `.then()` in multi-user testing. (required) (boolean)
    - `deep` indicates the index of the conversation response to return in `.then()`. 0 (default) is the last response, 1 is the second-to-last, etc.. (integer)
    - `timeout` set timeout for message in milliseconds (integer)
    - `waitBefore` alias for `timeout`, indicates how many milliseconds to wait before sending the message to the bot (integer)
    - `waitAfter` indicates how many milliseconds to wait for the bot response, useful for long-running commands (integer)
    - `text` the message's text (string)
    - `channel` indicates the channel the message was sent in. This overrides the channel defined in `usersInput` for this current message. (string)
    - ...any other fields you may be testing for including `attachments`, `callback_id`, etc...


## Contributing ##
Botkit-Mock supports all of Botkit's core functionality by default. We also support extended Slack and Facebook functionality, via adapters.

To add functionality to Botkit-Mock for your favorite chat platform, please open an issue.

## Examples ##

- [botkit-slack](examples/botkit-slack) - tests from a fresh Botkit starter kit, from the Yeoman generator or [a starter kit on Glitch](https://glitch.com/botkit)
                                                    
```
npm install -g yo generator-botkit
yo botkit
```

- [Slack API](./tests/updateApiResponsesSpec.js) - API calls and API response overrides


Built by the team at https://www.gratify.ai.

Like Botkit-Mock? Donate BTC to our team: 1KwpqzTvpLWiUST2V5wmPiT3twwc1pZ9tP

## Change Log
See change log [here](CHANGELOG.md)