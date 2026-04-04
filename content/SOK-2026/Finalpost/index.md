---
title: "Project Wrap-Up: XMPP Bookmark Sync for Falkon"
date: 2026-04-5
tags: [KDE, OpenSource, C++, XMPP]
draft: false
---
## 1. Project Overview

The goal of this project was to provide **Falkon** users with a decentralized, private, and secure way to synchronize bookmarks using the **XMPP** protocol, specifically leveraging **PubSub (XEP-0060)** and **PEP (XEP-0163)**.

### Key Achievements:

- **MVP Completion:** A functional C++ plugin that authenticates, creates server-side storage nodes, and handles data transfer.
    
- **Upstream Contribution:** Submitted [**MR #750**](https://invent.kde.org/libraries/qxmpp/-/merge_requests/750) to **QXmpp** to implement OMEMO client support for encrypted bookmarks.
    
- **Standalone Tooling:** Developed a standalone tool for **[Chawan](https://invent.kde.org/shibe/pubsub)**, a CLI utility for raw XMPP PubSub testing.
    

---

## 2. Technical Journey & Architecture

### The Evolution of Sync Logic

My approach to synchronization evolved through three distinct phases:

1. **Phase 1: Automatic Trigger (The "Flicker" Issue)** Initially, I attempted to sync on every bookmark change. This led to **Race Conditions** where a "Download" would trigger before a previous "Upload" had finished indexing on the server, causing local bookmarks to disappear or revert.
    
2. **Phase 2: Manual Control (The MVP)** To stabilize the state, I pivoted to a **Manual Sync** model with three explicit actions: **Upload**, **Download**, and **Merge**. This allowed for clear debugging of the XML stanzas.
    
3. **Phase 3: The CRDT Pivot (Current State)** Following mentor advice of Benson Muite the next step would be using  **CRDT (Conflict-free Replicated Data Type)**.While the current MVP uses a manual merge, moving toward LWW (Last-Write-Wins) Element Sets is the long-term plan to allow seamless, conflict-free merges mathematically
    

---

## 3. Challenges and "Lessons Learned"

### What Failed (and Why)

- **Deletion Handling:** Simply deleting a bookmark locally caused it to "re-appear" during the next sync because the client thought it was "new data" from the server.
    
- **ID Instability:** Using temporary database IDs caused duplicate entries. I solved this by implementing a **stable hashing algorithm** (URL + Title) for bookmark IDs.


### What Worked

- **QXmpp Integration:** The library proved robust for handling complex SASL authentication (SCRAM-SHA-1) and stream management.
    
- **Modular Debugging:** Building the logic in the **Chawan** sandbox before moving into the Falkon source tree saved dozens of hours in compile-time waiting.
    

---

## 4. Current Status & Remaining Work

|Feature|Status|Notes|
|---|---|---|
|**Auth & Connection**|**100%**|Stable on `xmpp.jp` and `hackerheaven.org`.|
|**Node Management**|**100%**|Automatically creates `org.kde.falkon:bookmarks`.|
|**Basic Sync**|**90%**|Works well for additions and updates.|
|**Deletions**|**In Progress**|Implementing **Tombstones** to ensure `retract` stanzas fire correctly.|
|**Encryption**|**In Progress**|Integrated OMEMO via QXmpp MR; needs final testing in Falkon UI.|

Export to Sheets

---

## 5. Final "Fix-It" Roadmap

Before the final code submission, I am focusing on:

1. **Tombstone Persistence:** Ensuring the local database remembers a deletion until the XMPP server acknowledges the `retract` command.
    
2. **Conflict UI:** Adding a "Last Sync" timestamp to the Falkon status bar to give users visual feedback.
    
3. **Code Cleanup:** Merging the `manualSync` and `newplugin` branches into a final, clean `master`-ready state for KDE review.
    

---

### Mentor Acknowledgments

Special thanks to **Schimon Jehudah** for the deep dives into XMPP architecture and **Benson Muite** for the architectural guidance on CRDTs and manual-first sync strategies. including **Juraj Oravec**