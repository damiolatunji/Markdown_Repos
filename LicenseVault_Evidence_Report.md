# LicenseVault: Mobile Wallet-to-Decryption Screenshot Evidence Report

**Scope:** screenshot analysis and evidence extraction only. Nothing was edited, deployed, or run against the project. No fixes are proposed.
**Prepared:** Friday 16 October 2026 (Lagos, UTC+01:00)
**Evidence set:** 7 screenshots supplied by the user.

**Evidence labels used throughout**

- **Observed:** directly visible in a screenshot.
- **Inferred:** supported by several screenshots or by verified repo content, but not directly shown.
- **Unknown:** cannot be determined from what was supplied.
- **[unclear]:** text that is cut off or unreadable. Nothing has been invented to fill it.

---

## A. Environment and context

| Item | Value | Status |
|---|---|---|
| Website | `licensevault.vercel.app` (address bar shows `licensevault.vercel.ap`, truncated) | Observed |
| Repository | https://github.com/Temmygabriel/LicenseVault (public, `main`, 29 commits per GitHub page) | Observed (fetched) |
| Device | Android phone. Status bar, 3-button Android nav bar, and a battery of 52–53% (12:5x am) and 32% (6:33 pm) are visible. Model unknown. | Observed / Unknown |
| Browser | **Rabby Wallet's built-in dApp browser on mobile.** Rabby's "Total Balance" home screen is visible behind a bottom sheet. Rabby's pill toolbar (star, •••, ✕) sits at the top. The bottom bar has a home icon, URL pill with a tab counter, and a round chain icon with a "Sto…" badge. | Observed |
| Tab counter | **16** in S6/S7 (6:33 pm); **17** in S1–S5 (12:52–12:53 am) | Observed |
| Wallet | Rabby (Android) | Observed |
| Network (as printed by the page) | `Story Aeneid · Chain 1315` | Observed in S1, S5, S6 |
| Network (as reported by wallet) | Not shown. Only a truncated "Sto" badge on Rabby's chain icon, consistent with Story but not proof. | Unknown / weak inference |
| Connected account | `0x1E97…7559` (abbreviated; the full address is never visible) | Observed in S2, S5, S6, S7. Partially cut off in S1 [unclear] |
| License token ID in use | `73227` (pre-filled in the input; "GATE TOKEN #73227") | Observed |
| Wallet balance (as displayed) | `1 testnet token` | Observed in S2, S7 |
| CDR protocol read fee (as displayed) | `0 testnet token` | Observed in S2, S7 |
| Dates | No date appears in any screenshot. The 12:52/12:53 am captures are about 10 minutes before the 01:04 message time, so 16 Oct. The 6:33 pm captures are most likely 15 Oct. | Inferred |

**Observation about the capture order.** The screenshots were supplied newest first: 12:53 am, then four at 12:52 am, then two at 6:33 pm. Within the 12:52 group the timestamps are identical to the minute, so their order is inferred from scroll position and content (see Section C).

---

## B. Screenshot-by-screenshot evidence

Numbering follows the order supplied (S1 = first image supplied). A suggested chronological order is in Section C.

### S1: 12:53 am · battery 52% · tab count 17

**Visible UI state:** top of the access docket scrolled down slightly, in an error state.

**Exact text:**
- `ACCESS STATUS`, with a red-outlined badge reading `● VERIFICATION ERROR`
- `NETWORK` / `Story Aeneid · Chain 1315`
- "Every result comes from the live protocol; errors never become a license denial."
- Button: `RETRY VERIFICATION` (solid terracotta, full width)
- "We could not establish whether this wallet holds the license. That is a failure on our side of the wire — it is not a statement about your rights."
- `LICENSE TOKEN ID`, input value `73227`
- Helper: "Check a token you already own for this asset. The field starts with the published demo token #73227 so judges can reproduce the recorded proof. If you do not have a license, use **Get demo license** after the check is refused."
- A section heading below is cut off at the bottom edge, probably `ACCESS SEQUENCE` [unclear].
- Connected address at the top is partially hidden by Rabby's toolbar, reading approximately `0x1E97…7559` [unclear].

**Relevant values:** chain 1315; token 73227; no transaction hash, request ID, HTTP status, or diagnostic label visible.

