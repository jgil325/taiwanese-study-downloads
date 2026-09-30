# Taiwanese Study for Mac

A private desktop analyzer and practice trainer for Taiwanese poker. Calculations and saved hands stay on your Mac. No server, account, or internet connection is needed after downloading.

## Download

**[Download Taiwanese Study 0.3.0 for Mac](https://github.com/jgil325/taiwanese-study-downloads/releases/download/v0.3.0/Taiwanese-Study-Mac.dmg)**

One universal download for **Apple Silicon and Intel**, requiring **macOS 13.3 or later**. [Release notes and checksum](https://github.com/jgil325/taiwanese-study-downloads/releases/tag/v0.3.0). [Measured model quality and sampling results](https://github.com/jgil325/taiwanese-study-downloads/releases/download/v0.3.0/MODEL_REPORT.md).

1. Open the DMG.
2. Drag **Taiwanese Study** into **Applications**.
3. Eject the DMG and open the app from Applications.

This free beta is **not Apple-notarized**. If macOS blocks first launch, attempt to open it, then go to **System Settings → Privacy & Security → Open Anyway** for Taiwanese Study. See [Apple's instructions](https://support.apple.com/en-us/102445).

First launch unpacks four compatible experimental strategies. Allow **6 GB of free space**, plus room for your own training; **8 GB RAM or more** is recommended. The blind-joker table can use several GB of RAM. Its bundled compact average is analysis-only; choose a uniform baseline or a resumable checkpoint to start training.

## Four game modes

| Mode | Deck | Information before setting |
|---|---|---|
| Classic | 52 ordinary cards | Your seven cards |
| Jokers | 52 ordinary cards + two wildcards | Your seven cards |
| Revealed flops | 52 ordinary cards | Your seven cards and both three-card flops |
| Flops + jokers | 54 cards | Your seven cards, both flops, and exposed discarded wildcards |

Every mode keeps Top 1 / Middle 2 / Bottom 4, two boards from one deck, row points 1/2/3, ties worth zero, and an additional eight points for six outright wins against an opponent. Top/Middle use best-five Hold'em construction; Bottom uses exactly two private and three board cards.

Both wildcards show **W** in warm plum purple. They automatically become the best legal ordinary card for each row/board; five of a kind is excluded. They never remain on a board: skipped wildcards are permanently removed and replaced with ordinary cards. In the combined mode, discarded wildcards encountered while dealing the flops are public before setting. Later replacements are unknown when setting. Settings are committed simultaneously before turns/rivers.

Each mode has its own frozen strategy family. The three new families each include a ten-million-deal artifact and independent audits; all remain **experimental, not certified GTO**. Monte Carlo SE does not include strategic model approximation error. See the measured report for actual attack gains, uncertainty, repeated seeds, and resource use.

## Inside the app

- **Practice:** arrange seven cards with dragging, clicking, or keyboard shortcuts. Commit before seeing EV loss, standard errors, and five leading settings. Unresolved choices get neutral feedback.
- **Analyze:** enter a hand and, when applicable, both public flops/discards, then compare all 105 settings against a compatible frozen strategy.
- **Study:** save decisions with their full public context, compare fixed matchups, inspect ordinary example runouts and wildcard roles, and customize foreground/background two-color or four-color decks.
- **Strategy reports:** chart Top ranks, a 13×13 natural Middle matrix plus wildcard bins, and Bottom rank/suit/joker patterns. Choose whole-game, fixed-public-state, or texture scopes; compare overall frequency with frequency when available. CSV/JSON exports retain context, model, and uncertainty.
- **Train:** run local shared-strategy self-play, pause with a resumable checkpoint, and audit policies independently. Compact blind-mode averages are clearly analysis-only.

**Keyboard controls in Practice and Analyze:** the first card selects automatically. Press **T / M / B** to place it and advance to the next unplaced card. Press **Enter** when all seven cards are placed. A full row keeps the current selection; New hand and Clear restart at the first card.

“Training deals” counts past self-play rounds used to train a policy. “Sampling budget” controls fresh compatible opponent draws for the current EV calculation; all 105 settings share each opponent, setting, and board pair. More samples improve estimation precision without training the policy. Quick uses **100,000** draws, Standard **250,000**, and Deep **1,000,000**. When reusing an opponent for multiple board pairs, SE is calculated from independent opponent-group means.

## Data and updates

Your models and saved hands live in:

```text
~/Library/Application Support/com.jgil325.taiwanesestudy/
```

Quit the app before backing up this folder. Updating the app in Applications preserves this workspace. Original Classic models and saved hands remain readable. This release has no automatic updater.

This repository distributes installers and notes. The source repository is private. The installer includes dependency notices and upstream source links. The packaged app was tested on Apple Silicon; Intel is included but has not yet been tested on Intel hardware.
