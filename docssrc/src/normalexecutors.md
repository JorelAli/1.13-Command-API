# Normal command executors

Command executors are of the following format, where `sender` is a [`CommandSender`](https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/command/CommandSender.html), and `args` is an `Object[]`, which represents arguments which are parsed by the CommandAPI.

```java
{{#include examples/4.1normalcommandexecutors.java}}
```

With normal command executors, these do not need to return anything. By default, this will return a _success value_ of 1 if it runs successfully, and a _success value_ of 0 if it runs unsuccessfully, either by throwing an exception _(RuntimeException)_ or by forcing the command to fail (See the section on [handling command failures](./commandfailures.html).

## Example - Creating a message broadcasting system

```java
{{#include examples/4.1messagebroadcast.java}}
```

-----

## Restricting who can run your command

The `CommandAPICommand` has multiple different `executes...()` methods that can restrict the command sender to any of the following objects:

- `CommandSender` - No restriction, players, the console etc. can use this command. This is what Bukkit normally uses.
- `Player` - Only in-game players can run this command

- `Entity` - Only entities (therefore, players as well) can run this command
- `BlockCommandSender` - Only command blocks can run this command

This is done using the respective method:

| Restricted sender      | Method to use             |
| ---------------------- | ------------------------- |
| `CommandSender`        | `.executes()`             |
| `Player`               | `.executesPlayer()`       |
| `Entity`               | `.executesEntity()`       |
| `BlockCommandSender`   | `.executesCommandBlock()` |
| `ConsoleCommandSender` | `executesConsole()`       |

### Example - A `/suicide` command

Say we wanted to create a command `/suicide`, which kills the player that executes it. Since this command isn't really "designed" for command senders that are not players, we can restrict it so only players can execute this command (meaning that the console and command blocks cannot run this command). Since it's a player, we can use the `.executesPlayer()` method:

```java
{{#include examples/4.1suicide.java}}
```

## Multiple command executor implementations

The CommandAPI allows you to chain different implementations of the command depending on the type of `CommandSender`. This allows you to easily specify what types of `CommandSender`s are required to run a command.

Extending on the suicide example above, we could write another implementation for a different `CommandSender`. Here, we write an implementation to make entities (non-player) go out with a bang when they run the command (using `/execute as <entity> run <command>` command).

### Example - A `/suicide` command with different implementations

```java
{{#include examples/4.1suicide2.java}}
```

This saves having to use `instanceof` multiple times to check the type of the `CommandSender`.