**Establishes:**
- At 12:53 am the page was in a VERIFICATION ERROR state and offered `RETRY VERIFICATION`.
- The copy explicitly says this is not a licensing denial.
- The `GET DEMO LICENSE` button is not visible in this state. The helper text ties it to "after the check is refused".

**Does not establish:** what caused the error, which request failed, whether Retry was tapped afterwards, or whether the error came from the license check or the protected read.

---

### S2: 12:52 am · battery 53% · tab count 17

**Visible UI state:** bottom of the card, in a "waiting for protected read" state.

**Exact text:**
- `…opens the sealed bytes.` (end of step 04 text, clipped by the top overlay) / `WAITING`
- Step `05` **Verify content**: "The browser checks the recorded SHA-256 before reveal." / `WAITING`
- "Asking the live Aeneid license condition. This is a real chain result, not a visual simulation."
- Callout with a terracotta bar and dot: **"Waiting for the protected read"**. "Your wallet may ask for transaction approval. Then the SDK waits for validator responses. This can take time; no progress percentage is invented."
- `WALLET BALANCE` `1 testnet token`
- `CDR PROTOCOL READ FEE` `0 testnet token`
- "Network gas is separate from the CDR protocol fee. These values are queried live before a protected read; zero protocol fee does not mean zero gas."
- `CONNECTED` `0x1E97…7559` / `GATE TOKEN` `#73227`

**Establishes:**
- The UI showed an in-progress protected-read state with steps 04 and 05 still WAITING.
- Fee and balance values were displayed.

**Does not establish:**
- Whether a Rabby approval popup appeared, was approved, or was rejected.
- Whether any transaction was submitted. No hash is shown.
- Which checkpoint (02 or 03) was active, because steps 01–03 are off-screen.
- Whether the state was live or stalled (a still image cannot show animation).

---

### S3: 12:52 am · battery 53% · tab count 17

**Visible UI state:** same lower region as S2, but showing a failure banner and an optional test.

**Exact text:**
- Step `05` Verify content, "The browser checks the recorded SHA-256 before reveal." / `WAITING`
- "The on-chain condition refused this wallet for the demo token being checked. Get a demo license or explicitly test a locked read."
- Banner (red bar): `ACTION NEEDS ATTENTION`. "The network could not establish a reliable result. This is not a licensing decision; try again later."
- Expanded disclosure: `▼ Diagnostic label` → **`INFRASTRUCTURE_FAILURE`**
- `OPTIONAL PROOF TEST` / "Show the gate refusing access"
- "This sends a real CDR read using the demo token ID. If your wallet does not own it, the transaction should be refused. A reverted transaction still spends network gas."
- Outlined button: `TEST LOCKED READ · GAS REQUIRED` (two lines)
- `CONNECTED` / `GATE TOKEN` labels at the bottom edge (values cut off)

**Establishes:**
- **The refusal sentence and the infrastructure-failure banner are on screen together.** One says the on-chain condition refused the wallet. The other says no reliable result was obtained and it is not a licensing decision.
- The only diagnostic detail the UI exposes is the label `INFRASTRUCTURE_FAILURE`.
- The "Test locked read" button appears idle and enabled-looking.

**Does not establish:** which attempt produced the banner, what the underlying error text was, or whether "Test locked read" had been tapped.

---

### S4: 12:52 am · battery 53% · tab count 17

**Visible UI state:** checkpoint list ("5 CHECKPOINTS").

**Exact text:**
- Heading clipped: `…readable content` [unclear start]
- `01` **Connect wallet** (green ✓): "Your address; no signature on connection." / `COMPLETE`
- `02` **Verify license** (red): "Read the live license condition on Aeneid." / `BLOCKED`
- `03` **Sign CDR read**: "Your wallet submits the protected read; gas may apply." / `WAITING`
- `04` **Recover & decrypt**: "The SDK collects validator partials and opens the sealed bytes." / `WAITING`
- `05` **Verify content**: "The browser checks the recorded SHA-256 before reveal." / `WAITING`
- "The on-chain condition refused this wallet for the demo token being checked. Get a demo license or explicitly test a locked read."
- The top of the red banner bar begins at the bottom edge.

