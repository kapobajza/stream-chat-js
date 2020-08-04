# Stream Chat JS

[![Actions Status](https://github.com/GetStream/stream-chat-js/workflows/build/badge.svg)](https://github.com/GetStream/stream-chat-js/actions)

[![NPM](https://nodei.co/npm/stream-chat.png)](https://www.npmjs.com/package/stream-chat)

stream-chat-js is the official JavaScript client for Stream Chat, a service for building chat applications.

You can sign up for a Stream account at https://getstream.io/chat/get_started/.

## Installation

### Install with NPM

```bash
npm install stream-chat
```

### Install with Yarn

```bash
yarn add stream-chat
```

### Using JS deliver

```html
<script src="https://cdn.jsdelivr.net/npm/stream-chat"></script>
```

## API Documentation

Documentation for this JavaScript client are available at the [Stream Website](https://getstream.io/chat/docs/?language=js).

### Typescript

The StreamChat client is setup to allow extension of the base types through use of generics when instantiated. The default instantiation has all generics set to `Record<string, unknown>`.

```typescript
StreamChat<ChannelType, UserType, MessageType, AttachmentType, ReactionType, EventType>
```

Custom types provided when initializing the client will carry through to all client returns and provide intellisense to queries.

**NOTE:** If you utilize the `setAnonymousUser` function you must account for this in your user types.

```typescript
import { StreamChat } from 'stream-chat';
// or if you are on commonjs
const StreamChat = require('stream-chat').StreamChat;

type ChatChannel = { image: string; category?: string };
type ChatUser1 = { nickname: string; age: number; admin?: boolean };
type ChatUser2 = { nickname: string; avatar?: string };
type UserMessage = { country?: string };
type AdminMessage = { priorityLevel: number };
type ChatAttachment = { originalURL?: string };
type CustomReaction = { size?: number };
type ChatEvent = { quitChannel?: boolean };

// Instantiate a new client (server side)
const client = new StreamChat<
  ChatChannel,
  ChatUser1 | ChatUser2,
  UserMessage | AdminMessage,
  ChatAttachment,
  CustomReaction,
  ChatEvent
>('YOUR_API_KEY', 'API_KEY_SECRET');

/**
 * Instantiate a new client (client side)
 * Unused generics default to Record<string, unknown>
 */
const client = new StreamChat<
  ChatChannel,
  ChatUser1 | ChatUser2,
  UserMessage | AdminMessage
>('YOUR_API_KEY');
```

Query operations will return results that utilize the custom types added via generics. In addition the query filters are type checked and provide intellisense using both the key and type of the parameter to ensure accurate use.

```typescript
// Valid queries
// users: { duration: string; users: UserResponse<ChatUser1 | ChatUser2>[]; }
const users = await client.queryUsers({ id: '1080' });
const users = await client.queryUsers({ nickname: 'streamUser' });
const users = await client.queryUsers({ nickname: { $eq: 'streamUser' } });

// Invalid queries
const users = await client.queryUsers({ nickname: { $contains: ['stream'] } });
const users = await client.queryUsers({ nickname: 1080 });
const users = await client.queryUsers({ name: { $eq: 1080 } });
```

**Note:** If you have differing union types like `ChatUser1 | ChatUser2` or `UserMessage | AdminMessage` you can use [type guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types) to maintain type safety when dealing with the results of queries.

```typescript
function isChatUser1(user: ChatUser1 | ChatUser2): user is ChatUser1 {
  return (user as ChatUser1).age !== undefined;
}

function isAdminMessage(msg: UserMessage | AdminMessage): msg is AdminMessage {
  return (msg as AdminMessage).priorityLevel !== undefined;
}
```

Intellisense, type checking, and return types are provided for all queries.

```typescript
const channel = client.channel('messaging', 'TestChannel');
await channel.create();

// Valid queries
// messages: SearchAPIResponse<UserMessage | AdminMessage, ChatAttachment, ChatChannel, CustomReaction, ChatUser1 | ChatUser2>
const messages = await channel.search({ country: 'NL' });
const messages = await channel.search({ priorityLevel: { $gt: 5 } });
const messages = await channel.search({ $and: [{ priorityLevel: { $gt: 5 } }, { deleted_at: { $exists: false }}] });

// Invalid queries
const messages = await channel.search({ country: { $eq 5 } });
const messages = await channel.search({ $or: [{ id: '2' }, { reaction_counts: { $eq: 'hello' }}] });
```

Custom types are carried into all creation functions as well.

```typescript
// Valid
client.setUser({ id: 'testId', nickname: 'testUser', age: 3 }, 'TestToken');
client.setUser({ id: 'testId', nickname: 'testUser', avatar: 'testAvatar' }, 'TestToken');

// Invalid
client.setUser({ id: 'testId' }, 'TestToken');
client.setUser({ id: 'testId', nickname: true }, 'TestToken');
client.setUser({ id: 'testId', nickname: 'testUser', country: 'NL' }, 'TestToken');
```

## More

- [Logging](docs/logging.md)
- [User Token](docs/userToken.md)

## Publishing a new version

Note that you need 2FA enabled on NPM, publishing with Yarn gives error, use NPM directly for this:

```bash
npm version bug
npm publish
```

## Contributing

We welcome code changes that improve this library or fix a problem, please make sure to follow all best practices and add tests if applicable before submitting a Pull Request on Github. We are very happy to merge your code in the official repository. Make sure to sign our [Contributor License Agreement (CLA)](https://docs.google.com/forms/d/e/1FAIpQLScFKsKkAJI7mhCr7K9rEIOpqIDThrWxuvxnwUq2XkHyG154vQ/viewform) first. See our license file for more details.
