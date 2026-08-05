# OpenSim AI Integration Module

This module seamlessly transforms standard viewer public/private chats and classic NPCs into advanced, interactive AI clients. By leveraging the [OpenRouter](https://openrouter.ai/) API and its extensive ecosystem of services, it bridges the gap between virtual worlds and modern generative AI models. Whether you want to provide helpful automated assistants for your grid's residents or deploy dynamic, domain-specific smart characters, this module delivers a robust, scalable, and highly customizable solution.

To use this integration, you have to:
* **Create a Free OpenRouter Account:** [https://openrouter.ai/](https://openrouter.ai/)
* **Generate a Free OpenRouter API Key:** [https://openrouter.ai/keys](https://openrouter.ai/keys)


## Available Features

* **Private & Public Service:** Configure the module to restrict access to the estate owner or open the AI service to all residents and visitors.
* **AI Assistant:** Automatically launches a persistent private chat window with the AI assistant upon login or teleportation to any module-enabled instance.
* **Public & Private Chat:** Interact with the AI in public chat by prefixing requests with the `@AI:` keyword, or initiate a private session using `@AI: private`.
* **OSSL SmartNPCs:** Unlike standard NPCs, SmartNPCs dynamically process and respond to queries sent via Instant Message (IM) by relaying AI responses.
* **Specialized SmartNPCs:** Leverage OpenRouter's architecture to deploy domain-specific NPCs, such as dedicated assistants for scripting, economics, or event management.
* **Monetization Support:** Toggle between a completely free-to-use tier or implement a pay-per-request model to generate ecosystem revenue.


## Compile & Setup

### 1. Download & Installation
* **Get the code:** Click the green **`<> Code`** button to download the repository as a `.ZIP` file, then extract it.
* **Pre-compiled option:** If you do not want to compile the source code manually, copy the pre-compiled `.dll` file found in the `bin` folder directly into your OpenSim `bin` directory.
* **Source setup:** If compiling, copy the `OpenSimAIModule` folder and paste it into `OpenSimSource\addon-modules\`. 
  * *Note: Do **NOT** place it inside `OpenSimSource\bin\addon-modules\`.*

### 2. Building the Module
* **Prebuild configuration:** Run `runprebuild.bat` (Windows) or `./runprebuild.sh` (Linux/macOS) and wait for the process to complete.
* **Compilation:** Run `Compile.bat` (Windows) or `./Compile.sh` (Linux/macOS).
* **Verification:** Check your OpenSim `bin` folder to ensure that `OpenSim.Region.OptionalModules.AI.dll` was successfully generated.

### 3. Configuration & Permissions
* **Main settings:** Open the `OpenSim.ini` file provided in this repository. Copy the entire `[OpenSimAI]` section and paste it into the `OpenSim.ini` file located in your OpenSim `bin` directory. Insert your OpenRouter API key and adjust the parameters if needed (default settings are optimized for private use).
* **OSSL permissions:** (not necessary but recommended) Copy the repository's `osslEnable.ini` file and paste it into `OpenSim\bin\config-include\`. 
  * *Note: If you have an existing `osslEnable.ini`, open it and append the required `Allow_xxx` lines from the repository's `osslEnable.ini` to the bottom of your file.*

### 4. Verification
Launch your OpenSim server normally and log into the grid. Your new AI assistant will welcome you upon arrival. To confirm everything works flawlessly, send a test question like: *"What can you do for me?"*


## Interacting With The AI Service

### Chat Modes & Interaction

* **Public Chat:** Ask questions in the public channel by prefixing your message with the `@AI:` keyword (e.g., `@AI: what is the hypergrid?`). For cultural and community reasons, the public chat is completely free (non-monetized) and stateless (conversations are not cached). You can also launch a private session by typing `@AI: private` in the public chat.
* **Private Chat:** Unlike public interactions, the private chat caches your conversation history in memory. This enables long, chained, and highly contextual discussions. You do not need to use the `@AI:` prefix here. To start a new conversation topic, type `reset` or `clear` to purge the session cache. Private chat access can be configured as either free or monetized.

### Using OSSL and SmartNPCs

The module implements 4 new OSSL functions: 2 to interact with the AI service and 2 to configure SmartNPCs.

#### 1. osAI
A generic function used to send custom requests directly to the AI service.

**Syntax:**
```lsl
string aiResponse = osAI(string systemPrompt, string userPrompt, integer maxTokens, float temperature);
```

**Parameters:**
* `string systemPrompt`: Defines the AI's identity, role, mission, and operational instructions.
* `string userPrompt`: The specific question or request sent to the AI.
* `integer maxTokens`: The maximum token length for the response (1 token ~ 4 characters).
* `float temperature`: Controls creativity and reasoning precision (ranges from `0.0` for strict/factual to `2.0` for highly creative).

**Returns:** The AI's response as a `string`.

**Example:**
```lsl
string systemPrompt = "You are a systems development engineer specialized in OpenSimulator scripting (LSL, OSSL). You write optimized, lag-free code, expertly managing zone events, network listeners (HTTP/XML-RPC), sensors, and regional database interactions.\n\nAlways return the result in the same language as the user prompt and in a friendly, cheerful and informative way.\n\nClean the response string for Second Life / OpenSim viewer chat protocol. Remove internal control codes. Return only the raw text and no markdown or code blocks.";

string userPrompt = "Please, explain in detail the llParticleSystem() LSL function and its parameters.";

integer maxTokens = 4096;
float temperature = 0.5;

default {
    state_entry() {
        llOwnerSay("Click to generate the AI answer notecard.\nNote: AI responses are not instantaneous! Please be patient...");
		llSetText("Ready", <0.0, 1.0, 0.0>,1.0);
    }

    touch_start(integer i) {
        string aiResponse = osAI(systemPrompt, userPrompt, maxTokens, temperature);
		llSetText("Waiting AI response...\nPlease be patient...", <1.000, 0.522, 0.106>,1.0);
        if (aiResponse != "") {
            osMakeNotecard("aiResponseNotecard", [aiResponse]);
            llOwnerSay("Generated AI answer notecard!");
            llGiveInventory(llGetOwner(), "aiResponseNotecard");
			llRemoveInventory("aiResponseNotecard");
			llSetText("Ready", <0.0, 1.0, 0.0>,1.0);
        } else {
            llOwnerSay("Failed to request the AI service!");
			llSetText("Failed to request\nthe AI service!", <1.0, 0.0, 0.0>,1.0);
        }
    }
}
```

#### 2. osCreateSmartNPC
SmartNPCs behave like regular owned NPCs but come with "a couple of extra neurons"... This function deploys specialized characters that can be interacted with directly via their Instant Message (IM) window (Right-Click > IM).

**Syntax:**
```lsl
key npc = osCreateSmartNPC(string firstName, string lastName, vector position, string notecardName, string expertise);
```

**Parameters:**
* `string firstName`: The SmartNPC's first name.
* `string lastName`: The SmartNPC's last name.
* `vector position`: The region coordinates where the SmartNPC will spawn.
* `string notecardName`: The name of the appearance notecard to apply to the SmartNPC.
* `string expertise`: The specialization field for the SmartNPC (leave as an empty string `""` for default behavior).

**Available Expertise Profiles:**
* `default`: Generalist AI, highly proficient in OpenSim matters (similar to the default Assistant).
* `avatars`: Specialized in avatar design, customization, and rigging.
* `scripting`: Expert in LSL and OSSL scripting architectures.
* `building`: Advanced assistant for 3D building, prim management, and mesh creation.
* `texturing`: Expert in texturing workflow and PBR materials.
* `animations`: Specialized in creating and editing BVH/ANIM assets.
* `environment`: Expert in scene customization, terraforming, and EEP/environment settings.
* `economy`: Specialized in in-world economics, markets, and regional marketing.
* `community`: Expert in events, scheduling, and group management.

**In-World Chat Commands:**
Owners can fetch the full list of expertise profiles by typing `#expertise: list` in the SmartNPC's IM window. To dynamically swap profiles, use `#expertise: {profile_name}` (e.g., `#expertise: scripting`).

**Returns:** The `key` (UUID) of the newly created SmartNPC.

**Example:**
```lsl
key npc = NULL_KEY;
integer listener;
integer channel = 9090;
string expertise = "scripting";

default {
    state_entry() {
        llOwnerSay("Click to start!");
    }

    touch_start(integer i) {
        listener = llListen(channel, "", "", "");
        llDialog(llGetOwner(), "[Create]: Spawn a new SmartNPC.\n[Remove]: Despawn the SmartNPC.\n", ["Create", "Remove", "Close"], channel);
    }

    listen(integer channel, string name, key id, string msg) {
        if (id == llGetOwner()) {
            if (msg == "Create") {
                vector pos = llGetPos() + <1.0, 0.0, 1.0>;
                osAgentSaveAppearance(id, "appearance");
                npc = osCreateSmartNPC("Smart", "NPC", pos, "appearance", expertise);
                llListenRemove(listener);
            }
            if (msg == "Remove") {
                osNpcRemove(npc);
                npc = NULL_KEY;
                llListenRemove(listener);
            }
            if (msg == "Close") {
                llListenRemove(listener);
            }
        }
    }
}
```

#### 3. osSetSmartNPC
Allows developers to dynamically change an existing SmartNPC's expertise profile using code.

**Syntax:**
```lsl
osSetSmartNPC(key npcKey, string expertise);
```

**Parameters:**
* `key npcKey`: The UUID of the active SmartNPC.
* `string expertise`: One of the expertise keywords listed in the profile section above.

**Example:**
```lsl
key npc = NULL_KEY;
integer listener;
integer channel = 9090;

default {
    state_entry() {
        llOwnerSay("Click to start!");
    }

    touch_start(integer i) {
        listener = llListen(channel, "", "", "");
        llDialog(llGetOwner(), "Click [Create] to spawn a SmartNPC, then change its expertise and ask it what it can do for you...\n\n[Create]: Spawn NPC.\n[Remove]: Despawn NPC.\n", ["Create", "Remove", "Close", "Avatars", "Building", "Economy"], channel);
    }

    listen(integer channel, string name, key id, string msg) {
        if (id == llGetOwner()) {
            if (msg == "Create") {
                vector pos = llGetPos() + <1.0, 0.0, 1.0>;
                osAgentSaveAppearance(id, "appearance");
                npc = osCreateSmartNPC("Smart", "NPC", pos, "appearance", "scripting");
                llListenRemove(listener);
            } else if (msg == "Remove") {
                osNpcRemove(npc);
                npc = NULL_KEY;
                llListenRemove(listener);
            } else if (msg == "Close") {
                llListenRemove(listener);
            } else {
                osSetSmartNPC(npc, msg);
            }
        }
    }
}
```

#### 4. osNpcInstantMessage
Instructs the SmartNPC to proactively send an Instant Message (IM) to a specific avatar.

**Syntax:**
```lsl
osNpcInstantMessage(key npcKey, key destination, string message);
```

**Parameters:**
* `key npcKey`: The UUID of the sending SmartNPC.
* `key destination`: The UUID of the target avatar.
* `string message`: The text string to transmit.

**Example:**
```lsl
key npc = NULL_KEY;
integer listener;
integer channel = 9090;
string message = "Hey there! I am a very smart NPC. Ask me anything via IM and see for yourself! :)";

default {
    state_entry() {
        llOwnerSay("Click to start!");
    }

    touch_start(integer i) {
        listener = llListen(channel, "", "", "");
        llDialog(llGetOwner(), "[Create]: Spawn a new SmartNPC.\n[Remove]: Despawn the SmartNPC.\n", ["Create", "Remove", "Close"], channel);
    }

    listen(integer channel, string name, key id, string msg) {
        if (id == llGetOwner()) {
            if (msg == "Create") {
                vector pos = llGetPos() + <1.0, 0.0, 1.0>;
                osAgentSaveAppearance(id, "appearance");
                npc = osCreateSmartNPC("Smart", "NPC", pos, "appearance", "");

                // SmartNPC welcomes the owner immediately upon creation
                osNpcInstantMessage(npc, llGetOwner(), message);

                llListenRemove(listener);
            }
            if (msg == "Remove") {
                osNpcRemove(npc);
                npc = NULL_KEY;
                llListenRemove(listener);
            }
            if (msg == "Close") {
                llListenRemove(listener);
            }
        }
    }
}
```


## The Monetization Strategy

Because the best AI models are not free, and even if the AI service providers platforms offer free (to taste) services, it remains limited and unsuited for heavy workloads like those a grid requires. It is therefore essential to establish a solid financial strategy to keep this service available 24/7 without draining the grid's treasury. 

### How OpenRouter Limits Work
[OpenRouter](https://openrouter.ai) offers around 2,500 free requests upon registration. However, it subsequently [restricts free accounts](https://openrouter.aidocs/api_reference/limits#rate-limits) to a strict daily limit after. To unlock this restriction and access a higher quota of up to 1,000 requests per day, you must add a $10 credit balance to your account. These credits will remain untouched as long as you exclusively query [free models](https://openrouter.aimodels?q=free). If you exceed the daily limit, the service will pause unless a paid fallback model is configured in your `OpenSim.ini` file.

### The Economic Loop
The core strategy relies on "reselling" these 1000 free requests to fund the paid fallback infrastructure. The module prioritizes the free daily quota. If the API triggers a **429 Rate Limit Exceeded** error, the system automatically routes traffic to your designated paid fallback model. This approach establishes a self-sustaining financial cycle. Because users are billed regardless of whether the underlying request was free or paid, your system generates a consistent surplus while guaranteeing uninterrupted 24/7 service. As soon as the free tier quota resets the following day, the module dynamically reverts to the free models to start the cycle anew.

### Recommended Configuration
To keep the service balanced and sustainable, I recommend setting the request price to **1 currency unit**:
```ini
PricePerRequest = 1
```

Additionally, assign the efficient [DeepSeek V4 Flash](https://openrouter.ai/deepseek/deepseek-v4-flash) model as your fallback solution:
```ini
FallbackModelName = "deepseek/deepseek-v4-flash"
```


## What Next?

### 1. Developer Motivation vs. Community "Generosity"
The next depend entirely on the legendary generosity of the Hypergrid communities and their profound love for developers. Mathematically speaking, we are looking at `llAbs(0);`. 
So, please, take a long look in the mirror before opening an issue to spam your *"I want this, I want that..."* feature requests. You are dealing with someone whose current `motivation level` about OpenSim thingies is sitting comfortably at `-1`.

### Origin Story (Or: How Spite Breeds Innovation)
To be perfectly honest, my initial goal was simply to test and dissect the OpenRouter API. If this project magically manifested as a polished C# region module instead of a quick, dirty Python script, you can thank the OpenSim core developer who graciously suggested that I should be banned (from i don't know what ?!?) for simply exploring AI technologies ?!?. So, consider this entire module as my muse's beautifully coded honor finger to the gatekeepers XD (just kidding... or am I?).

### Bugs & Crazy Ideas
* **Found a bug?** If something breaks or completely falls apart, drop it on the **Issues board**. I might look at it.
* **Have a wild idea?** If you have a truly unhinged, crazy, or fun concept, pitch it in the **Discussions board**. If it is chaotic enough, it might just wake up my muse and accidentally get turned into code XD.

#### Who knows? Miracles can happen sometimes...

- BTC: bc1ququ5vxy3yfpqn6dd5p4xfnk35ljfgem2jf2tf2
- LTC: LL6dMuxLCWy6rB7h3AHCR5nkvdV6SoFNEk
- ETH: 0xa3E82b8D653Db863dBB557C8Ed8A01f6570bd990
- USDT ETH Network: 0xa3E82b8D653Db863dBB557C8Ed8A01f6570bd990
- USDT SOL Network: 4NzQh2wW8yxsmQzs5mVdQaaRkcpvvuonj439sQ67DgDv
- DOGE: DFPrdKUtYJS8jqPpmECypdQPumCykK5jHp
- XMR: 4AEJf5HiQkiiafQRzV2gmfJjUHEKUSgetjb7bqn7fQQ7GfJe21nmE29GMBhV1z6pvC45yVKVkvAH97cp4bkPpJHH4m3gZfQ

Enjoy :)