**Establishes:**
- Wallet connection is marked complete.
- Step 02 is BLOCKED, and steps 03–05 are WAITING.
- No step is shown as complete past connection.
- The refusal text appears at the bottom of the checkpoint list, directly above the failure banner seen in S3.

**Does not establish:** why step 02 is blocked (refusal vs error). The sentence below the list says "refused", but S3 shows the infrastructure banner right under it.

---

### S5: 12:52 am · battery 53% · tab count 17

**Visible UI state:** top of the docket, in a no-license state.

**Exact text:**
- `LICENSE HOLDER` `0x1E97…7559`
- `ACCESS STATUS` `● NO LICENSE` (red-outlined badge)
- `NETWORK` `Story Aeneid · Chain 1315`
- "The demo license is minted by your wallet. Its recorded minting fee is zero; network gas still applies."
- Button: `GET DEMO LICENSE` (solid brown, full width, looks enabled)
- "The license condition refused the read. This is a real answer from the chain, not a guess — access requires a Commercial Use license for this asset."
- `LICENSE TOKEN ID` input `73227`, helper text (cut off after "If you do not have a")

**Establishes:**
- At 12:52 am the status badge read NO LICENSE for the connected wallet.
- The "Commercial Use license" wording is the copy attached to this refusal state.
- The `LICENSE HOLDER` field shows the connected wallet's address even though the status says NO LICENSE. This is a labeling observation, not an error.

**Does not establish:** whether the refusal came from the free license check or from a protected read, or whether `GET DEMO LICENSE` was tapped.

---

### S6: 6:33 pm · battery 32% · tab count 16

**Visible UI state:** same top-of-docket view as S5, at an earlier session.

**Exact text:** same as S5 for status (`● NO LICENSE`), network, minting-fee sentence, `GET DEMO LICENSE`, refusal sentence, and token field `73227`. The helper text is fully visible, and the address `0x1E97…7559` is partially visible at the top [partially clipped].

**Establishes:** the NO LICENSE state was reproduced in a separate session (different time of day, battery, and tab count).

**Does not establish:** anything about gas, tx, or minting.

---

### S7: 6:33 pm · battery 32% · tab count 16

**Visible UI state:** bottom of the card in an error state.

**Exact text:**
- Step `05` Verify content / `WAITING`
- "The request did not produce a trustworthy result. This is not a license denial; resolve the error and retry."
- Banner: `ACTION NEEDS ATTENTION`. "The network could not establish a reliable result. This is not a licensing decision; try again later."
- `▶ Diagnostic label` (**collapsed**, contents not visible)
- `WALLET BALANCE` `1 testnet token`
- `CDR PROTOCOL READ FEE` `0 testnet token`
- The same note about network gas
- `CONNECTED` `0x1E97…7559` / `GATE TOKEN` `#73227`

**Establishes:**
- The same infrastructure-failure banner already seen in S3 occurred in the 6:33 pm session too.
- The sentence above the banner here reads as a *technical failure* ("not a license denial"). In S3 the sentence above the banner reads as a *refusal*.
- **There is no "Optional proof test / Test locked read" block in this capture.** It appears in S3 but not here, between the banner and the balance rows.

**Does not establish:** the diagnostic label for this occurrence (collapsed), the cause, or its order relative to S6.

---

## C. Reconstructed chronological timeline

**Ordering basis.** S6/S7 (6:33 pm) precede S1–S5 (12:52–12:53 am). The order inside each group is inferred. For the 12:52 group I used a top-to-bottom scroll reading (S5 → S4 → S3 → S2), which also matches the reverse of the supplied order, then S1 at 12:53.

