# LuminaClient: A Deep Dive into Advanced Minecraft Client Modification

<div align="center">
  <img src="https://raw.githubusercontent.com/LuminaDevelopment/LuminaClient/main/src/main/resources/assets/lumina/icon.png" alt="Lumina Client Logo" width="200">
</div>

## Introduction

LuminaClient is a sophisticated ghost client for Minecraft 1.19.4, built on the Fabric modding framework. The name stands for "Light Utilization Management Integrating Notable Acceleration" - a fitting description for a client that seamlessly integrates powerful features while maintaining a low profile. As an alpha-stage project (v1.0.0), LuminaClient showcases advanced software architecture patterns and deep integration with Minecraft's client-side codebase.

This blog post provides a comprehensive technical overview of the LuminaClient project, exploring its architecture, feature set, and the innovative approaches used to extend Minecraft's functionality.

---

## Table of Contents

1. [Technology Stack](#technology-stack)
2. [Project Architecture](#project-architecture)
3. [The Module System](#the-module-system)
4. [Event-Driven Architecture](#event-driven-architecture)
5. [Mixin System: The Foundation](#mixin-system-the-foundation)
6. [Feature Showcase](#feature-showcase)
7. [GUI and HUD Systems](#gui-and-hud-systems)
8. [Configuration and Persistence](#configuration-and-persistence)
9. [Notable Implementations](#notable-implementations)
10. [Code Organization](#code-organization)
11. [Conclusion](#conclusion)

---

## Technology Stack

LuminaClient is built on a modern Java-based technology stack:

### Core Technologies
- **Java 17+**: Target and source compatibility for modern Java features
- **Minecraft Version**: 1.19.4
- **Fabric Loader**: v0.14.17 - A lightweight modding framework
- **Fabric API**: Provides essential hooks and utilities
- **Yarn Mappings**: 1.19.4+build.1 - Human-readable deobfuscated names

### Build System
- **Gradle**: Project automation and dependency management
- **Fabric-Loom**: v1.2-SNAPSHOT - Gradle plugin for Fabric development

### External Dependencies
- **MongoDB Driver** (v4.9.1): Potential server-side data storage integration
- **Apache Log4j2**: Robust logging framework
- **GSON**: JSON parsing and serialization

### Project Scale
- **171 Java files** organized across a modular architecture
- **98 functional modules** spanning 8 categories
- **25 mixin injections** for Minecraft integration
- **15 event types** for reactive programming

---

## Project Architecture

LuminaClient follows a well-structured, modular architecture with clear separation of concerns. The project employs several classic design patterns that ensure maintainability and extensibility.

### Design Patterns

#### 1. Singleton Pattern
Critical components use the singleton pattern for global access:
- `Lumina.INSTANCE` - Main client instance
- `ModuleManager.INSTANCE` - Module registry
- `EventManager` - Event dispatcher

#### 2. Abstract Base Classes
Extensible hierarchies for major components:
- `Module` - Base for all feature modules
- `HudModule` - Base for HUD overlays
- `Setting` - Base for configuration settings
- `Event` - Base for event system

#### 3. Annotation-Driven Configuration
- `@EventTarget` - Marks event listener methods
- `@Mixin` - Declares bytecode injection points

#### 4. Registry Pattern
Central registries manage component lifecycles:
- ModuleManager maintains module instances
- EventManager maintains event-to-listener mappings

#### 5. Observer Pattern
The event system implements observer pattern for decoupled communication between game events and module responses.

### Component Interaction Flow

```
┌─────────────────────────────────────────────────┐
│              Lumina (Main Entry)                │
│          ClientModInitializer Entry Point        │
└───────────────────┬─────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
┌──────────────┐        ┌──────────────┐
│ EventManager │        │ModuleManager │
│  (Dispatch)  │        │  (Registry)  │
└──────┬───────┘        └──────┬───────┘
       │                       │
       │                       ▼
       │              ┌────────────────┐
       │              │  98 Modules    │
       │              │  - Combat      │
       │              │  - Ghost       │
       │              │  - Movement    │
       │              │  - Player      │
       └──────────────┤  - Render      │
                      │  - Misc        │
                      │  - Scanner     │
                      │  - CategoryMng │
                      └────────┬───────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
              ┌──────────┐         ┌──────────┐
              │ Settings │         │  Events  │
              │ System   │         │ Handlers │
              └──────────┘         └──────────┘
```

### Initialization Sequence

When Minecraft launches with LuminaClient, the following initialization sequence occurs:

1. **Fabric Entry**: `Lumina.onInitializeClient()` is called
2. **Event Registration**: Main client registers as event listener
3. **Cape System**: `CapeManager.init()` loads cosmetic capes
4. **GitHub Integration**: `GithubRetriever.retrieve()` checks for updates
5. **Module Initialization**: All 98 modules are instantiated
6. **Config Loading**: Asynchronous thread loads saved configurations
7. **Ready State**: Client is ready for gameplay

---

## The Module System

At the heart of LuminaClient is its modular architecture, which allows features to be toggled independently and configured precisely.

### Module Base Class

Every module extends the abstract `Module` class located at `src/main/java/me/stormcph/lumina/module/Module.java`:

```java
public abstract class Module {
    private String name;
    private String description;
    private String displayName;
    private Category category;
    private int key;
    private boolean enabled;
    private List<Setting> settings;
    private DecelerateAnimation animation;

    public abstract void onEnable();
    public abstract void onDisable();
    public void toggle() { ... }
}
```

**Key Features:**
- **Name & Description**: Human-readable identification
- **Category**: Organizational grouping
- **Keybind**: Customizable activation key (GLFW key codes)
- **Enabled State**: Toggle status
- **Settings**: List of configurable parameters
- **Animation**: Smooth UI transitions

### Module Lifecycle

Modules follow a well-defined lifecycle:

1. **Instantiation**: Created during `ModuleManager.init()`
2. **Configuration**: Settings loaded from JSON config
3. **Enable**: User activates module via keybind or GUI
   - `onEnable()` called
   - Module registers with `EventManager`
4. **Active**: Module listens to relevant events and executes logic
5. **Disable**: User deactivates module
   - `onDisable()` called
   - Module unregisters from `EventManager`
6. **Persistence**: Settings saved to JSON config

### Module Categories

The 98 modules are organized into 8 categories:

#### 1. COMBAT (3 modules)
Offensive gameplay mechanics for direct combat:
- **Killaura**: Automated targeting and attacking within range
- **TriggerBot**: Automatic attack on crosshair target
- **PvpHubExploit**: Server-specific kit exploitation

#### 2. GHOST (9 modules)
Stealth-focused PvP automation (Crystal PvP):
- **CrystalPop**: Intelligent end crystal destruction
- **SCC** (Sword Crystal Combo): Multi-step attack sequence
- **CrystalPlacer**: Automated crystal placement
- **PearlMacro**: Quick ender pearl usage
- **AutoRefill**: Automatic inventory restocking
- **LegitTotem**: Legitimate-looking totem mechanics
- **NoLootBlow**: Prevents destroying valuable loot

#### 3. MOVEMENT (5 modules)
Navigation and mobility exploits:
- **Flight**: Creative-mode flying
- **Scaffold**: Automated block bridging
- **Speed**: Movement acceleration
- **FakeLag**: Network lag simulation
- **Sprint**: Auto-sprint functionality

#### 4. PLAYER (4 modules)
Personal inventory and interaction mechanics:
- **Blink**: Packet buffering with position desync
- **ChestStealer**: Automated chest looting
- **NoFall**: Fall damage negation
- **GhostPlace**: Invisible block placement

#### 5. RENDER (13 modules)
Visual enhancements and HUD elements:
- **Arraylist**: Enabled modules display with animations
- **TargetHUD**: Current target information overlay
- **ClickguiModule**: Configuration GUI access
- **Cape**: Custom cosmetic capes
- **ESP**: Entity highlighting
- **XRay**: Ore visualization
- **FullBright**: Darkness removal
- **NoHurtCam**: Damage shake removal
- **Animations**: Custom hand animations
- **ViewModel**: Hand rendering modifications

#### 6. MISC (7 modules)
Miscellaneous utilities:
- **PacketLogger**: Network traffic debugging
- **Freecam**: Spectator camera mode
- **ChatHandler**: Chat message manipulation
- **NameProtect**: Privacy protection
- **Advertise**: Client promotion system
- **AutoEz**: Automated chat messages

#### 7. SERVER_SCANNER (11 modules)
Server discovery and filtering system with configurable criteria for scanning Minecraft servers.

#### 8. CATEGORY_MNG (6 modules)
UI category management for organizing the ClickGUI interface.

---

## Event-Driven Architecture

LuminaClient uses a sophisticated event system to respond to game events in real-time. This reactive architecture allows modules to remain decoupled from Minecraft's code while still responding to in-game actions.

### Event System Components

#### EventManager
The `EventManager` class serves as the central event dispatcher:

```java
public class EventManager {
    private static Map<Class<? extends Event>, List<Data>> REGISTRY_MAP = new HashMap<>();

    public static void register(Object listener);
    public static void unregister(Object listener);
    public static void call(Event event);
}
```

**Registration Process:**
1. Module calls `EventManager.register(this)`
2. EventManager uses reflection to scan all methods
3. Methods annotated with `@EventTarget` are registered
4. Methods are sorted by priority (0-255)

**Event Dispatch:**
1. Mixin intercepts Minecraft method
2. Creates Event instance
3. Calls `event.call()`
4. EventManager invokes all registered listeners
5. Listeners execute in priority order

#### Event Hierarchy

Base `Event` class provides cancellation support:

```java
public abstract class Event {
    private boolean cancelled;

    public void call() {
        EventManager.call(this);
    }

    public void cancel() {
        this.cancelled = true;
    }
}
```

### Available Event Types

LuminaClient defines 15 event types that modules can listen to:

1. **KeyEvent**: Keyboard input detection
2. **Render2DEvent**: 2D screen rendering (HUD)
3. **Render3DEvent**: 3D world rendering
4. **EventUpdate**: Game tick update (20 times per second)
5. **PacketSendEvent**: Outgoing network packets (cancellable)
6. **PacketReceiveEvent**: Incoming network packets
7. **ChatMessageSentEvent**: Chat message sending (cancellable)
8. **ChatMessageReceivedEvent**: Chat message receiving
9. **RenderLabelEvent**: Entity name tag rendering
10. **HandSwingEvent**: Arm swing animation
11. **PlayerMoveEvent**: Player movement
12. **PlayerInteractItemEvent**: Item right-click
13. **ItemSwitchAnimationEvent**: Hotbar slot switching
14. **PacketEvent**: Base packet event

### Event Usage Example

Here's how a module listens to events:

```java
public class ExampleModule extends Module {
    @EventTarget(priority = 2)
    public void onUpdate(EventUpdate event) {
        // Execute logic every game tick
    }

    @EventTarget(priority = 1)
    public void onPacket(PacketSendEvent event) {
        if (shouldCancelPacket()) {
            event.cancel(); // Prevents packet from being sent
        }
    }
}
```

**Priority System:**
- Higher priority values execute first
- Default priority: 2
- Range: 0-255
- Allows modules to interact with events in specific order

---

## Mixin System: The Foundation

The mixin system is what makes LuminaClient possible. Mixins allow the client to inject custom code into Minecraft's classes at runtime without modifying the original JAR files.

### What Are Mixins?

Mixins are a bytecode manipulation technique that:
- Injects code into existing methods
- Adds new methods/fields to existing classes
- Intercepts method calls and returns
- Redirects or cancels original behavior

### How Mixins Work

1. **Compile Time**: Mixin definitions are compiled normally
2. **Launch Time**: Fabric Loader applies mixins before Minecraft loads
3. **Runtime**: Injected code executes as part of Minecraft's logic

### Key Mixins in LuminaClient

#### ClientPlayNetworkHandlerMixin
**Purpose**: Intercepts packet sending and chat messages

**Injection Points**:
```java
@Inject(method = "sendPacket", at = @At("HEAD"), cancellable = true)
private void onSendPacket(Packet<?> packet, CallbackInfo ci) {
    PacketSendEvent event = new PacketSendEvent(packet);
    event.call();
    if (event.isCancelled()) {
        ci.cancel(); // Prevents packet from being sent
    }
}
```

**Enables**:
- Packet logging
- Packet modification
- Packet cancellation (for exploits)
- Chat message interception

#### MinecraftClientMixin
**Purpose**: Main game loop integration

**Injection Points**:
- Game tick updates
- Client initialization
- Session management

**Enables**:
- Module update ticks
- Global state management

#### InGameHudMixin
**Purpose**: HUD rendering integration

**Injection Points**:
```java
@Inject(method = "render", at = @At("RETURN"))
private void onRender(MatrixStack matrices, float tickDelta, CallbackInfo ci) {
    Render2DEvent event = new Render2DEvent(matrices, tickDelta);
    event.call();
}
```

**Enables**:
- Custom HUD overlays
- Module status displays
- Visual indicators

#### KeyboardMixin
**Purpose**: Keyboard input capture

**Enables**:
- Module keybind system
- Custom key handlers
- GUI activation

#### GameRendererMixin
**Purpose**: Camera and view manipulation

**Methods Modified**:
- `tiltViewWhenHurt()` - Disables damage shake
- `bobView()` - Removes view bobbing
- `getFov()` - Field of view modifications

**Enables**:
- NoHurtCam module
- Freecam perspective changes
- Custom camera effects

#### WorldRendererMixin
**Purpose**: World block rendering

**Enables**:
- XRay (selective block rendering)
- ESP rendering
- Custom world overlays

#### CameraMixin
**Purpose**: Camera positioning

**Additions**:
```java
@Unique
public void moveFreecamBy(double x, double y, double z) {
    // Custom camera movement for Freecam
}
```

**Enables**:
- Freecam out-of-body camera
- Custom camera paths

#### AbstractClientPlayerEntityMixin
**Purpose**: Player model rendering

**Enables**:
- Custom cape rendering
- Model modifications
- Cosmetic features

### Mixin Configuration

All mixins are declared in `src/main/resources/lumina.mixins.json`:

```json
{
  "required": true,
  "minVersion": "0.8",
  "package": "me.stormcph.lumina.mixins",
  "compatibilityLevel": "JAVA_17",
  "client": [
    "ClientPlayNetworkHandlerMixin",
    "MinecraftClientMixin",
    "InGameHudMixin",
    "KeyboardMixin",
    "GameRendererMixin",
    // ... 20 more mixins
  ],
  "injectors": {
    "defaultRequire": 1
  }
}
```

---

## Feature Showcase

Let's explore some of LuminaClient's most notable modules in detail.

### Killaura: Automated Combat

**Category**: Combat
**File**: `src/main/java/me/stormcph/lumina/module/impl/combat/Killaura.java`

Killaura automatically attacks nearby entities within a configurable range.

**Key Features**:
- **Range Setting**: 1-6 blocks (NumberSetting)
- **Priority Modes**: Distance or Health-based targeting
- **Entity Filters**: Players, Mobs, Passive animals
- **Cooldown Respect**: Waits for attack cooldown
- **Rotation Calculation**: Uses `RotationUtil` for aim

**Algorithm**:
1. Scan for entities within range
2. Filter by type (players/mobs/passive)
3. Sort by priority (distance or health)
4. Calculate rotation to target
5. Apply rotation to player
6. Execute attack when cooldown ready

**Settings**:
```java
NumberSetting range = new NumberSetting("Range", 1, 6, 3, 0.1);
ModeSetting priority = new ModeSetting("Priority", "Distance", "Health");
BooleanSetting targetPlayers = new BooleanSetting("Players", true);
BooleanSetting targetMobs = new BooleanSetting("Mobs", false);
```

---

### Scaffold: Automated Bridging

**Category**: Movement
**File**: `src/main/java/me/stormcph/lumina/module/impl/movement/scaffold/Scaffold.java`

Scaffold automatically places blocks beneath the player for bridging across gaps.

**Key Features**:
- **Block Data Caching**: Pre-calculates placement positions
- **Head Turner Mode**: Rotates head away from placed blocks
- **Dynamic Block Selection**: Automatically selects blocks from hotbar
- **Blacklist System**: Avoids placing certain blocks
- **Packet Manipulation**: Sends rotation packets for server sync

**Algorithm**:
1. Detect if block below player is air
2. Calculate placement position and face
3. Select appropriate block from hotbar
4. Send rotation packet to face block
5. Send placement packet
6. Rotate head away (bypass mode)

**Block Data System**:
```java
public class BlockData {
    public BlockPos position;
    public Direction face;

    // Pre-calculated optimal placement data
}
```

**This module is particularly impressive** because it implements sophisticated rotation mathematics and state management to create smooth, legitimate-looking bridging.

---

### Freecam: Spectator Mode

**Category**: Misc
**File**: `src/main/java/me/stormcph/lumina/module/impl/misc/Freecam.java`

Freecam allows the camera to move independently of the player's position.

**Key Features**:
- **Movement Modes**: Velocity-based physics or static movement
- **Packet Cancellation**: Prevents server from updating real position
- **Speed Control**: Configurable movement speed (0.1-5.0)
- **Rotation Modes**: Different camera rotation behaviors
- **Camera Interface**: Uses custom mixin methods

**Movement System**:
```java
// Velocity Mode - Physics-based movement
velocity = velocity.multiply(0.9); // Air friction
velocity = velocity.add(input.multiply(speed));
camera.moveFreecamBy(velocity.x, velocity.y, velocity.z);

// Static Mode - Direct movement
camera.moveFreecamBy(input.x * speed, input.y * speed, input.z * speed);
```

**Server Desync**:
When enabled, Freecam cancels all movement packets, causing:
- Client-side camera moves freely
- Server-side player stays in place
- Other players see player standing still
- Allows scouting without detection

---

### Blink: Position Desync

**Category**: Player
**File**: `src/main/java/me/stormcph/lumina/module/impl/player/Blink.java`

Blink buffers all outgoing packets and releases them on disable, creating a teleportation effect.

**Key Features**:
- **Packet Queue**: Concurrent queue stores packets
- **Fake Player**: Renders client-side player model
- **Synchronization**: Thread-safe packet handling
- **Item Preservation**: Prevents item desync on re-enable

**Algorithm**:
1. **On Enable**:
   - Start capturing packets to queue
   - Spawn FakePlayerEntity at current position
   - Cancel all outgoing packets

2. **While Active**:
   - Player can move freely client-side
   - Server receives no movement updates
   - Packets accumulate in queue

3. **On Disable**:
   - Release all buffered packets at once
   - Player "teleports" to current position
   - Remove FakePlayerEntity
   - Re-sync inventory

**Use Cases**:
- Avoid anti-cheat detection
- Confuse opponents in PvP
- Create unpredictable movement
- Test lag-based exploits

---

### CrystalPop: Intelligent Crystal Combat

**Category**: Ghost
**File**: `src/main/java/me/stormcph/lumina/module/impl/ghost/CrystalPop.java`

CrystalPop automatically destroys end crystals while protecting nearby loot.

**Key Features**:
- **Ray-Cast Detection**: Accurate crystal detection
- **Loot Protection**: Configurable radius for item safety
- **Placement Tracking**: Distinguishes player-placed vs. existing crystals
- **Precious Item Detection**: Identifies valuable items (diamond, netherite, etc.)
- **Smart Targeting**: Avoids crystals near important items

**Loot Protection Algorithm**:
```java
1. Detect end crystal placement
2. Check for dropped items within radius
3. For each item:
   - Is it precious? (diamond, netherite, etc.)
   - Is it within loot radius?
4. If precious items nearby: skip this crystal
5. Otherwise: attack crystal
```

**This module is particularly useful** in Crystal PvP scenarios where preserving loot after kills is important.

---

### SCC: Sword Crystal Combo

**Category**: Ghost
**File**: `src/main/java/me/stormcph/lumina/module/impl/ghost/SCC.java`

SCC (Sword Crystal Combo) executes a complex multi-step attack sequence automatically.

**Key Features**:
- **Packet Interception**: Detects sword swings
- **State Machine**: Tracks combo progress
- **Hotbar Management**: Auto-selects required items
- **Automatic Sequence**: Sword → Obsidian → Crystal
- **Auto-Disable**: Turns off after completion

**Attack Sequence**:
```
1. Player swings sword at enemy
2. SCC detects HandSwingEvent
3. Switch to obsidian
4. Place obsidian at enemy feet
5. Switch to end crystal
6. Place crystal on obsidian
7. Switch back to sword
8. Module auto-disables
```

**State Management**:
```java
boolean hasAttacked = false;      // Track if sword swung
boolean crystalPlaced = false;    // Track if crystal placed
boolean crystalBroken = false;    // Track if crystal destroyed
```

---

### TargetHUD: Enemy Information

**Category**: Render
**File**: `src/main/java/me/stormcph/lumina/module/impl/render/TargetHUD.java`

TargetHUD displays information about the nearest player target.

**Display Information**:
- Player head (rendered from skin texture)
- Player name
- Current health (numerical and bar)
- Health percentage
- Animated health bar transitions

**Rendering**:
```java
1. Find nearest player within range (default 6 blocks)
2. Render background panel
3. Render player head (8x8 skin texture)
4. Render player name
5. Render health bar with animation
6. Render health text (20.0 HP / 100%)
```

**Animation System**:
- Health changes animate smoothly
- Uses `DecelerateAnimation` for easing
- Prevents jarring health bar jumps

---

### XRay: Ore Visualization

**Category**: Render
**File**: `src/main/java/me/stormcph/lumina/module/impl/render/XRay.java`

XRay makes all blocks except ores and liquids invisible.

**Visible Block Types** (21 total):
- Coal, Iron, Gold, Diamond, Emerald, Redstone, Lapis
- Deepslate variants (Diamond, Gold, Iron, etc.)
- Nether ores (Quartz, Gold, Ancient Debris)
- Liquids (Water, Lava)

**Implementation**:
Works by manipulating Minecraft's rendering system through mixins:
1. `WorldRendererMixin` checks if XRay is enabled
2. For each block being rendered:
   - If block is in ore list: render normally
   - If block is not in list: skip rendering
3. Disables chunk culling (renders all loaded chunks)

**Performance Considerations**:
- Disabling culling increases rendering load
- Only recommended for short-term use
- Can cause FPS drops in large areas

---

### Arraylist: Module Display

**Category**: Render
**File**: `src/main/java/me/stormcph/lumina/module/impl/render/Arraylist.java`

Arraylist displays a list of enabled modules on screen with smooth animations.

**Visual Features**:
- **Gradient Colors**: Pink to blue color interpolation
- **Smooth Animations**: DecelerateAnimation for enable/disable
- **Smart Sorting**: Modules sorted by text width
- **Position Modes**: Top-left, top-right, bottom-left, bottom-right
- **Color Customization**: Speed and spread settings

**Color System**:
```java
// Time-based color cycling
float time = System.currentTimeMillis() / colorSpeed;
float offset = index * colorSpread;

// HSB interpolation between two colors
Color color1 = Color.getHSBColor((time + offset) % 1.0f, 1.0f, 1.0f);
Color color2 = Color.getHSBColor((time + offset + 0.5f) % 1.0f, 1.0f, 1.0f);

// Blend between colors
Color finalColor = ColorUtil.interpolate(color1, color2, 0.5f);
```

**Animation System**:
Each module has its own animation instance:
- **Enable**: Animation transitions from 0 to 1
- **Disable**: Animation transitions from 1 to 0
- **Rendering**: Alpha and position based on animation value

---

## GUI and HUD Systems

LuminaClient features two complete GUI implementations and a flexible HUD system.

### ClickGUI Systems

#### New ClickGUI (Tab-Based)
**Location**: `src/main/java/me/stormcph/lumina/ui/clickgui/`

**Architecture**:
```
ClickGui (Main Screen)
├── Tab (Category Headers)
│   ├── Draggable position
│   ├── Collapsible panels
│   ├── Category icons
│   └── Rounded UI elements
└── Panel (Module Lists)
    ├── Module buttons
    ├── Toggle switches
    ├── Setting components
    │   ├── Checkboxes (Boolean)
    │   ├── Sliders (Number)
    │   ├── Dropdowns (Mode)
    │   └── Keybind displays
    └── Expandable settings
```

**User Interactions**:
- **Left-Click Drag**: Move tabs
- **Right-Click**: Toggle panel visibility
- **Module Click**: Toggle module enable/disable
- **Setting Click**: Open setting controls
- **Slider Drag**: Adjust number values
- **Dropdown Click**: Cycle through modes

**Visual Design**:
- Rounded rectangles with shadows
- Gradient backgrounds
- Smooth expand/collapse animations
- Category-specific icons (textures)
- Modern, clean aesthetic

#### Old ClickGUI (Frame-Based)
**Location**: `src/main/java/me/stormcph/lumina/ui/old_ui/`

**Architecture**:
- Traditional frame-based layout
- Multiple draggable frames (one per category)
- Modular button system
- Legacy UI style

**Supported for backwards compatibility** and user preference.

### HUD System

**HudModule Base Class**: `src/main/java/me/stormcph/lumina/module/HudModule.java`

All HUD elements extend `HudModule`:
```java
public abstract class HudModule extends Module {
    protected int x, y;           // Screen position
    protected int width, height;  // Element dimensions

    public abstract void draw(MatrixStack matrices);
}
```

**HUD Configuration Screen**:
- Press **H key** to open HUD editor
- Drag HUD elements to reposition
- Visual feedback (highlight boxes)
- Real-time preview
- Saves positions to config

**Available HUD Modules**:
- Arraylist (enabled modules)
- TargetHUD (target information)
- LuminaLogo (client branding)
- Coordinates display
- FPS counter
- Potion effects
- Armor status

---

## Configuration and Persistence

LuminaClient uses a JSON-based configuration system for persisting module states and settings.

### Configuration Architecture

**Config Location**: `%appdata%\.minecraft\luminaConfig.json`

**Components**:
1. **ConfigReader** (`src/main/java/me/stormcph/lumina/config/ConfigReader.java`)
   - Loads configuration on client start
   - Runs in separate thread (non-blocking)
   - Creates default config if not exists
   - Handles version compatibility

2. **ConfigWriter** (`src/main/java/me/stormcph/lumina/config/ConfigWriter.java`)
   - Saves configuration on changes
   - Thread-safe write operations
   - JSON serialization

### Configuration Format

```json
{
  "Killaura": {
    "enabled": true,
    "Range": 3.0,
    "Priority": "Distance",
    "Players": true,
    "Mobs": false,
    "Passive": false,
    "Keybind": 75
  },
  "Scaffold": {
    "enabled": false,
    "HeadTurner": true,
    "Keybind": 0
  },
  "Arraylist": {
    "enabled": true,
    "ColorSpeed": 1000.0,
    "ColorSpread": 0.1,
    "x": 10,
    "y": 10
  }
}
```

### Settings System

**Abstract Setting Class**: `src/main/java/me/stormcph/lumina/setting/Setting.java`

All settings implement:
```java
public abstract void save(JsonObject object);
public abstract void load(JsonObject object);
```

**Setting Types**:

#### 1. BooleanSetting
```java
BooleanSetting autoSprint = new BooleanSetting("Auto Sprint", true);
autoSprint.toggle();              // Flip value
boolean enabled = autoSprint.isEnabled();
```

**GUI Representation**: Checkbox

#### 2. NumberSetting
```java
NumberSetting range = new NumberSetting("Range", min, max, default, increment);
double value = range.getValue();
int intValue = range.getIntValue();
range.setValue(5.5);
range.increment();                // Add increment value
```

**GUI Representation**: Slider with number display

**Features**:
- Min/Max clamping
- Configurable increment
- Precision rounding
- Float or integer retrieval

#### 3. ModeSetting
```java
ModeSetting priority = new ModeSetting("Priority", "Distance", "Health", "Angle");
String current = priority.getMode();
priority.setMode("Health");
```

**GUI Representation**: Dropdown menu

**Features**:
- Multiple mode options
- String-based selection
- Cyclic navigation

#### 4. KeybindSetting
```java
KeybindSetting keybind = new KeybindSetting("Keybind", GLFW.GLFW_KEY_R);
int key = keybind.getKeyCode();
```

**GUI Representation**: Key display with binding interface

**GLFW Key Codes**: Standard keyboard codes (A=65, Shift=340, etc.)

### Auto-Save System

Configuration is saved automatically when:
- Module is toggled
- Setting is changed
- Client shuts down
- HUD elements are repositioned

This ensures user preferences are never lost.

---

## Notable Implementations

Let's examine some particularly interesting technical implementations in LuminaClient.

### Rotation Calculation

**File**: `src/main/java/me/stormcph/lumina/utils/player/RotationUtil.java`

Many modules need to rotate the player to face specific positions. The rotation calculation uses trigonometry:

```java
public static float[] getRotations(Entity target) {
    // Calculate delta positions
    double deltaX = target.getX() - player.getX();
    double deltaY = target.getY() - player.getY();
    double deltaZ = target.getZ() - player.getZ();

    // Calculate horizontal distance
    double distance = Math.sqrt(deltaX * deltaX + deltaZ * deltaZ);

    // Calculate yaw (horizontal rotation)
    // atan2 returns angle from -PI to PI
    // Subtract 90 because Minecraft's 0° is south, not east
    float yaw = (float)(Math.toDegrees(Math.atan2(deltaZ, deltaX)) - 90.0);

    // Calculate pitch (vertical rotation)
    // Positive pitch is downward in Minecraft
    float pitch = (float)(-Math.toDegrees(Math.atan(deltaY / distance)));

    return new float[]{ yaw, pitch };
}
```

**Mathematical Breakdown**:
- **Yaw**: Horizontal angle (compass direction)
  - Uses `atan2(z, x)` for quadrant-aware angle
  - Range: -180° to 180°
  - 0° = South, 90° = West, -90° = East, ±180° = North

- **Pitch**: Vertical angle (up/down)
  - Uses `atan(y / horizontalDistance)`
  - Range: -90° (straight up) to 90° (straight down)
  - 0° = horizontal

This calculation is used by Killaura, TriggerBot, and other combat modules.

### Animation System

**File**: `src/main/java/me/stormcph/lumina/utils/animations/DecelerateAnimation.java`

LuminaClient uses a custom animation system for smooth UI transitions:

```java
public class DecelerateAnimation extends Animation {
    @Override
    public double getOutput() {
        if (direction == Direction.FORWARDS) {
            if (output >= endPoint) {
                output = endPoint;
                return output;
            }
            // Increase output
            output += (endPoint - initialPoint) / (duration * 50);
        } else {
            if (output <= initialPoint) {
                output = initialPoint;
                return output;
            }
            // Decrease output
            output -= (endPoint - initialPoint) / (duration * 50);
        }
        return output;
    }
}
```

**Usage**:
```java
DecelerateAnimation animation = new DecelerateAnimation(250, 1.0);
animation.setDirection(Direction.FORWARDS);

// In render loop:
double progress = animation.getOutput(); // 0.0 to 1.0
int alpha = (int)(255 * progress);       // Fade in
int offsetY = (int)(20 * (1 - progress)); // Slide in
```

**Applications**:
- Module enable/disable animations in Arraylist
- GUI panel expand/collapse
- HUD element fade in/out
- Smooth value transitions

### Color Interpolation

**File**: `src/main/java/me/stormcph/lumina/utils/render/ColorUtil.java`

The Arraylist module uses sophisticated color cycling:

```java
public static Color interpolate(Color color1, Color color2, double percentage) {
    double inverse = 1.0 - percentage;

    int r = (int)(color1.getRed() * inverse + color2.getRed() * percentage);
    int g = (int)(color1.getGreen() * inverse + color2.getGreen() * percentage);
    int b = (int)(color1.getBlue() * inverse + color2.getBlue() * percentage);
    int a = (int)(color1.getAlpha() * inverse + color2.getAlpha() * percentage);

    return new Color(r, g, b, a);
}
```

**HSB Color Cycling**:
```java
// Create rainbow effect
float hue = (System.currentTimeMillis() % 10000) / 10000.0f;
Color rainbow = Color.getHSBColor(hue, 1.0f, 1.0f);

// Create gradient for modules
for (int i = 0; i < modules.size(); i++) {
    float hue = (System.currentTimeMillis() / 2000.0f + i * 0.05f) % 1.0f;
    Color color = Color.getHSBColor(hue, 0.8f, 1.0f);
}
```

**Result**: Smooth rainbow gradients that cycle over time with configurable speed and spread.

### Fake Player Rendering

**File**: `src/main/java/me/stormcph/lumina/utils/render/FakePlayerEntity.java`

The Blink module renders a fake player entity to show where the real player appears to others:

```java
public class FakePlayerEntity extends OtherClientPlayerEntity {
    public FakePlayerEntity(ClientWorld world, GameProfile profile) {
        super(world, profile);
        copyPositionAndRotation(mc.player);
        copyInventory();
    }

    public void spawn() {
        world.addEntity(getId(), this);
    }

    public void despawn() {
        world.removeEntity(getId());
    }
}
```

**Features**:
- Copies player skin
- Matches player position and rotation
- Shows player inventory items
- Rendered like a real player
- Only visible client-side

### Packet Manipulation

Many modules intercept and modify packets for exploits:

```java
@EventTarget
public void onPacket(PacketSendEvent event) {
    if (event.getPacket() instanceof PlayerMoveC2SPacket) {
        PlayerMoveC2SPacket packet = (PlayerMoveC2SPacket) event.getPacket();

        // Modify rotation
        ((IPMC2SP) packet).setYaw(customYaw);
        ((IPMC2SP) packet).setPitch(customPitch);

        // Or cancel entirely
        // event.cancel();
    }
}
```

**Common Packet Modifications**:
- **Rotation Packets**: Change where player is looking
- **Movement Packets**: Modify position/velocity
- **Interaction Packets**: Alter block placement/breaking
- **Entity Packets**: Modify entity interactions

### Block Data Caching

**File**: `src/main/java/me/stormcph/lumina/module/impl/movement/scaffold/BlockData.java`

Scaffold pre-calculates optimal block placement positions:

```java
public class BlockData {
    public BlockPos position;    // Where to place block
    public Direction face;       // Which face to place on

    public static BlockData getBlockData(BlockPos pos) {
        // Check all 6 faces
        for (Direction face : Direction.values()) {
            BlockPos neighbor = pos.offset(face);
            if (!world.getBlockState(neighbor).isAir()) {
                return new BlockData(neighbor, face.getOpposite());
            }
        }
        return null;
    }
}
```

**Benefits**:
- Reduces calculations per tick
- Enables complex placement logic
- Improves performance
- Allows for placement validation

---

## Code Organization

The LuminaClient codebase follows a clean, modular structure:

```
LuminaClient/
├── src/main/
│   ├── java/me/stormcph/lumina/
│   │   ├── Lumina.java                    # Main entry point
│   │   ├── module/                        # Module system
│   │   │   ├── Module.java                # Base module class
│   │   │   ├── HudModule.java             # Base HUD class
│   │   │   ├── ModuleManager.java         # Module registry
│   │   │   ├── Category.java              # Category enum
│   │   │   ├── Bypass.java                # Bypass classification
│   │   │   └── impl/                      # All module implementations
│   │   │       ├── combat/                # Combat modules (3)
│   │   │       ├── ghost/                 # Ghost modules (9)
│   │   │       ├── movement/              # Movement modules (5)
│   │   │       ├── player/                # Player modules (4)
│   │   │       ├── render/                # Render modules (13)
│   │   │       ├── misc/                  # Misc modules (7)
│   │   │       ├── server_scanner/        # Scanner modules (11)
│   │   │       ├── category_mng/          # Category UI (6)
│   │   │       └── chat/                  # Chat commands
│   │   ├── event/                         # Event system
│   │   │   ├── Event.java                 # Base event class
│   │   │   ├── EventManager.java          # Event dispatcher
│   │   │   ├── EventTarget.java           # Listener annotation
│   │   │   ├── Data.java                  # Listener data holder
│   │   │   └── impl/                      # Event implementations (15)
│   │   ├── setting/                       # Settings system
│   │   │   ├── Setting.java               # Base setting class
│   │   │   └── impl/                      # Setting types (5)
│   │   │       ├── BooleanSetting.java
│   │   │       ├── NumberSetting.java
│   │   │       ├── ModeSetting.java
│   │   │       └── KeybindSetting.java
│   │   ├── ui/                            # GUI systems
│   │   │   ├── clickgui/                  # New tab-based GUI
│   │   │   │   ├── ClickGui.java
│   │   │   │   ├── Tab.java
│   │   │   │   ├── Panel.java
│   │   │   │   └── components/
│   │   │   ├── old_ui/                    # Old frame-based GUI
│   │   │   │   ├── Frame.java
│   │   │   │   ├── Button.java
│   │   │   │   └── components/
│   │   │   └── HudConfigScreen.java       # HUD position editor
│   │   ├── mixins/                        # Minecraft injections (25)
│   │   │   ├── ClientPlayNetworkHandlerMixin.java
│   │   │   ├── MinecraftClientMixin.java
│   │   │   ├── InGameHudMixin.java
│   │   │   ├── KeyboardMixin.java
│   │   │   ├── GameRendererMixin.java
│   │   │   ├── WorldRendererMixin.java
│   │   │   └── ... (19 more mixins)
│   │   ├── mixinterface/                  # Mixin accessor interfaces
│   │   │   ├── IPMC2SP.java               # Packet modification
│   │   │   ├── CameraInterface.java       # Camera access
│   │   │   ├── IVec3d.java                # Vector access
│   │   │   └── ISession.java              # Session modification
│   │   ├── config/                        # Persistence system
│   │   │   ├── ConfigReader.java          # Load config
│   │   │   └── ConfigWriter.java          # Save config
│   │   ├── cape/                          # Cosmetics system
│   │   │   ├── Cape.java
│   │   │   └── CapeManager.java
│   │   └── utils/                         # Utility classes
│   │       ├── player/                    # Player utilities
│   │       │   ├── RotationUtil.java      # Rotation math
│   │       │   ├── PacketUtil.java        # Packet sending
│   │       │   └── ChatUtils.java         # Chat messages
│   │       ├── render/                    # Rendering utilities
│   │       │   ├── RenderUtils.java       # Shapes/primitives
│   │       │   ├── ColorUtil.java         # Color manipulation
│   │       │   ├── SFUtils.java           # Shape interface
│   │       │   └── FakePlayerEntity.java  # Fake player model
│   │       ├── animations/                # Animation system
│   │       │   ├── Animation.java         # Base animation
│   │       │   ├── DecelerateAnimation.java
│   │       │   ├── Direction.java         # Animation direction
│   │       │   └── Pair.java              # Generic tuple
│   │       └── misc/                      # Miscellaneous
│   │           ├── TimerUtil.java         # Timing/cooldowns
│   │           ├── JsonUtil.java          # JSON helpers
│   │           └── GithubRetriever.java   # Update checker
│   └── resources/
│       ├── fabric.mod.json                # Fabric metadata
│       ├── lumina.mixins.json             # Mixin configuration
│       └── assets/lumina/
│           ├── icon.png                   # Mod icon
│           └── textures/                  # GUI textures
├── build.gradle                           # Build configuration
└── gradle.properties                      # Project properties
```

**Code Statistics**:
- **171 Java source files**
- **Approximately 15,000+ lines of code**
- **98 feature modules**
- **25 mixin injections**
- **15 event types**
- **5 setting types**
- **2 complete GUI implementations**

**Architecture Highlights**:
- Clear separation of concerns
- Package-by-feature organization
- Abstract base classes for extensibility
- Consistent naming conventions
- Well-defined interfaces

---

## Conclusion

LuminaClient represents a sophisticated approach to Minecraft client modification, combining:

### Technical Excellence
- **Modular Architecture**: 98 independent modules with clean abstractions
- **Event-Driven Design**: Reactive programming model for game integration
- **Mixin System**: Non-invasive bytecode injection for deep hooks
- **Configuration Management**: JSON-based persistence with thread safety
- **Animation System**: Smooth UI transitions and visual feedback

### Feature Richness
- **Combat Automation**: Killaura, TriggerBot, Crystal PvP mechanics
- **Movement Exploits**: Flight, Scaffold, Speed, Blink
- **Visual Enhancements**: ESP, XRay, Custom HUD, Animations
- **Utility Tools**: Freecam, Packet Logger, Chat Handler
- **Server Scanner**: Advanced server discovery system

### User Experience
- **Dual GUI Systems**: Modern tab-based and legacy frame-based
- **Customizable HUD**: Drag-and-drop positioning
- **Extensive Settings**: Per-module configuration
- **Keybind System**: Customizable hotkeys
- **Visual Feedback**: Smooth animations and clear indicators

### Development Quality
- **Clean Code**: Well-organized package structure
- **Design Patterns**: Singleton, Abstract Factory, Observer
- **Extensibility**: Easy to add new modules and features
- **Documentation**: Clear class and method purposes
- **Community**: Open source with active development

### Educational Value
LuminaClient serves as an excellent case study for:
- **Bytecode Manipulation**: Understanding mixin technology
- **Event Systems**: Building reactive architectures
- **GUI Development**: Creating custom Minecraft interfaces
- **Game Modding**: Deep integration with game engines
- **Software Architecture**: Applying design patterns at scale

### Project Status
Currently in **alpha (v1.0.0)**, LuminaClient continues to evolve with:
- Active development team
- Community contributions
- Regular updates
- Discord community support
- GitHub repository for collaboration

### Final Thoughts
Whether viewed as a technical achievement, a gaming tool, or an educational resource, LuminaClient demonstrates how modern software engineering principles can be applied to game modification. The project's clean architecture, extensive feature set, and attention to detail make it a noteworthy example of what's possible with Fabric-based Minecraft modding.

**For the curious developer**, the codebase offers valuable insights into:
- How to structure large modding projects
- Best practices for event-driven architectures
- Techniques for GUI development
- Methods for game engine integration
- Patterns for extensible systems

**For the technical user**, LuminaClient provides:
- Powerful automation capabilities
- Extensive customization options
- Professional-grade implementation
- Active development and support

---

## Project Links

- **GitHub Repository**: [LuminaDevelopment/LuminaClient](https://github.com/LuminaDevelopment/LuminaClient)
- **Discord Community**: [Join Discord](https://discord.gg/K8g9hmza)
- **Fabric Modding**: [fabricmc.net](https://fabricmc.net)
- **License**: GPL-3.0

---

## Authors & Contributors

**Core Development Team**:
- Stormcph
- Qweru
- CorruptionHades
- Cornbread2100

**Contributors**:
- Crosby
- Javatrix
- gabswastaken
- And many community members

---

## Disclaimer

This project is created for educational purposes and demonstrates advanced Java programming, bytecode manipulation, and software architecture. Users should be aware of server rules and terms of service when using client modifications. The authors do not endorse cheating or rule-breaking behavior on multiplayer servers.

---

*This blog post was written to provide a comprehensive technical overview of the LuminaClient project. All information is based on analysis of the public source code repository.*

---

**Published**: November 11, 2025
**Version Covered**: v1.0.0 (Alpha)
**Minecraft Version**: 1.19.4
**Fabric Loader**: 0.14.17
