# udmilla-whatsapp-bot

**Udmilla** is a small bot for WhatsApp made with [whatsapp-web.js](https://github.com/pedroslopez/whatsapp-web.js) library using **Typescript**.

A [hot-swap mode](#hot-swap) is available for development purposes, allowing new commands to be written and tested without restarting the server.

<div align="center">
<img src="./docs/pingpong-example.png">
</div>

## Quick links

* [Quick Start](#quick-start)
* [Create command](#create-command)
    * [Command](#create-command)
        * [Alias](#command-alias)
        * [Callback](#command-callback)
        * [Data](#command-data)
    * [Parameters](#command-parameters)
        * [Types](#parameter-type)
        * [Info](#parameter-info)
        * [Default value](#parameter-default-value)
* [Send response](#send-response)
* [Examples](#examples)
* [Hot-swap mode](#hot-swap)


## Quick Start

Node v12+ is required.

### 1. Clone repository

```bash
git clone https://github.com/Lautauro/udmilla-whatsapp-bot.git
```

### 2. Install packages

```bash
npm i
```

### 3. Build and log in WhatsApp Web

```bash
npm run build
npm start
```

Scan the QR code.

### 4. Done

Start playing!

> [!NOTE]
> You can change the command prefix **"."** and other configurations in **/config/commands.json**.

## Create command

Go to **/src/modules/commands/** and open **commandsList.ts**.


To create a command you need to use the **createCommand** function.

```js
createCommand(['alias'], {
        // Command options (Optional)
        options: { /* ... */ },

        // Command info (Optional, but recommended)
        info: { /* ... */ },
    })
    // Command callback
    .setCallback((args, message) => {
        console.log('Number: ', args[0]);
    })
    .addParameter('number')
.closeCommand(); // Add command to list
```

It's very important not to forget to add **".closeCommand()"** at the end, otherwise the command will not be recognised by the bot.

<div align="center">
<img src="./docs/diagram-command.png">
</div>

### Command alias:

Names by which the command can be invoked

```js
createCommand(['ping', 'pingpong', 'p']) /* etc... */
```
### Command callback:

When a command is invoked, this function is called. You can read the arguments passed by the user using the "args" variable, and use the "message" object to interact with the chat.

```js
createCommand(['alias'])
    // Receive argument of type string
    .addParameter('string')
    // Command execution
    .setCallback(async (args, message) => {
        if (args[0] === 'Hi') {
            // Send "Hello!"
            await sendResponse('Hello!', message);
        } else {
            // Send "Bye!"
            await sendResponse('Bye!', message);
        }
    })
.closeCommand(); // Add command to list
```
[Send Response](#send-response)

I recommend using await for sendResponse as in the example, this will make the command stop executing once the message has been sent.

### Command Data:

```js
{
    options: {
        // If true, command can only be executed by admin
        adminOnly: boolean,

        // The command must be or not a quoted message
        needQuotedMessage: boolean,

        // If the command has a string type parameter, this overrides the need for quotes for multiple lines of text
        disableQuotationMarks: boolean
    },

    // This information will be used by the "help" command to describe the command itself.
    info: {
        name: string,           // Command name
        description: string,    // What it does
    }
}
```

## Command Parameters:

```js
addParameter(ParameterType, ParameterInfo?, defaultValue?);
```

### Parameter Type:
```ts
type ParameterType = 'string' | 'number' | 'boolean' | 'any';
```

### Parameter info:

This information will be used by the "help" command to describe the parameter.

```js
{
    name: string,           // Parameter name
    description: string,    // What it does
    example: string         // Value used in the example 
}
```

### Parameter default value:

If the parameter is an optional argument, set its default value here. Otherwise, **undefined** should do the trick.

### Example:

```ts
createCommand(['foo'])
    .addParameter('string', {
        name: 'Argument',
        description: 'Parameter 1 description',
        example: 'Lorem ipsum dolor',
    })
    // You should always put optional parameters at the end, such as the following.
    // Otherwise it may cause problems.
    .addParameter('number', {
        name: 'Optional Argument',
        description: 'Parameter 2 description',
        example: '123',
    }, 456 /* Now it's an optional parameter */ )
    .setCallback((args, message) => {
        // Do something
    })
.closeCommand();
```

<div align="center">
<img src="./docs/help-command-example1.png">
</div>

## Send Response

Send a message to a chat.

```js
sendResponse(MessageContent, MessageObj, options?)
```

For **MessageObj** info see: https://docs.wwebjs.dev/Message.html

### Message Content

```ts
type MessageContent = string | MessageMedia | Location | Poll | Contact | Contact[] | List | Buttons
```

See: https://docs.wwebjs.dev/Client.html#sendMessage

### Command response options

```js
// All are optional
{
    asReply: boolean;   // Send message as a reply
    asError: boolean;   // Send message as error
    reaction: string;   // Reaction to message. Example: "🐕‍🦺"
    messageOptions: MessageSendOptions;
}
```

For more information on MessageSendOptions, see: https://docs.wwebjs.dev/global.html#MessageSendOptions

## Examples:

Command with **no arguments**:
```js
createCommand(['ping', 'pingpong'], {
        // Command options
        options: { adminOnly: true, },
        // Command info
        info: {
            name: 'Ping',
            description: 'Ping-pong! 🏓',
        }
    })
    .setCallback(async (args, message) => {
        await sendResponse('Pong!', message, {
            reaction: '🏓'
        });
    })
.closeCommand();
```

<div align="center">
<img src="./docs/help-command-example2.png">
</div>


Command with arguments:

```ts
createCommand(['repeat'], {
        info: { name: 'Repeat text', }
    })
    .addParameter('string', {
        name: 'Text',
        description: 'Text to repeat.',
        example: 'Hello x5',
    })
    .addParameter('number', {
        name: 'Times', 
        description: 'Number of times repeated.', 
        example: '5',
    }, 1)
    .setCallback(async (args, message) => {
        let msgToSend: string = args[0];

        for (let i = 1; i < args[1]; i++) {
            msgToSend += '\n' + args[0];
        }

        await sendResponse(msgToSend, message, {
            asReply: true,
            reaction: '🗣️'
        });
    })
.closeCommand();
```

<div align="center">
<img src="./docs/repeat-command-example.png">
</div>

Command with **quoted message**:

```ts
createCommand(['quote', 'cite'], {
        options: { needQuotedMessage: true, },
        info: {
            name: 'Quote this',
            description: 'This command makes an author quote with the selected message. It needs to quote a message to work.',
        }
    })
    .setCallback(async (args, message) => {
        await message.getQuotedMessage()
            .then(async (quotedMessage) => {
                const msgToSend = `*" ${quotedMessage.body} "*\n\n` +
                                  `-  _${quotedMessage._data.notifyName}_`;
                await sendResponse(msgToSend, message, {
                    reaction: '💭'
                });
            });
    })
.closeCommand();
```

## Hot-swap

The bot has a **command hot-swap mode**. It consists of a system that allows the developer to test the commands contained in **"./src/modules/commandsList.ts"** without having to completely restart the server, by simply recompiling the project. This is enabled by the **"hotSwappingEnabled"** configuration variable in **/config/commands.json**.

> [!WARNING]
> It is not recommended to enable this mode for non-development environments.

## Cool-down system

It can happen that a user decides to spam the bot with commands and make it difficult for it to work. To prevent this, there is a cool-down system. Every time a user sends a command, a timestamp is stored and based on certain conditions it is decided whether the command should be executed or not. If not enough time has elapsed since the last execution of a command, it will be ignored. The timeout will increase progressively for each command sent, you can observe these values in **"./src/modules/commands/cooldown.ts"** in the **COOLDOWN_MULTIPLIER** variable. On the other hand, if the timeout has been reached or even exceeded, the cool-down will gradually decrease.

This system also makes it possible to prevent a user from executing a command if a command is already being executed by the same user. To achieve a good performance of the system, it is recommended to use asynchronous functions for callbacks and await functions to wait for the completion of a promise. For example:

```ts
createCommand(['ping'])
    // Async
    .setCallback(async (args, message) => {
        // Await
        await sendResponse('Pong!', message, {
            reaction: '🏓',
        });
    })
.closeCommand();
```

In this example, once the message "Pong!" is sent, the execution of the command is considered as completed.