| # | Event | Label | Source |
|---|---|---|---|
| 1 | The page is open in Rabby's in-app browser, with the connected wallet shown as `0x1E97…7559`. | Observed | S6, S7 |
| 2 | Status reads NO LICENSE and offers GET DEMO LICENSE. The token field is pre-filled with the published demo token 73227. | Observed | S6 |
| 3 | A failure banner (`ACTION NEEDS ATTENTION`, "network could not establish a reliable result") is shown in the same session. | Observed | S7 |
| 4 | The relative order of events 2 and 3 within the 6:33 pm session is unknown. They could be sequential or simultaneous. | Unknown | S6/S7 |
| 5 | Between 6:33 pm and 12:52 am the tab count changes from 16 to 17 and the battery from 32% to 53%, so the page was reopened or reloaded (maybe a new deployment). | Inferred | Status bars |
| 6 | At 12:52 am the page shows `ACCESS STATUS: NO LICENSE`, the refusal copy, GET DEMO LICENSE, and token 73227. | Observed | S5 |
| 7 | Checkpoints show Connect wallet ✓ COMPLETE and Verify license **BLOCKED**, with steps 03–05 WAITING. | Observed | S4 |
| 8 | Below the list, the "on-chain condition refused this wallet" sentence and the `INFRASTRUCTURE_FAILURE` banner are displayed together, with an idle "Test locked read · gas required" button. | Observed | S3 |
| 9 | A protected-read wait state is displayed: "Asking the live Aeneid license condition… Waiting for the protected read… Your wallet may ask for transaction approval." | Observed | S2 |
| 10 | Whether event 9 was triggered by tapping "Test locked read" is **not shown**. | Unknown | none |
| 11 | Whether Rabby displayed a transaction/signature prompt is **not shown**. | Unknown | none |
| 12 | Whether any transaction was submitted or confirmed is **not shown**. No hash appears anywhere. | Unknown | none |
| 13 | At 12:53 am the status badge reads **VERIFICATION ERROR** with `RETRY VERIFICATION`. | Observed | S1 |
| 14 | No screenshot shows `GET DEMO LICENSE` being tapped, a mint prompt, a mint transaction, a new token ID, decrypted content, or a SHA-256 match. | Observed (absence) | all |

**Uncertainty to be clear about.** The 12:52 ordering is a best reading, not a fact. If these were captured in a different order, events 8–9 could swap. In particular, S3 shows the banner *before* the S2 wait in my ordering, but the reverse is possible.

---

## D. Error-message comparison

| # | Message (verbatim) | Where | Category | What the screenshot establishes |
|---|---|---|---|---|
| D1 | `NO LICENSE` + "The license condition refused the read. This is a real answer from the chain, not a guess — access requires a Commercial Use license for this asset." | S5, S6 | **Licensing refusal copy** | The UI is presenting a refusal for the connected wallet. It does not show which request produced it. |
| D2 | "The on-chain condition refused this wallet for the demo token being checked. Get a demo license or explicitly test a locked read." | S3, S4 | **Licensing refusal copy** | Same refusal category as D1. In S4 it accompanies step 02 BLOCKED. |
| D3 | `ACTION NEEDS ATTENTION`: "The network could not establish a reliable result. This is not a licensing decision; try again later." + label `INFRASTRUCTURE_FAILURE` | S3 (label visible), S7 (label collapsed) | **Technical failure** | The app is classing an attempt as "no reliable result". The underlying error text is not shown. |
| D4 | "The request did not produce a trustworthy result. This is not a license denial; resolve the error and retry." | S7 | Technical failure | Lead-in copy for D3 in the 6:33 pm session. |
| D5 | `VERIFICATION ERROR` + "We could not establish whether this wallet holds the license. That is a failure on our side of the wire — it is not a statement about your rights." / "Every result comes from the live protocol; errors never become a license denial." | S1 | Technical failure | At 12:53 am the top-level status was error, not no-license. |
| D6 | Step `02 Verify license` = `BLOCKED` (red) | S4 | Ambiguous | "Blocked" does not say whether by refusal or by error. |

**Are D1/D2 and D3–D5 the same event?** The screenshots do not settle this.

- **They appear simultaneously in S3** (D2 directly above D3) and in the same session in S6/S7 (D1 vs D3). A single event would not normally be both "refused" and "not a licensing decision".
- **Most consistent readings (inferred, unconfirmed):** (a) two separate checks (a refusal from one, an error from another) are being displayed together; (b) one display is stale; or (c) a refusal was misclassified as a failure. The screenshots cannot separate these.
- The wording in S1 and S7 is explicit that errors should not be read as license denials. S3 contradicts that intent by showing the refusal sentence directly above the error banner.

---

## E. Workflow stage table

