# Bot Creator - Kotlin Development Onboarding Guide

Welcome! This guide will help you understand the technologies used in this bot and how to work with them. If you're coming from JavaScript/TypeScript, don't worry - Kotlin is similar but has some key differences.

## Table of Contents

1. [Kotlin Basics](#kotlin-basics)
2. [JDA (Java Discord API)](#jda-java-discord-api)
3. [Exposed ORM (Database)](#exposed-orm-database)
4. [Configuration System](#configuration-system)
5. [Project Structure](#project-structure)
6. [Common Patterns](#common-patterns)
7. [Building & Running](#building--running)

---

## Kotlin Basics

### What is Kotlin?

Kotlin is a modern programming language that runs on the Java Virtual Machine (JVM). It's designed to be more concise and safer than Java while being fully interoperable with Java code.

### Key Language Features

#### 1. **Variables and Types**

```kotlin
// Immutable variable (preferred)
val name: String = "Phoenix"

// Mutable variable
var count: Int = 5
count = 10  // OK

// Type inference (compiler guesses the type)
val message = "Hello"  // Automatically inferred as String
```

**Important:** Always prefer `val` over `var`. Only use `var` when you actually need to change the value.

#### 2. **Data Classes**

Data classes are like the object types in JavaScript - they're perfect for holding structured data:

```kotlin
// Our bot uses this in Configs.kt
data class BotConfig(
    val botToken: String,
    val status: OnlineStatus = OnlineStatus.ONLINE,
    val activity: ActivityType? = ActivityType.PLAYING,
    val activityName: String? = "with the API"
)
```

Key points:
- `val` = immutable property
- `= OnlineStatus.ONLINE` = default value (optional)
- `?` = nullable (can be null)

#### 3. **Null Safety** ⭐ (Very Important!)

Kotlin forces you to handle `null` safely:

```kotlin
// This is NOT allowed in Kotlin - type system prevents null by default
val name: String = null  // ❌ Compilation error

// If you want nullable, use the ? operator
val name: String? = null  // ✅ OK

// You MUST check before using
val length = name?.length  // Returns Int? (nullable int), returns null if name is null
val length = name!!.length  // Use !! only if you're 100% sure it's not null (risky!)
val length = name?.length ?: 0  // Use Elvis operator: if name is null, use 0

// Smart cast - after checking, compiler knows it's not null
if (name != null) {
    println(name.length)  // No need for name?.length here
}
```

#### 4. **Extension Functions**

You can add functions to existing types:

```kotlin
// This is added to String type
fun String.isValidToken(): Boolean {
    return this.length > 20
}

// Use it like a normal method
val token = "mytoken12345"
if (token.isValidToken()) { ... }
```

#### 5. **Lambda Functions**

Like arrow functions in JavaScript:

```kotlin
// Normal function
fun double(x: Int): Int {
    return x * 2
}

// Lambda (equivalent)
val double = { x: Int -> x * 2 }

// In higher-order functions (functions that take functions)
listOf(1, 2, 3).map { x -> x * 2 }  // Returns [2, 4, 6]

// If parameter is obvious, you can use 'it'
listOf(1, 2, 3).map { it * 2 }
```

#### 6. **Sealed Classes & When Expressions**

Sealed classes are used for type-safe enums with data:

```kotlin
// From Config.kt - a sealed class for representing different outcomes
sealed class ConfigStatus {
    data class Loaded(val config: BotConfig) : ConfigStatus()
    object CreatedFromTemplate : ConfigStatus()
    data class Error(val message: String, val throwable: Throwable? = null) : ConfigStatus()
}

// Using it with 'when' (like switch but better)
when (val status = configLoader.load()) {
    is ConfigStatus.Loaded -> {
        println("Config loaded: ${status.config}")  // status.config is available here
    }
    is ConfigStatus.CreatedFromTemplate -> {
        println("Config created from template")
    }
    is ConfigStatus.Error -> {
        println("Error: ${status.message}")  // status.message is available here
    }
}
```

#### 7. **Scope Functions** (`.let`, `.apply`, `.run`, `.also`)

These are powerful tools for working with objects:

```kotlin
// .let - transform or process an object
val result = config?.botToken?.let { token ->
    validateToken(token)
}

// .apply - configure and return the same object
val jdaBuilder = JDABuilder.createDefault(token)
    .apply {
        setStatus(OnlineStatus.ONLINE)
        setActivity(ActivityType.PLAYING, "with the API")
    }

// .run - like let but with 'this' instead of parameter
val description = user.run {
    "${this.username}#${this.discriminator}"
}

// .also - do something and return the same object (often used for logging)
val message = "Hello".also {
    logger.info("Created message: $it")
}
```

#### 8. **Object Declarations** (Singletons)

In the codebase, you'll see:

```kotlin
// This is a singleton - only one instance ever exists
object DatabaseManager {
    fun initialize() { ... }
}

// Use it
DatabaseManager.initialize()
```

---

## JDA (Java Discord API)

JDA is the library we use to interact with Discord. It's similar to discord.js but for Java/Kotlin.

### Core Concepts

#### 1. **JDA Object**

The main entry point for everything Discord-related:

```kotlin
// How it's created in Bot.kt
val jda = JDABuilder.createDefault(botToken)
    .setStatus(OnlineStatus.ONLINE)
    .setActivity(ActivityType.PLAYING, "with the API")
    .enableIntents(GatewayIntent.GUILD_MEMBERS)
    .build()
    .awaitReady()  // Wait until bot is connected

// Now you can use it to get guilds, users, channels, etc.
val guild = jda.getGuildById("123456789")
val user = jda.getUserById("987654321")
```

#### 2. **Intents**

Intents tell Discord what data your bot needs. They're like "permissions" for data:

```kotlin
.enableIntents(
    GatewayIntent.GUILD_MEMBERS,      // Member join/leave events
    GatewayIntent.MESSAGE_CONTENT,    // Read message contents
    GatewayIntent.GUILD_PRESENCES     // User presence updates
)
```

#### 3. **Event Listeners**

Listen for Discord events by extending `ListenerAdapter`:

```kotlin
class ClearCommand : ListenerAdapter() {
    init {
        BotManager.getJda().addEventListener(this)  // Register listener
    }

    override fun onSlashCommandInteraction(event: SlashCommandInteractionEvent) {
        if (event.name == "clear") {
            // Handle /clear command
            event.reply("Messages cleared").queue()
        }
    }
}
```

Common events:
- `onSlashCommandInteraction` - Slash commands
- `onButtonInteraction` - Button clicks
- `onStringSelectInteraction` - Select menu clicks
- `onGuildMemberJoin` - Member joins
- `onMessageReceived` - Message sent

#### 4. **Slash Commands**

Modern Discord commands (start with `/`):

```kotlin
// Define the command structure
fun getCommandData() = Commands.slash("clear", "Clears your messages")
    .addOption(OptionType.INTEGER, "amount", "How many messages to clear")

// Handle it
override fun onSlashCommandInteraction(event: SlashCommandInteractionEvent) {
    if (event.name == "clear") {
        val amount = event.getOption("amount")?.asInt  // Get the option value
        event.reply("Clearing $amount messages").queue()
    }
}
```

#### 5. **Queue vs Complete**

Many JDA operations are async. Always use `.queue()` to send them:

```kotlin
// ✅ Correct - async, doesn't block
channel.sendMessage("Hello").queue()

// ❌ Wrong - would block the bot
channel.sendMessage("Hello").complete()

// With callbacks
message.delete().queue(
    { logger.info("Message deleted") },
    { error -> logger.error("Failed to delete: ${error.message}") }
)
```

#### 6. **Getting Discord Objects**

```kotlin
// From guild
val guild = jda.getGuildById("123456789")
val channel = guild?.getTextChannelById("987654321")
val member = guild?.getMemberById("654321987")

// From event
val user = event.user  // The user who triggered the event
val member = event.member  // The member object
val guild = event.guild  // The guild where it happened
val channel = event.channel  // The channel
```

---

## Exposed ORM (Database)

Exposed is an ORM (Object-Relational Mapping) library for working with databases. It's like an Object Document Mapper but for SQL.

### Database Setup

```kotlin
// In DatabaseManager.kt
fun initialize() {
    val database = Database.connect("jdbc:sqlite:database.db")  // SQLite file
    TransactionManager.defaultDatabase = database
    
    transaction {
        SchemaUtils.create(ConfigTable)  // Create tables if they don't exist
    }
}
```

### Defining Tables

```kotlin
// In ConfigTable.kt
object ConfigTable : Table("tblConfig") {
    val id = long("id")
    val checkStatus = varchar("checkStatus", 255)
    
    override val primaryKey = PrimaryKey(id)
}
```

Key points:
- `Table("tblConfig")` - the table name in the database
- `long("id")` - a BIGINT column
- `varchar("checkStatus", 255)` - VARCHAR column with max 255 characters
- Other types: `integer()`, `text()`, `boolean()`, `datetime()`, etc.

### CRUD Operations (Create, Read, Update, Delete)

```kotlin
import org.jetbrains.exposed.sql.*
import org.jetbrains.exposed.sql.transactions.transaction

// Create (Insert)
transaction {
    ConfigTable.insert {
        it[id] = 123
        it[checkStatus] = "pending"
    }
}

// Read (Select)
transaction {
    val row = ConfigTable.selectAll()
        .where { ConfigTable.id eq 123 }
        .firstOrNull()
    
    val status = row?.get(ConfigTable.checkStatus)
}

// Update
transaction {
    ConfigTable.update({ ConfigTable.id eq 123 }) {
        it[checkStatus] = "resolved"
    }
}

// Delete
transaction {
    ConfigTable.deleteWhere { ConfigTable.id eq 123 }
}
```

### Important: Always Use Transactions

Every database operation must be inside a `transaction { }` block:

```kotlin
// ✅ Correct
transaction {
    ConfigTable.insert { ... }
}

// ❌ Wrong - will throw exception
ConfigTable.insert { ... }
```

---

## Configuration System

The bot loads configuration from a properties file. Here's how it works:

### The Config Classes

**BotConfig.kt** - The data structure:
```kotlin
data class BotConfig(
    val botToken: String,              // Required
    val status: OnlineStatus = OnlineStatus.ONLINE,  // Optional, has default
    val activity: ActivityType? = ActivityType.PLAYING,  // Optional, nullable
    val activityName: String? = "with the API"       // Optional, nullable
)
```

**Config.kt** - Loads from file:
```kotlin
val config = Config("bot.properties")
when (val status = config.load()) {
    is Config.ConfigStatus.Loaded -> {
        val botConfig = status.config
        // Use botConfig.botToken, etc.
    }
    is Config.ConfigStatus.CreatedFromTemplate -> {
        // File was created, user needs to edit it
    }
    is Config.ConfigStatus.Error -> {
        println("Error: ${status.message}")
    }
}
```

### Using Config Values

Once you have a `BotConfig`, access values like:

```kotlin
val token = config.botToken
val status = config.status
val activity = config.activity
val activityName = config.activityName  // Might be null!

// If value might be null, use safe navigation
val name = config.activityName?.toUpperCase()

// Or provide a default
val name = config.activityName ?: "playing with the API"
```

### Adding New Config Values

1. Add to `BotConfig` in **Configs.kt**:
```kotlin
data class BotConfig(
    val botToken: String,
    val ticketCategoryId: String = "",  // New field
    val status: OnlineStatus = OnlineStatus.ONLINE,
    // ...
)
```

2. Add to **bot.properties**:
```properties
bot-token=YOUR_TOKEN_HERE
ticket-category-id=123456789
status=ONLINE
activity=PLAYING
activity-name=with the API
```

3. Use in your code:
```kotlin
val categoryId = config.ticketCategoryId
```

---

## Project Structure

```
src/
├── main/
│   ├── kotlin/
│   │   └── phoenix/botcreator/
│   │       ├── commands/          # Command handlers
│   │       │   ├── ClearCommand.kt
│   │       │   └── HybridCommand.kt
│   │       ├── config/            # Configuration
│   │       │   ├── Config.kt      # Loader
│   │       │   └── Configs.kt     # Data class
│   │       ├── core/              # Core bot functionality
│   │       │   ├── Bot.kt         # Main JDA setup
│   │       │   ├── BotManager.kt  # Singleton accessor
│   │       │   ├── Main.kt        # Entry point
│   │       │   ├── RegisterSlashCommands.kt
│   │       │   └── ConsoleInputHandler.kt
│   │       ├── sql/               # Database
│   │       │   ├── DatabaseManager.kt
│   │       │   └── tables/
│   │       │       └── ConfigTable.kt
│   │       └── util/              # Utilities
│   │           ├── ErrorHandler.java
│   │           └── TextUtils.java
│   └── resources/
│       ├── bot.properties         # Configuration file
│       └── logback.xml            # Logging config
```

### Key Files

- **Main.kt** - Entry point, loads config and initializes bot
- **Bot.kt** - Creates JDA instance and sets up bot
- **BotManager.kt** - Singleton to access the JDA instance from anywhere
- **DatabaseManager.kt** - Initializes database connection
- **commands/** - Add new commands here (create a new file for each command)

---

## Common Patterns

### Pattern 1: Creating a New Command

```kotlin
package phoenix.botcreator.commands

import net.dv8tion.jda.api.events.interaction.command.SlashCommandInteractionEvent
import net.dv8tion.jda.api.hooks.ListenerAdapter
import net.dv8tion.jda.api.interactions.commands.OptionType
import net.dv8tion.jda.api.interactions.commands.build.Commands
import phoenix.botcreator.core.BotManager

private const val NAME = "mycommand"
private const val OPTION_NAME = "option1"

class MyCommand : ListenerAdapter() {
    init {
        BotManager.getJda().addEventListener(this)
    }

    fun getCommandData() = Commands.slash(NAME, "Description of command")
        .addOption(OptionType.STRING, OPTION_NAME, "Description of option")

    override fun onSlashCommandInteraction(event: SlashCommandInteractionEvent) {
        event.takeIf { it.name == NAME }?.let {
            val optionValue = it.getOption(OPTION_NAME)?.asString
            
            it.reply("Response").queue()
        }
    }
}
```

Then register it in **RegisterSlashCommands.kt**:
```kotlin
MyCommand().getCommandData()  // Add this to the commands list
```

### Pattern 2: Database Operation with Error Handling

```kotlin
import org.jetbrains.exposed.sql.transactions.transaction
import org.slf4j.LoggerFactory

private val logger = LoggerFactory.getLogger(MyClass::class.java)

fun saveTicketStatus(ticketId: Long, status: String) {
    try {
        transaction {
            ConfigTable.update({ ConfigTable.id eq ticketId }) {
                it[checkStatus] = status
            }
        }
        logger.info("Ticket $ticketId updated to $status")
    } catch (e: Exception) {
        logger.error("Failed to update ticket: ${e.message}", e)
        // Rethrow or handle as needed
    }
}
```

### Pattern 3: Using Null Safety in Discord Operations

```kotlin
val user = event.user  // Never null in event context
val member = event.member  // Can be null (user might not be a member)
val guild = event.guild  // Can be null (might be in DM)

// Safe way to get guild
val guildId = guild?.id ?: run {
    event.reply("This command only works in servers").queue()
    return
}

// Or check at start
if (guild == null) {
    event.reply("This command only works in servers").queue()
    return
}

// Now guild is definitely not null
val owner = guild.owner  // Safe to access
```

### Pattern 4: Logging

```kotlin
import org.slf4j.LoggerFactory

private val logger = LoggerFactory.getLogger(MyClass::class.java)

logger.info("Informational message: $variable")
logger.warn("Warning: something might be wrong")
logger.error("Error: ${exception.message}", exception)  // Include exception for stack trace
logger.debug("Debug info: $detail")  // Only in debug mode
```

---

## Building & Running

### Prerequisites

- Java 21 or higher
- Gradle (included via gradlew)

### Build the Project

```bash
# Windows PowerShell
./gradlew shadowJar

# This creates bot.jar in build/libs/
```

The `shadowJar` task creates a **fat JAR** that includes all dependencies, so you only need one file to run the bot.

### Run the Bot

```bash
# After building with shadowJar
java -jar build/libs/bot.jar
```

That's it! The bot.jar file is completely standalone and doesn't require any external dependencies.

### Development Tips

1. **Edit bot.properties** - Put your bot token there
2. **Make changes** to Kotlin files
3. **Build with shadowJar** - `./gradlew shadowJar`
4. **Run the JAR** - `java -jar build/libs/bot.jar`
5. **Check logs** - Look at console output for errors

### Common Build Issues

```bash
# Clean build (if you get weird errors)
./gradlew clean shadowJar

# Run with debug output
./gradlew shadowJar --info

# Check dependencies
./gradlew dependencies

# Check for dependency updates
./gradlew dependencyUpdates
```

### Checking for Dependency Updates

This project includes the dependency update checker plugin. To see which dependencies have updates available:

```bash
./gradlew dependencyUpdates
```

This generates a report showing:
- **UP-TO-DATE** - Dependencies at latest version
- **EXCEEDED** - Using a newer version than available (rare)
- **OUTDATED** - Updates available
- **UNRESOLVED** - Could not determine version info

To update dependencies, edit `gradle/libs.versions.toml`.

---

## Tips for Success

### 1. Type Safety is Your Friend
Kotlin's type system will catch many bugs at compile time. Trust the compiler!

### 2. Prefer Immutability
Use `val` instead of `var` whenever possible. It makes code easier to understand.

### 3. Use Null Safety
Never ignore the `?` operator. Handle nulls explicitly.

### 4. Keep Transactions Small
Database transactions should be as short as possible:
```kotlin
// ✅ Good
transaction {
    ConfigTable.update { ... }
}

// ❌ Bad - keeps transaction open while doing other work
transaction {
    ConfigTable.update { ... }
    doSomeSlowOperation()  // Don't do this!
}
```

### 5. Always Queue Discord Operations
Never call `.complete()` on Discord API calls in the main thread.

### 6. Log Everything Important
Good logging helps debug issues later. Always include relevant context.

### 7. Reference the JDA Documentation
Official JDA docs: https://docs.jda.dev/

---

## Quick Reference

### Kotlin Syntax Cheat Sheet

| JavaScript | Kotlin |
|-----------|--------|
| `const x = 5` | `val x = 5` |
| `let x = 5; x = 10` | `var x = 5; x = 10` |
| `function foo() {}` | `fun foo() {}` |
| `const arr = [1,2,3]` | `val arr = listOf(1, 2, 3)` |
| `arr.map(x => x * 2)` | `arr.map { it * 2 }` |
| `if (x) { ... }` | `if (x) { ... }` |
| `x?.prop` | `x?.prop` |
| `x || default` | `x ?: default` |
| `class User {}` | `class User` or `data class User(...)` |

### Common JDA Operations

```kotlin
// Get Discord objects
jda.getGuildById(id)
jda.getUserById(id)
jda.getTextChannelById(id)

// Send messages
channel.sendMessage("text").queue()
channel.sendMessageEmbeds(embed).queue()

// Create channels
guild.createTextChannel("name")
    .setParent(category)
    .queue()

// Delete messages
message.delete().queue()

// Add reactions
message.addReaction("👍").queue()

// Get message history
channel.iterableHistory.asSequence()
    .filter { it.author == user }
    .take(5)
```

---

## Getting Help

- **Kotlin Docs:** https://kotlinlang.org/docs/
- **JDA Docs:** https://docs.jda.dev/
- **Exposed Docs:** https://github.com/JetBrains/Exposed
- **Common Issues:** Check the `#` section at the bottom

Good luck with development! 🚀
