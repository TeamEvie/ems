
# ems 🐶🔧

Evie's codespace keeps growing uncontrollably, so we decided to split every module into its microservices. This gives us the freedom to create our modules in whatever languages we desire, make it easier to maintain, and add more features to Evie.  

## How does it work?  

Currently, the EMS stack works via multiple gateway connections to the same bot account and each microservice handles its interactions. In the future, we plan to proxy the Discord gateway and REST API to filter interactions to the microservice processes.

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