| Stage | Status | Evidence |
|---|---|---|
| Wallet connection | **Confirmed successful** (UI-level) | S4 "Connect wallet ✓ COMPLETE"; connected address shown in S2, S5, S6, S7. No signature was involved ("no signature on connection"). |
| Chain detection | **Ambiguous** | Page prints "Story Aeneid · Chain 1315", but nothing shows it was read from the wallet. No wrong-network message appears anywhere. The "Sto…" badge on Rabby's chain icon is truncated. |
| License lookup | **Ambiguous** | Result shown as NO LICENSE (S5, S6) and, in other places, as an error (S1, S3, S7). Step 02 BLOCKED (S4). |
| Ownership verification (token 73227) | **Confirmed failed (UI-reported)** for the connected wallet, but **ambiguous** in cause | Refusal copy in S3/S4/S5/S6; infrastructure-failure copy in S3/S7/S1. The UI does not itself prove ownership either way. |
| Demo license minting | **Not shown** | Button visible (S5, S6). No click, prompt, tx hash, or new token ID anywhere. |
| Protected request | **Pending or stalled** (S2) → **Ambiguous** afterwards | S2 shows "Waiting for the protected read". No outcome beyond the S3/S1 error states. |
| Authorization (CDR read condition) | **Ambiguous** | Refusal copy and infrastructure-failure copy coexist (S3). |
| Decryption | **Not shown** | Step 04 "Recover & decrypt" is WAITING in S2, S3, S4, S7. |
| SHA-256 verification | **Not shown** | Step 05 is WAITING in every capture. |
| Download | **Not shown** | No download control or content appears. |

---

## F. Potential technical explanations (all unconfirmed hypotheses)

Ranked by how much evidence supports each. None is a conclusion.

### H1: A legitimate on-chain refusal reached the app through a path whose error text was not recognized, so it was classified as an infrastructure failure
- **Supporting:**
  - The wallet `0x1E97…7559` is not the address the repo records as the owner of token 73227. PROGRESS.md says token **73227 was minted to the "reader" wallet `0x226e…A863`**, while the repo's *contract owner* is `0x75D9…aCa9`. Neither is `0x1E97…7559`. So a refusal is the expected outcome for this wallet and this token.
  - The refusal copy (D1, D2) is shown alongside the error banner.
  - In `lib/protocol/access.ts` (verified), a refusal is recognized **only if the error text contains `"CDR: Read condition not met"`**. A bare "execution reverted" becomes `MALFORMED_REQUEST`, which maps to `INFRASTRUCTURE_FAILURE`. Any other error is also `INFRASTRUCTURE_FAILURE`. The UI label in S3 is `INFRASTRUCTURE_FAILURE`, which cannot distinguish those cases.
- **Contradicting / unknown:** no error text is visible. The repo code I read may not be the code that was deployed (see Section G). S1 and S7's labels are collapsed or absent.
- **Check needed:** capture the underlying error detail (the `failure.detail`) for one failing attempt, and see whether it contains the contract's refusal string.

### H2: Two independent state sources are being displayed together (or one is stale)
- **Supporting:** S3 shows refusal text and the error banner together. S6/S7 show NO LICENSE and the error banner in one session. The repo's `reduceAccess` in `lib/access/state.ts` (verified) holds a **single** phase at a time, so one reducer state cannot be both NO_LICENSE and VERIFICATION_ERROR.
- **Contradicting:** I could not read the UI components that render these. The panels could legitimately reflect different actions.
- **Check needed:** read the component(s) that render the checkpoint list, the status badge, and the banner, and see which state each reads from.

### H3: Deployed build changed between the 6:33 pm and 12:52 am sessions
- **Supporting:** S3 contains an "Optional proof test / Test locked read" block that S7 lacks. The lead-in sentence above the banner differs (D2 vs D4). S1's copy ("errors never become a license denial") fits a later behavior. The tab count and battery also show a different session.
- **Contradicting:** conditional rendering based on state could also explain the differences.
- **Check needed:** the Vercel deployment list with commit hashes and timestamps for 15–16 Oct.

### H4: The wallet-signed protected read failed or was interrupted in the Rabby in-app browser (provider, gas estimation, or approval flow)
- **Supporting:** the failure appears in the stage that needs a wallet prompt (S2 says "Your wallet may ask for transaction approval"). The environment is Rabby's mobile dApp browser. `classifyThrownError` and `classifyReadFailure` (verified) send timeouts and unknown errors to `INFRASTRUCTURE_FAILURE`.
- **Contradicting:** no Rabby prompt, rejection, or error is shown. Nothing here singles out Rabby as the cause.
- **Check needed:** screenshots of Rabby's prompt (or its absence), and the same flow in the Rabby desktop extension for comparison.

