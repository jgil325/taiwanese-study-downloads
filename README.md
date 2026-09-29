# Taiwanese Study for Mac

A desktop analyzer and practice trainer for Taiwanese poker. Calculations and saved hands stay on your Mac. No server, account, or internet connection is needed after downloading.

## Download

**[Download Taiwanese Study 0.2.1 for Mac](https://github.com/jgil325/taiwanese-study-downloads/releases/download/v0.2.1/Taiwanese-Study-Mac.dmg)**

One universal download for **Apple Silicon and Intel**, requiring **macOS 13.3 or later**. [Release notes and checksum](https://github.com/jgil325/taiwanese-study-downloads/releases/tag/v0.2.1).

1. Open the DMG.
2. Drag **Taiwanese Study** into **Applications**.
3. Eject the DMG and open the app from Applications.

This free beta is **not Apple-notarized**. If macOS blocks first launch, attempt to open it, then go to **System Settings → Privacy & Security → Open Anyway** for Taiwanese Study. See [Apple's instructions](https://support.apple.com/en-us/102445).

The first launch unpacks the included experimental 262,144-deal strategy. Allow about **1 GB of free space**, plus room for additional training. Choose it in the **Opponent strategy** menu; the uniform baseline is also available.

## Inside the app

- **Practice:** arrange seven cards with dragging, clicking, or keyboard shortcuts. Commit before seeing EV loss, standard errors, and the top five estimated settings. Unresolved leaders and random-setting fallback are displayed alongside results.
- **Analyze:** set your seven cards, then compare all 105 settings against a frozen opponent strategy with Monte Carlo estimates.
- **Study:** save decisions and notes, compare fixed matchups, and customize two-color or four-color decks.
- **Strategy reports:** chart Top ranks, a complete 13×13 Middle matrix, and Bottom rank/suit patterns across random hands. Compare overall frequency with frequency when a hand type is available, inspect standard errors, and export CSV/JSON to Downloads. Choose solver recommendations or the frozen opponent policy's own setting frequencies.
- **Train:** run local shared-strategy self-play, pause with a resumable checkpoint, and audit policies independently.

**Keyboard controls in Practice and Analyze:** the first card selects automatically. Press **T / M / B** to place it and advance to the next unplaced card. Press **Enter** when all seven cards are placed to submit. A full row keeps the current selection; New hand and Clear setting restart at the first card.

“Training deals” counts the past self-play rounds used to train a policy. “Sampling budget” controls fresh scenarios for the current EV calculation. More samples reduce estimation uncertainty; they do not train the opponent policy. Study estimates require at least **100,000** samples; **Standard uses 250,000 by default**, and Deep uses **1,000,000**. More samples do not guarantee a resolved leader. The included strategy is **experimental, not certified GTO**, and still uses random settings for most unseen opponent hand classes.

## Data and updates

Your own models and saved hands live in:

```text
~/Library/Application Support/com.jgil325.taiwanesestudy/
```

Quit the app before backing up this folder. To update, download a newer installer and replace the app in Applications; your local data remains. This release has no automatic updater.

This repository distributes the app and installation notes. The source repository is private. The installer includes dependency notices and their upstream source links.

The packaged app was tested on Apple Silicon. Intel is included in the universal binary but has not yet been tested on Intel hardware.
