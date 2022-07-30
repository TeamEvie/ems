
# ems 🐶🔧

Evie's codespace keeps growing uncontrollably, so we decided to split every module into its microservices. This gives us the freedom to create our modules in whatever languages we desire, make it easier to maintain, and add more features to Evie.  

## How does it work?  

Each EMS should be able to process an environment variable `REST_PROXY` and use it as its base Discord API constant. This will make every request go through the proxy and also tell the EMS to use [ClientPack](https://github.com/TeamEvie/ClientPack) our Gateway proxy. At the time of writing this ClientPack will send every gateway event, though in the future we will mixin command names and interaction custom names in the identify payload to specify what the EMS should get sent.

### Disnake Example

```py
proxy_on = os.environ.get("REST_PROXY") is not None

if proxy_on:
    http.Route.BASE = os.environ.get("REST_PROXY")
    print("Using proxy")
else:
    print("Not using proxy")
```

### Discord.js Example

```js
const client = new Client({
  intents: 37383,
  rest: {
    api: getSecret("REST_PROXY", undefined),
  },
  shards: "auto",
});
```

## Configuration

Most EMS's would require per guild configuration except for static EMS's. Use gRPC to communicate with the [core's api](https://github.com/TeamEvie/Evie/blob/main/protos/leash.proto), otherwise/also you can use Redis for non-guild configuration for example a user's level.

## Registering application commands

If a microservice depends on its application commands, it shouldn't register them. Rather the [core](https://github.com/TeamEvie/Evie/tree/main/services/bot) should, here's a ping command for example:

### Registering:

```ts
// services/bot/src/commands/System/kord.ts

import { registeredGuilds } from "@evie/config";
import { ApplyOptions } from "@sapphire/decorators";
import { ApplicationCommandRegistry, ChatInputCommand, Command, RegisterBehavior } from "@sapphire/framework";

@ApplyOptions<ChatInputCommand.Options>({
	description: "hello kord?",
})
export class Kord extends Command {
	public override registerApplicationCommands(registry: ApplicationCommandRegistry) {
		registry.registerChatInputCommand((builder) => builder.setName(this.name).setDescription(this.description), {
			guildIds: registeredGuilds,
			behaviorWhenNotIdentical: RegisterBehavior.Overwrite,
		});
	}
}
```

## Handling:

```kt
// src/main/kotlin/modules/impl/KordPing.kt
package modules.impl

import dev.kord.core.Kord
import dev.kord.core.behavior.interaction.respondEphemeral
import dev.kord.core.entity.interaction.GuildChatInputCommandInteraction
import modules.Module

class KordPing(client: Kord) : Module(client, "KordPing", arrayOf("kord")) {
    override suspend fun onCommand(req: GuildChatInputCommandInteraction) {
        req.respondEphemeral { content = "pong!" }
    }
}
```