### H5: Network/RPC/keeper transport problem (RPC, `/api/cdr/...` proxy, or validator partial collection)
- **Supporting:** the repo notes the keeper API is plain HTTP, proxied through `app/api/cdr/[...path]/route.ts`. PROGRESS.md also records transient HTTP 500 responses from the partials endpoint that the SDK treats as retryable, and a 120-second default timeout in `requestDataKey`.
- **Contradicting:** partials are only collected after a *successful* read transaction. For a wallet that does not own the token, the read should revert first. The free license check did return a definite NO LICENSE, so the RPC was reachable at least from the server side at that moment. No evidence of a submitted transaction.
- **Check needed:** the browser network log for `/api/cdr/*` and `/api/verify-license`, and the explorer view of the wallet's recent transactions.

### H6: The wallet was on a different chain at the time of the read
- **Supporting:** the page prints the chain as static text and I could not verify that it reads it from the wallet. The "Sto…" badge is truncated.
- **Contradicting:** no WRONG_NETWORK message appears, and the repo's state machine has a dedicated `WRONG_NETWORK` failure that would normally surface.
- **Check needed:** Rabby's active chain ID shown in the wallet, and the `eth_chainId` the page receives.

---

## G. Missing evidence

| Missing item | Why it matters | Best way to get it |
|---|---|---|
| Rabby's own popups (transaction or signature prompts, rejection, errors) | Shows whether a read or mint was ever submitted | Screen-record the whole flow on the phone, or screenshot each Rabby sheet |
| The underlying error text behind `INFRASTRUCTURE_FAILURE` | Needed to separate refusal vs malformed vs timeout vs provider error | Expand "Diagnostic label" on each failure and screenshot; capture the browser console and network logs (desktop browser with the Rabby extension is easier than the phone) |
| Whether `Test locked read` or `Get demo license` was tapped | Decides whether S2 was an in-flight read and whether any mint was attempted | The user's recollection; Rabby's activity list; the wallet's transactions on the explorer |
| Transaction hashes or Rabby activity for `0x1E97…7559` | Establishes whether anything was submitted or confirmed | Story Aeneid explorer (`aeneid.datanetscan.io`, per the repo) |
| On-chain owner of token 73227 and the wallet's license balance | Settles whether this wallet owns the token | Read `ownerOf(73227)` on the license token contract `0xFe3838BF…C6bC` and `balanceOf(0x1E97…7559)` |
| Full connected address | Currently abbreviated, and cut off in S1 | Screenshot of the Rabby account screen |
| Wallet's active chain ID | Confirms network | Rabby network indicator |
| Exact capture order within 12:52 | The timeline depends on it | Original file timestamps (seconds) from the phone gallery |
| Dates of captures | Only inferred | File metadata |
| Deployed commit at each capture time | S3 vs S7 differ | Vercel deployments page |
| Response from the free license-check route for this address | Separates the free check from the wallet-signed read | Network log entry for `/api/verify-license` |
| Contents of UI components and any code added after 13 Oct | See repository section below | Provide the relevant files or a fresh clone |

### Repository inspection (read-only)

**What I could and could not access**
- Fetched successfully: the repo landing page and README, `PROGRESS.md`, `lib/protocol/access.ts`, and `lib/access/state.ts`.
- My fetch tool was blocked on GitHub folder listings (`/tree/...` returned a robots.txt refusal), so I did not read `components/`, `app/`, or any file whose exact path I could not confirm. **I did not read the UI components or the browser read flow.**
- PROGRESS.md says "Last updated: 2026-10-13" and describes the browser read flow as **not yet implemented ("PHASE 2 … is next")**. The README says the same. Yet the screenshots from 15–16 Oct show a "Sign CDR read" step, "Waiting for the protected read", and a "Test locked read" button. The repo docs I could read are therefore **older than the deployed behavior**, and the code that produced the screenshots may not be what I read.

**Verified findings (with source)**

