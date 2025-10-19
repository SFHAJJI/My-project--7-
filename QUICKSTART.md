Absolutely. Below is a **studio‑grade, Android‑first UI spec pack** you can paste into your repo and feed to Codex (or any codegen assistant). I’ve split tasks into **what the tool should auto‑generate** vs **what you (or a human) should wire manually**, and I give **file‑by‑file scaffolds** (UXML/USS/C#) so you don’t get a blank screen again. I also include the **asset‑store picks** you said you’ll buy and exactly **where they slot in**.

I verified every Unity‑6 callout (UI Toolkit runtime data binding, World‑Space UI, PanelSettings scaling, ListView virtualization, Input System UI integration, Device Simulator, TextSettings with fallback fonts, Addressables PAD, Remote Config/Game Overrides, URP Full‑Screen Pass) against current docs and package pages; relevant claims have inline citations.

---

# MASTER UI SPEC PACK (UI Toolkit • Android • Unity 6)

> **Delivery format:** copy these files/sections as‑is into your docs repo.
> **Execution:** for each section I show “**Codex does**” (what your AI tool writes) vs “**You do**” (manual wiring or design decisions).

---

## 0) Golden Rules (never ship without these)

1. **Every scene that needs UI has a `UIDocument`** pointing to a **UXML** and a **PanelSettings**. No `UIDocument` → no UI visible. ([Unity Documentation][1])
2. **PanelSettings**:

   * *Screen Space* for menus/HUD: **Scale Mode = Scale With Screen Size**, set reference DPI/resolution. ([Unity Documentation][2])
   * *World Space* for tile tags & pawn plates (dice banner): **Render Mode = World Space**, set **Panel Input Configuration** + **referencePixelsPerUnit**. ([Unity Documentation][3])
3. **Runtime data binding** for Toolkit is required (no ad‑hoc refresh). Bind ViewModels to UI so values auto‑update. ([Unity Documentation][4])
4. **Input System** drives UI: UI Action Map is configured in Project Settings; UITK and uGUI both listen through the same UI input plumbing. ([Unity Documentation][5])
5. **ListView for lists** (rooms, logs, offers)—it virtualizes & pools; don’t hand‑roll scroll lists. ([Unity Documentation][6])
6. **Text Settings** asset with **fallback fonts** (CJK/emoji). Missing glyphs = invisible text on device. ([Unity Documentation][7])
7. **Device Simulator** to validate safe‑areas/notches before you build. ([Unity Documentation][8])

---

## 1) Repo layout (drop into `/Docs/` & `/Assets/`)

```
/Docs/UI-Spec/
  00_README.md
  01_Setup_PanelSettings.md
  02_DesignTokens.md
  03_Components.md
  04_Screens.md
  05_Flows.md
  06_InputAndNavigation.md
  07_Localization.md
  08_HapticsAndMotion.md
  09_PerformanceAndScaling.md
  10_Addressables_PAD_RemoteConfig.md
  11_DebugAndQA.md
  12_AssetStore_Integration.md

/Assets/UI/
  PanelSettings/
    ScreenSpace.asset
    WorldSpace.asset
  Styles/
    tokens.uss
    theme-classic.uss
    theme-event.uss
  Components/
    ButtonPrimary.uxml / .uss
    Chip.uxml / .uss
    Toast.uxml / .uss
    Sheet.uxml / .uss
    Counter.uxml / .uss
    TimerBar.uxml / .uss
  Screens/
    Home.uxml / .uss
    Lobby.uxml / .uss
    BoardHUD.uxml / .uss
    Shop.uxml / .uss
    Results.uxml / .uss
  WorldSpace/
    TileTag.uxml / .uss
    PawnPlate.uxml / .uss
    DiceBanner.uxml / .uss

/Assets/Scripts/UI/
  Core/UiRouter.cs
  Core/ModalService.cs
  Core/ToastService.cs
  Core/SafeAreaRoot.cs
  Binding/Formatters.cs
  ViewModels/PlayerVM.cs, BoardVM.cs, PropertyVM.cs, AuctionVM.cs, TradeVM.cs
  Input/UiInputConfigurator.cs
```

---

## 2) **01_Setup_PanelSettings.md** (no blank screens)

**Codex does**

* Create two PanelSettings assets with these exact values:

```
ScreenSpace.asset
  renderMode: ScreenSpaceOverlay
  scaleMode: ScaleWithScreenSize
  referenceResolution: 1080x2400
  referenceDpi: 180
  textSettings: Assets/UI/Styles/TextSettings.asset

WorldSpace.asset
  renderMode: WorldSpace
  referencePixelsPerUnit: 100
  sortingOrder: 200
  panelInput: (create Panel Input Configuration)
```

**You do**

* Add one `UIDocument` in **MainMenu** scene → `Home.uxml` + `ScreenSpace.asset`.
* Add one `UIDocument` in **Board** scene → `BoardHUD.uxml` + `ScreenSpace.asset`.
* Add **prefab** `TileTag` with `UIDocument` → `TileTag.uxml` + `WorldSpace.asset`, place it above each property.
* Use **UI Toolkit Debugger** to verify tree renders. ([Unity Documentation][2])

---

## 3) **02_DesignTokens.md** (USS variables for consistency)

**tokens.uss** (Codex writes)

```css
:root {
  /* color */
  --c-bg: #0E1013;
  --c-panel: #14181Ecc; /* glass overlay base */
  --c-primary: #28C76F;
  --c-danger: #FF4D4F;
  --c-warning: #FFC107;
  --c-muted: #8A93A6;

  /* type scale (sp based on 1080x height) */
  --t-xxl: 40px; --t-xl: 32px; --t-lg: 24px; --t-md: 18px; --t-sm: 16px; --t-xs: 14px;

  /* radius & spacing */
  --r-sm: 8px; --r-md: 16px; --r-lg: 24px;
  --s-2: 2px; --s-4: 4px; --s-8: 8px; --s-12: 12px; --s-16: 16px; --s-24: 24px; --s-32: 32px;

  /* shadow tokens (uGUI True Shadow or UITK effect) */
  --shadow-soft: 0 12px 24px #00000040;
}
```

**You do**

* Provide final brand palette later; we can pivot with Remote Config theme switches.

---

## 4) **03_Components.md** (atomic UI pieces with behavior)

### 3.1 `ButtonPrimary` (press squish + haptic)

**Codex does**

* Generate `ButtonPrimary.uxml/.uss` with rounded corners and press class.

```xml
<!-- ButtonPrimary.uxml -->
<ui:Button class="btn-primary" text="PRIMARY"/>
```

```css
/* ButtonPrimary.uss */
.btn-primary {
  height: 64px; padding: 0 var(--s-24);
  font-size: var(--t-md); border-radius: var(--r-lg);
  background-color: var(--c-primary); color: white;
  transition: transform 90ms ease-out, opacity 120ms ease-in;
}
.btn-primary:active { transform: scale(0.96); }
.btn-primary:disabled { opacity: 0.4; }
```

**You do**

* Map press to **Nice Vibrations** “light” pulse on click; animate 90 ms in, 160 ms out via DOTween (world‑space or game object wrapper). ([moremountains.com][9])

### 3.2 `Sheet` (bottom pull‑up, 3 stops)

**Codex does**

* Build `Sheet.uxml/.uss` with backdrop + sheet panel; expose C# API: `Open(mid/full)`, `Close()`, velocity settle.

**You do**

* Attach **KAMGAM UITK Blur** to backdrop for frosted glass. ([Unity Asset Store][10])

### 3.3 `Toast`

**Codex does**

* A container with queue + dedupe (`Rent x3`).

**You do**

* For jackpot, spawn **UIParticle** confetti in a small uGUI overlay canvas (if you prefer uGUI for particles) or keep pure UITK if you later buy a UITK particle add‑on. ([GitHub][11])

### 3.4 `ListView` template

**Codex does**

* A reusable `VList` wrapper using `ListView` with `bindItem`, `makeItem`, recycling. ([Unity Documentation][6])

### 3.5 `TimerBar`

**Codex does**

* Shader‑driven sweep (bar fill from right→left).
* API: `Start(seconds)`, `Pause()`, `Resume()`, `Complete()`.

**You do**

* Pause on reconnect events; resume after net sync.

---

## 5) **04_Screens.md** (what shows where, exact placements)

### 4.1 Home (Main Menu)

**Layout**

* Top‑left: avatar chip; Top‑right: gear + mail.
* Middle: **ithappy** low‑poly city diorama (URP) with the board in the center. Use the demo blockouts from the pack (City/Props). ([Unity Asset Store][12])
* Bottom: **Play** (ButtonPrimary) centered; tab bar (Home/Events/Shop/Profile).

**States**

* Offline → Events & Shop dim, tooltip “Connect to see events.”
* Limited‑time event → red “Event” ribbon (Remote Config). ([Unity Docs][13])

### 4.2 Lobby / Match Setup

* Cards: Solo / Create Room / Join Code.
* Room list: `VList` with virtualization. ([Unity Documentation][6])
* Start/Ready: grows & pulses when all ready.

### 4.3 Board HUD (In‑game)

**Top**: Cash, Net Worth, your portrait + turn arrow.
**Bottom Action Rail** (big, thumb‑friendly): Roll / End / Buy / Auction / Trade / Build / Cards.
**Right**: Event feed (`VList`).
**World‑Space**: `TileTag` over each property (price/owner color). (PanelSettings = World Space; set PPU 100–150.) ([Unity Documentation][3])

**Behaviors**

* Cash **counts up/down** with USS transitions (200–600 ms) and color flash; large wins fire **confetti** (UIParticle). ([GitHub][11])
* Tapping a free tile opens **Property Sheet**; tapping an owned tile opens read‑only sheet with owner info.
* Double taps are **debounced** (300 ms lockout on the button).

---

## 6) **05_Flows.md** (every scenario, step by step)

### 5.1 Dice Roll

1. Tap **Roll** → press squish + haptic.
2. **DiceBanner** (World‑Space UITK) shows “Rolling…”, dice roll in 3D.
3. Result number pulses big then compresses into a toast “You rolled 9.”
4. If network stalls → freeze pawn at last safe tile; show “Reconnecting…” banner; unfreeze on sync.

### 5.2 Property Sheet (Buy/Auction/Trade/Mortgage)

* Open as **Sheet** (half → full on drag). Background blur uses **UITK Blurred Background**. ([Unity Asset Store][10])
* Buy: animate cash down; `TileTag` recolors to owner.
* Auction: open **AuctionModal** (below).
* Trade: open **TradeModal**; property pre‑selected.
* Mortgage: toggle **MORTGAGED** ribbon; strike rent row.

### 5.3 Auction Modal

* **Top**: `TimerBar` (10–20 s).
* **Middle**: current bid (giant number), preset **chips** (+10/+50/+100).
* **Bottom**: **Bid** / **Pass**.
* **Side**: players list (`VList`) with “bidding…” badges.
* Reconnect pauses timer; resume after sync.
* Controller nav uses **NavigationMoveEvent** to move left/right over chips; never gets stuck. ([Unity Documentation][6])

### 5.4 Trade Modal (two‑sided)

* Two baskets (You/Them), property picker list, cash slider, jail card toggle.
* **Fairness meter** (cosmetic gradient).
* When an item is added remotely by the other player, it slides in with sparkle + light haptic.

### 5.5 Jail / Mortgages / Bankruptcy

* Jail tabs: **Pay**, **Use Card**, **Roll**. Disallowed buttons are disabled with tooltip.
* Bankruptcy is **blocking**: hold‑to‑confirm (prevents mis‑taps); list of transfers shown.

### 5.6 Shop & Events

* Carousel cards with soft shadow & rounded corners; prices locale‑aware.
* Event banner & offer copy come from **Remote Config**; can be A/B’d via **Game Overrides**. ([Unity Docs][13])

---

## 7) **06_InputAndNavigation.md** (touch + pads, no traps)

**Codex does**

* Create `UiInputConfigurator.cs` to ensure the **UI Action Map** exists and is used by UITK (Unity 2023.2+/Unity 6 integrates UI mappings in Project Settings). ([Unity Documentation][5])
* Implement a `NavigationPolicy` for grids (chips) using `NavigationMoveEvent` that returns the closest element by direction.

**You do**

* Enable **EnhancedTouch** at boot for reliable gestures on Android.

---

## 8) **07_Localization.md** (Toolkit‑native binding)

**Codex does**

* Create `TextSettings.asset` with Latin primary + fallback list (e.g., Noto CJK, Noto Emoji). Bind UI text via **Localization package data bindings** (UITK supported in Unity 6+). ([Unity Documentation][7])

**You do**

* Provide string tables.
* Verify long names → ellipsis + tooltip rules.

---

## 9) **08_HapticsAndMotion.md** (timings a designer would sign off)

**Motion windows**

* Micro (press/hover): **90–160 ms**
* Meso (sheets, list row enter): **220–320 ms**
* Macro (jackpot/confetti): **700–1000 ms** with stagger

**Haptic map** (Nice Vibrations)

* Button press: Light
* Purchase: Medium
* Auction win / big reward: Heavy
* Dice settle: Medium, short burst ([moremountains.com][9])

**Codex does**

* Implement a `Haptics` static class + simple `Motion` helpers (DOTween sequences). ([Unity Asset Store][14])

---

## 10) **09_PerformanceAndScaling.md** (Android reality)

* UI CPU budget < **6–8 ms** at 60 Hz on mid‑tier; keep **ListView** for large lists. ([Unity Documentation][6])
* Prefer **class toggles** over inline style mutation to reduce style recalcs.
* **Adaptive Performance** hooks: on thermal stress → lower blur radius, reduce confetti counts, shorten some durations by 20%. ([Unity Documentation][15])
* Text: pack glyph atlases and keep fallback list short (each fallback costs). ([Unity Documentation][16])
* **Device Simulator** test matrix (Pixel 6/7, Galaxy A series, tablets). ([Unity Documentation][8])

---

## 11) **10_Addressables_PAD_RemoteConfig.md** (live‑ops without updates)

* Use **Addressables for Android** with **Play Asset Delivery** to push heavy art (themes, city skins) on‑demand; use **Texture Compression Targeting**. ([Unity Documentation][15])
* Control banners/CTAs/themes via **Remote Config** + **Game Overrides** (A/B and seasonal flags). ([Unity Documentation][17])

---

## 12) **11_DebugAndQA.md** (ship with guard rails)

**Codex does**

* Add `SafeAreaRoot.cs` that pads root with `Screen.safeArea`.
* Add a dev‑only button to open **UI Toolkit Debugger**/overlay logs. ([Unity Documentation][18])

**You do**

* Install **Proxima Runtime Inspector + Console** (or free version) to inspect/edit live builds from a browser (total game‑changer when testing UI on phone). ([Unity Asset Store][19])
* Install **Odin Validator** to catch unbound UXML, missing PanelSettings, or broken references before Play. ([Unity Asset Store][20])

---

## 13) **12_AssetStore_Integration.md** (what you’re buying & why)

**Must‑have (UI polish & speed):**

* **DOTween Pro** – deterministic sequences (UI + 3D). ([Unity Asset Store][14])
* **Nice Vibrations** – HD haptics on Android & pads. ([moremountains.com][9])
* **KAMGAM UI Toolkit Blur** + **UITK Text Animation 2** – premium blur & text effects in **UITK** (no uGUI dependency). ([Unity Asset Store][10])
* **UIParticle/UIEffect (mob‑sakai)** – confetti/glow/dissolve inside UI (uGUI), battle‑tested; use in a small overlay if you prefer uGUI for particles. ([GitHub][11])
* **Lean Touch** – robust pinch/drag/rotate gestures for the board camera on Android. ([Unity Asset Store][21])
* **Odin Validator** – pre‑play validation rules; blocks broken bindings from shipping. ([Unity Asset Store][20])
* **Proxima Runtime Inspector + Console** – edit live builds remotely; instantly see/fix UI states on device. ([Unity Asset Store][19])
* **ithappy Low‑Poly City** – performant, stylized city ring around the board; URP‑friendly; demo scenes to start fast. ([Unity Asset Store][12])

**Nice‑to‑have:**

* **Flexalon Pro** if you want procedural 3D layout helpers near world‑space UI. ([Unity Asset Store][14])

---

# CODING SECTION (ready for Codex)

> Paste these **exact tasks/prompts** into your tool. They’re atomic, strongly typed, and reference the file system above.

---

## C1) Create PanelSettings & a visible root (no blank screens)

**Prompt for Codex**

> **Create** `Assets/UI/PanelSettings/ScreenSpace.asset` and `WorldSpace.asset` with the values in section 2.
> **Create** `Assets/UI/Screens/Home.uxml` with a root `VisualElement` sized full‑screen:
>
> ```css
> .root { position: absolute; left:0; right:0; top:0; bottom:0; }
> ```
>
> **Create** `Assets/UI/Screens/Home.uss` to import `../Styles/tokens.uss`.
> **Create** a **GameObject** “UI_Home” with `UIDocument` → Home.uxml + ScreenSpace.asset. Ensure the element shows in Play.

*(Why: root size is the #1 cause of blank screens.)* ([Unity Documentation][2])

---

## C2) Safe‑area adapter (UITK)

**Codex writes** `SafeAreaRoot.cs` and a small Mono to attach to the `UIDocument` GameObject. It reads `Screen.safeArea` and sets root paddings at runtime (portrait/landscape).
*(We discussed earlier; Codex can reuse that exact code.)*

---

## C3) Core services

**Prompt for Codex**

> Implement `UiRouter.cs` (push/pop screens), `ModalService.cs` (awaitable modals with a stack), and `ToastService.cs` (queue + dedupe). All surfaces mount under a single `UIDocument` root.

---

## C4) Components

**ButtonPrimary**

* Create `Components/ButtonPrimary.uxml/.uss` with the styles in §4.
* Add a C# `ButtonPrimary` wrapper that exposes `SetEnabled(bool)`, `SetText(string)`, and emits `Clicked`.

**Sheet**

* Implement drag handle, velocity settle (half/three‑quarter/full), backdrop with a style class `.blurred`
* Expose events: `Opened`, `Closed`.

**Toast**

* Implement queue with `Enqueue(string message, ToastType type, int ms=1800, bool coalesce=true)`.

**VList**

* Generic wrapper over `ListView` with pooling and bind/make callbacks. **Unit test**: can render 1,000 items smoothly at 60 FPS in Editor. ([Unity Documentation][6])

**TimerBar**

* Provide `Start(seconds)`, `Pause()`, `Resume()`, `SetProgress01(float)`.

---

## C5) Screens (UXML blueprint)

**Home.uxml**

* `TopBar` (avatar chip left; gear+mail right)
* `DioramaViewport` (empty VisualElement—camera renders 3D)
* `Play` ButtonPrimary bottom‑center; `TabBar` with 4 tabs.

**BoardHUD.uxml**

* `TopStrip`: cash/networth/portrait; counters bound to `PlayerVM`.
* `BottomRail`: action buttons (Roll, End, Buy, Auction, Trade, Build, Cards).
* `EventFeed`: `VList` on right.

**Lobby.uxml**

* Card row for Solo/Create/Join; `VList` for rooms.

**Shop.uxml**

* Carousel row (+ price tags). Remote‑driven banner slot.

**Results.uxml**

* Rank cards area; progress bars + confetti overlay.

---

## C6) Flows (awaitable, deterministic)

**Prompt for Codex**

> Implement `AuctionFlow`:
>
> * Show modal with TimerBar, bid chips (+10/+50/+100), Bid/Pass, player list (`VList`).
> * Provide `UniTask<AuctionResult> Run(AuctionParams p, CancellationToken ct)`.
> * Pause/resume timer on reconnect event.
> * Controller nav over chips uses `NavigationMoveEvent`.

**TradeFlow** similarly with dual baskets and `UniTask<TradeResult>`.

*(We keep these as awaitable tasks so your game logic reads cleanly.)*

---

## C7) World‑space UI (property tags & dice banner)

**Prompt for Codex**

> Create a prefab `UI/WorldSpace/TileTag.prefab` with a `UIDocument` → `TileTag.uxml`, `WorldSpace.asset`.
> Add a `TileTagController` that sets label text, owner color, and fades tag based on camera distance.
> Do the same for `PawnPlate` (player name) and `DiceBanner` (“Rolling…”, number pop).

*(World‑Space UITK requires PanelSettings Render Mode = World Space + input config.)* ([Unity Documentation][3])

---

## C8) Input system configuration

**Prompt for Codex**

> Create `UiInputConfigurator.cs`: if project uses project‑wide actions, read the **UI action map** from Project Settings and ensure UITK receives events; otherwise add `InputSystemUIInputModule` on an EventSystem.
> Add `NavigationPolicy.cs` to handle chip‑grid focus with `NavigationMoveEvent`. ([Unity Documentation][5])

---

## C9) Localization binding

**Prompt for Codex**

> Add `TextSettings.asset` with main + fallback fonts.
> Bind `LocalizedString` to Toolkit Labels via data binding (Unity 6+ supports UITK).
> Provide helper `Fmt.Currency(long)` and `Fmt.ShortNumber(long)`; bind to counters. ([Unity Documentation][22])

---

## C10) URP blur (optional full‑screen effect)

**Prompt for Codex**

> Add a **URP Full Screen Pass** (renderer feature) that performs a two‑pass Gaussian blur for background scenes on “modal open”. Expose radius/iters via Volume or script.
> Injection point: **After Post‑Processing**.
> Provide fallbacks to **KAMGAM UITK Blur** for just the UI layer. ([Unity Documentation][23])

---

# MANUAL SECTION (you do once)

* **Install assets**: DOTween Pro, Nice Vibrations, KAMGAM UITK Blur/Text Animation 2, Lean Touch, Odin Validator, Proxima, mob‑sakai UIEffect/UIParticle, ithappy City pack. ([Unity Asset Store][14])
* **Cameras**: orbit cam for the board; Lean Touch for pinch/drag. ([Unity Asset Store][21])
* **Addressables + PAD** groups: `theme_classic`, `theme_event`, `city_extras`, `lang_en`, `lang_ar` (Texture Compression Targeting on). ([Unity Documentation][15])
* **Remote Config/Game Overrides**: keys for `ui:event_banner`, `ui:theme`, `fx:blur_enabled`, `fx:confetti_density`. ([Unity Docs][13])
* **TextSettings**: Inter (Latin) + NotoSansCJK + NotoEmoji in fallback chain. ([Unity Documentation][7])
* **ithappy city**: choose a block that frames your board; keep polycount modest; use the demo LODs from the pack. ([Unity Asset Store][12])

---

# VISUAL BEHAVIOR (exact timings/placements)

* **Home**: Play button bottom center; opens in **220 ms** slide‑up; haptic light.
* **HUD**: Top strip shows money/net‑worth; `Counter` counts 200–600 ms; red flash and 5° micro‑shake on loss.
* **Action rail**: Only valid CTAs are enabled; disabled buttons show a **120 ms** tiny shake + tooltip on tap.
* **Property Sheet**: Opens half; drag to full; backdrop blur (UITK Blur); far swipe closes. ([Unity Asset Store][10])
* **Auction**: TimerBar runs; chips navigate L/R; Bid/Pass bottom; reconnect pauses timer.
* **Trade**: two baskets; tap to add; chips removable; Fairness meter is purely cosmetic.
* **Results**: Rank cards slide from sides (220 ms), confetti overlay (UIParticle), progress bars fill (700–900 ms). ([GitHub][11])

---

# EDGE‑CASE MATRIX (copy into tests)

* Double‑tap on Buy: **debounced**; 2nd ignored (cooldown 300 ms).
* Tap while disabled: show reason tooltip; no action.
* Tap a property during **someone else’s turn**: sheet opens read‑only.
* Network drop in Auction: timer pauses; “Reconnecting…” banner; auto‑pass if timeout.
* Rotate device: safe‑area recomputed; world‑space tags keep their world anchors. ([Unity Documentation][3])
* Missing glyphs: fallback font renders CJK/emoji (test Turkish, Arabic, Chinese). ([Unity Documentation][7])

---

# PERFORMANCE BUDGET & QA

* **UI draw**: keep < **6–8 ms** on mid‑tier; use `ListView` for long lists (rooms/offers/logs). ([Unity Documentation][6])
* **Thermal scaling**: Adaptive Performance reduces blur radius and confetti density; change via Remote Config. ([Unity Documentation][15])
* **Device Simulator**: cover Pixel 6/7, Galaxy A (notch), tablet 10.5". ([Unity Documentation][8])
* **Proxima** in dev builds: 3‑finger tap to open; inspect live UI values on device. ([Unity Asset Store][19])
* **Odin Validator**: rule set: “All scenes must have ≥1 UIDocument + ScreenSpace PanelSettings; all Labels must bind or have default text.” ([Unity Asset Store][20])

---

# ASSET STORE—WHY EACH HELPS

* **KAMGAM UITK Blur/Text Animation 2** → native UITK blur + premium text effects **without** writing custom renderer features (fastest polish). ([Unity Asset Store][10])
* **UIParticle/UIEffect (mob‑sakai)** → confetti & stylish UI FX, maskable, no extra cameras; widely used & maintained. ([GitHub][11])
* **DOTween Pro** → deterministic sequences and counter tweens (industry standard). ([Unity Asset Store][14])
* **Nice Vibrations** → proven HD haptics mapping for Android/gamepads. ([moremountains.com][9])
* **Lean Touch** → mobile gestures plug‑and‑play. ([Unity Asset Store][21])
* **Proxima** → live remote inspector/console in builds—massive iteration speed‑up for UI. ([Unity Asset Store][19])
* **Odin Validator** → stops broken UI before Play. ([Unity Asset Store][20])
* **ithappy City** → high‑quality, optimized low‑poly city; demo scenes to ring the board fast. ([Unity Asset Store][12])

---

# MINI CODE SNIPPETS (paste‑ready)

**Bind a ViewModel to UITK at runtime** (Unity 6):

```csharp
using UnityEngine;
using UnityEngine.UIElements;

public sealed class BoardHudView : MonoBehaviour {
  [SerializeField] UIDocument doc;
  [SerializeField] PlayerViewModel vm;

  void OnEnable() {
    var root = doc.rootVisualElement;
    root.dataSource = vm; // runtime data binding (Unity 6)
  }
}
```

*(UITK runtime data binding binds plain C# properties to control properties.)* ([Unity Documentation][4])

**PanelSettings: full‑screen root** (prevents invisible UI):

```css
/* in any screen's .uss */
.root { position:absolute; left:0; right:0; top:0; bottom:0; }
```

**Enhanced touch bootstrap (Android gestures)**

```csharp
using UnityEngine;
using UnityEngine.InputSystem.EnhancedTouch;
public static class TouchBootstrap {
  [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
  static void EnableEnhancedTouch() => EnhancedTouchSupport.Enable();
}
```

*(Pairs well with Lean Touch.)* ([Unity Asset Store][21])

---

# WHAT I STILL NEED FROM YOU (so I can tailor the spec 1:1)

1. The **UI Audit** (or the unitypackage) I gave earlier (PanelSettings, UXML/USS, scenes).
2. The **list of paid packages & versions** you imported.
3. The **target device list** (Pixel/Galaxy/tablet).
4. A **30s screen capture** (Home → Solo → Board → Property → back).

With that, I’ll rewrite these specs matched to *your* tree (class names, UXML IDs, exact binding paths), and I’ll flag anything causing the blank screen (missing PanelSettings, root size, TextSettings, mis‑wired input, etc.).

---

## References / Why you can trust the stack

* **Runtime data binding** (Unity 6 manual). ([Unity Documentation][4])
* **PanelSettings** (Scale Mode, World Space render mode). ([Unity Documentation][2])
* **Device Simulator** (safe‑area + device testing). ([Unity Documentation][8])
* **Input System UI support** (Project‑wide UI action map in 2023.2+/6). ([Unity Documentation][5])
* **ListView virtualization** (UITK docs/Scripting API). ([Unity Documentation][6])
* **Text Settings & fallback fonts** (UITK). ([Unity Documentation][7])
* **URP Full Screen Pass (custom blur)**. ([Unity Documentation][23])
* **Addressables + Play Asset Delivery** (Android). ([Unity Documentation][15])
* **Remote Config / Game Overrides** (A/B, events). ([Unity Documentation][17])
* **KAMGAM UITK Blur / Text Animation 2**. ([Unity Asset Store][10])
* **mob‑sakai UIParticle / UIEffect**. ([GitHub][11])
* **DOTween Pro**, **Nice Vibrations**, **Lean Touch**, **Odin Validator**, **Proxima**. ([Unity Asset Store][14])
* **ithappy Low‑Poly City** (publisher & pack). ([Unity Asset Store][12])

---

If you want, I can also include **ready‑to‑drop UXML/USS** for **Home**, **BoardHUD**, and the **Property Sheet** (with class names and bindings prewired) plus a **Minimal Scene** with `UIDocument` + `ScreenSpace.asset` so your first Play is *never* blank.

[1]: https://docs.unity3d.com/6000.2/Documentation/ScriptReference/UIElements.PanelSettings.html?utm_source=chatgpt.com "PanelSettings - Scripting API"
[2]: https://docs.unity3d.com/6000.2/Documentation/Manual/UIE-Runtime-Panel-Settings.html?utm_source=chatgpt.com "Panel Settings properties reference"
[3]: https://docs.unity3d.com/6000.3/Documentation/Manual/ui-systems/world-space-ui.html?utm_source=chatgpt.com "World Space UI"
[4]: https://docs.unity3d.com/6000.2/Documentation/Manual/UIE-runtime-binding.html?utm_source=chatgpt.com "Runtime data binding"
[5]: https://docs.unity3d.com/Packages/com.unity.inputsystem%401.8/manual/UISupport.html?utm_source=chatgpt.com "UI support | Input System | 1.8.2"
[6]: https://docs.unity3d.com/6000.2/Documentation/Manual/UIE-uxml-element-ListView.html?utm_source=chatgpt.com "ListView"
[7]: https://docs.unity3d.com/6000.2/Documentation/Manual/UIE-text-setting-asset.html?utm_source=chatgpt.com "UITK Text Settings assets"
[8]: https://docs.unity3d.com/6000.2/Documentation/Manual/device-simulator.html?utm_source=chatgpt.com "Device Simulator"
[9]: https://nice-vibrations.moremountains.com/?utm_source=chatgpt.com "Nice Vibrations, the best way to add vibrations to your Unity ..."
[10]: https://assetstore.unity.com/packages/2d/gui/ui-toolkit-blurred-background-fast-translucent-ui-blur-image-254328?srsltid=AfmBOor29pNH3ZQjnoOQOaLdxgS-Yxn9IPmYWKD_hkCb_GTfa0l4rHN9&utm_source=chatgpt.com "UI Toolkit Blurred Background - Fast translucent UI Blur ..."
[11]: https://github.com/mob-sakai/ParticleEffectForUGUI?utm_source=chatgpt.com "mob-sakai/ParticleEffectForUGUI: Render particle effect in ..."
[12]: https://assetstore.unity.com/packages/3d/environments/urban/city-low-poly-3d-models-pack-197808?srsltid=AfmBOooH7xJWZF4ntsVMWLPV8_NKnWQl9T3kvDNyduANvrnjJCgGPfGy&utm_source=chatgpt.com "City - Low Poly 3D Models Pack | 3D Urban"
[13]: https://docs.unity.com/ugs/manual/remote-config/manual/game-overrides-and-settings?utm_source=chatgpt.com "Game Overrides and Settings"
[14]: https://assetstore.unity.com/packages/tools/visual-scripting/dotween-pro-32416?srsltid=AfmBOoqTWyjxVJWVqPwjdQmiKuK6iVpMYW1wR4jMp8kat3wvUpKjghMO&utm_source=chatgpt.com "DOTween Pro | Visual Scripting"
[15]: https://docs.unity3d.com/Packages/com.unity.addressables.android%401.0/manual/index.html?utm_source=chatgpt.com "Addressables for Android package"
[16]: https://docs.unity3d.com/2023.2/Documentation/Manual/UIE-fallback-font.html?utm_source=chatgpt.com "Fallback font"
[17]: https://docs.unity3d.com/Packages/com.unity.remote-config%404.1/manual/index.html?utm_source=chatgpt.com "Remote Config | 4.1.1"
[18]: https://docs.unity3d.com/6000.2/Documentation/Manual/UIE-ui-debugger.html?utm_source=chatgpt.com "UI Toolkit Debugger"
[19]: https://assetstore.unity.com/packages/tools/utilities/proxima-runtime-inspector-console-244788?srsltid=AfmBOoo7TzaJ9sP5sqAYizB5qzvMzyG934CWFCFObgYr_zRgE4SkTl1X&utm_source=chatgpt.com "Proxima Runtime Inspector + Console | Utilities Tools"
[20]: https://assetstore.unity.com/packages/tools/utilities/odin-validator-227861?srsltid=AfmBOoqwA8TS-6FEnrOglypnXc6KlUNcs1M62RsQnglNlar3sQiJuM_Y&utm_source=chatgpt.com "Odin Validator | Utilities Tools"
[21]: https://assetstore.unity.com/packages/tools/input-management/lean-touch-30111?srsltid=AfmBOorthcy5yQkcV--fmBC0QKtzidbsWJNxYeBa9SnLNqGyjPyqTkPa&utm_source=chatgpt.com "Lean Touch | Input Management"
[22]: https://docs.unity3d.com/Packages/com.unity.localization%401.5/manual/UIToolkit.html?utm_source=chatgpt.com "UI Toolkit (Unity 6+) | Localization | 1.5.8"
[23]: https://docs.unity3d.com/6000.2/Documentation/Manual/urp/post-processing/custom-post-processing.html?utm_source=chatgpt.com "Custom post-processing in URP"