1. `lib/protocol/access.ts` defines four outcomes: `GRANTED`, `REFUSED`, `MALFORMED_REQUEST`, `INFRASTRUCTURE_FAILURE`.
   - `classifyReadFailure` returns `REFUSED` only when the error text contains `"CDR: Read condition not met"`.
   - Timeout/"aborted" text returns `INFRASTRUCTURE_FAILURE`.
   - Text containing "execution reverted" (without the refusal string) returns `MALFORMED_REQUEST`.
   - Anything else returns `INFRASTRUCTURE_FAILURE`.
   - `failureForAttempt` maps `MALFORMED_REQUEST` and `INFRASTRUCTURE_FAILURE` to `INFRASTRUCTURE_FAILURE`.
   - `requestDataKey` calls the SDK's `accessCDR` with a default `timeoutMs` of 120,000.
2. `lib/access/state.ts` defines seven phases: `LOCKED`, `CHECKING`, `NO_LICENSE`, `LICENSE_VERIFIED`, `ACCESSING`, `UNLOCKED`, `VERIFICATION_ERROR`.
   - There is one `AccessState` at a time.
   - `phaseForFailure` sends `AUTHORIZATION_FAILURE` to `NO_LICENSE` and every other failure kind to `VERIFICATION_ERROR`.
   - `READ_REFUSED` moves `CHECKING` or `ACCESSING` to `NO_LICENSE`.
   - `UNLOCKED` needs `PLAINTEXT_RECOVERED` evidence from the `ACCESSING` phase.
   - `phaseLabel("VERIFICATION_ERROR")` returns "LICENSE COULD NOT BE VERIFIED", which differs from the "VERIFICATION ERROR" badge in S1, so the badge text comes from a different file (probably `components/LicenseStatus.tsx`, not read).
3. PROGRESS.md (as of 13 Oct) records:
   - License token **73227** was minted to the reader wallet `0x226e…A863`, and the repo's contract owner `0x75D9…aCa9` "holds no licence".
   - A free `eth_call` to the read condition returns `false` for a non-owner presenting a real token ID, and `true` for the owner.
   - The recorded CLI run decrypted 804 bytes and matched SHA-256 `46cab4a8…4cd7`. It was done from a CLI harness, not a browser.
   - The keeper API is plain HTTP, proxied at `app/api/cdr/[...path]/route.ts`. The partials endpoint can return HTTP 500 transiently while partials are not yet available, and the SDK keeps polling.
   - Read fee was 0 when measured, and gas is separate.

**Not verified:** which component renders each message, whether the shown refusal text and error banner are driven by one state, how "Get demo license" mints, how the browser read flow signs the transaction, how chain ID is detected in the browser, or what the deployed code at the time of the screenshots was.

---

## H. Handoff to the debugging assistant (paste-ready)

```
CONTEXT
LicenseVault (Next.js, Vercel: licensevault.vercel.app; repo Temmygabriel/LicenseVault) on Story Aeneid (chain 1315).
Tested on Android in Rabby Wallet's built-in dApp browser. Connected account: 0x1E97…7559 (full address unknown).
Gate token shown: #73227. Wallet balance shown: "1 testnet token". CDR protocol read fee shown: "0 testnet token".
Evidence = 7 screenshots from two sessions: ~6:33 pm (15 Oct, inferred; tab count 16; battery 32%) and 12:52–12:53 am (16 Oct; tab count 17; battery 52–53%).

SEQUENCE (S-numbers = order supplied; best-reading chronology in parentheses)
6:33 pm session
 - S6: ACCESS STATUS "NO LICENSE"; "The license condition refused the read. This is a real answer from the chain, not a guess — access requires a Commercial Use license for this asset."; GET DEMO LICENSE visible; token field prefilled 73227.
 - S7: banner ACTION NEEDS ATTENTION "The network could not establish a reliable result. This is not a licensing decision; try again later." with "Diagnostic label" collapsed. Lead-in: "The request did not produce a trustworthy result. This is not a license denial; resolve the error and retry." No "Optional proof test" block present. Order of S6 vs S7 unknown.
12:52 am session (best reading: S5 → S4 → S3 → S2)
 - S5: NO LICENSE, same refusal copy, "LICENSE HOLDER 0x1E97…7559".
 - S4: checkpoints: 01 Connect wallet ✓ COMPLETE; 02 Verify license BLOCKED; 03 Sign CDR read WAITING; 04 Recover & decrypt WAITING; 05 Verify content WAITING. Text: "The on-chain condition refused this wallet for the demo token being checked. Get a demo license or explicitly test a locked read."
 - S3: the refusal sentence above AND banner "…could not establish a reliable result…" with expanded label INFRASTRUCTURE_FAILURE; plus "OPTIONAL PROOF TEST — Show the gate refusing access" and button "TEST LOCKED READ · GAS REQUIRED" (idle).
 - S2: "Asking the live Aeneid license condition…" + "Waiting for the protected read — Your wallet may ask for transaction approval…" Steps 04/05 WAITING.
12:53 am
 - S1: ACCESS STATUS "VERIFICATION ERROR"; button RETRY VERIFICATION; text "We could not establish whether this wallet holds the license. That is a failure on our side of the wire — it is not a statement about your rights." GET DEMO LICENSE not shown in this state.

VERIFIED FACTS
 - Wallet connection step shows complete (S4). No wrong-network message in any screenshot. Page prints "Story Aeneid · Chain 1315" but its source (wallet vs constant) is unknown.
 - Refusal copy and infrastructure-failure copy are displayed together (S3) and in the same session (S6/S7).
 - Decryption (step 04) and SHA-256 verification (step 05) are WAITING in every capture. No decrypted content, hash, download, or tx hash appears anywhere.
 - No screenshot shows GET DEMO LICENSE or TEST LOCKED READ being tapped, a Rabby prompt, or any submitted/confirmed transaction.
 - UI differences between sessions: S3 has an "Optional proof test" block that S7 lacks; lead-in sentences differ.
 - Repo (lib/protocol/access.ts, verified): REFUSED only if the error text contains "CDR: Read condition not met"; bare "execution reverted" → MALFORMED_REQUEST; timeouts/unknown → INFRASTRUCTURE_FAILURE; MALFORMED_REQUEST and INFRASTRUCTURE_FAILURE both surface as the label INFRASTRUCTURE_FAILURE.
 - Repo (lib/access/state.ts, verified): single AccessState; only AUTHORIZATION_FAILURE/READ_REFUSED reach NO_LICENSE; all other failures → VERIFICATION_ERROR.
 - Repo (PROGRESS.md, last updated 2026-10-13): token 73227 was minted to 0x226e…A863 (not to 0x1E97…7559); browser read flow was documented as not yet implemented. Screenshots show a browser read flow, so the docs are older than the deployed behavior.
 - I could not read UI components (GitHub folder pages blocked for my fetch tool).

UNRESOLVED
 1. Is the "NO LICENSE" refusal (free check / S5,S6) the same event as the INFRASTRUCTURE_FAILURE banner (S3,S7)? Or two separate checks, a stale state, or a misclassified refusal?
 2. What was the underlying error behind INFRASTRUCTURE_FAILURE (diagnostic label expansion only shows the label)?
 3. Was a protected-read transaction ever submitted? Was there a Rabby prompt?
 4. Which wallet chain ID did the page actually see?
 5. Did the deployed build change between the two sessions?
 6. Does 0x1E97…7559 own any license token, and who owns 73227 now?

RECOMMENDED DIAGNOSTIC CHECKS (read-only; no fixes assumed)
 1. Reproduce once in a desktop browser with the Rabby extension; capture console + Network tab for /api/verify-license and /api/cdr/* and the raw error object for the failing read.
 2. Expand the diagnostic label and record failure.detail for each failure; check whether the text contains "CDR: Read condition not met".
 3. On-chain reads: ownerOf(73227) and balanceOf(0x1E97…7559) on the license token contract 0xFe3838BFb30B34170F00030B52eA4893d8aAC6bC; wallet activity for 0x1E97…7559 on the Aeneid explorer.
 4. Confirm Rabby's active chain is 1315 and what eth_chainId the page receives.
 5. List Vercel deployments and commits for 15–16 Oct; compare with the S3 vs S7 UI differences.
 6. Read the components that render the status badge, checkpoint list, banner, and optional-proof-test block, to see how many state sources drive them.
 7. Do not weaken the license check or change contracts, chain, or wallet based on this evidence alone.
```

---

*End of report. No files in the project were created, edited, or deployed in producing it.*
