# LicenseVault: "Sign protected read" Failure, Read-Only Diagnostic Report

**Date:** Friday 16 October 2026
**Scope:** read-only. Nothing in the repository was edited, committed, pushed, or deployed. The repo clone is clean (`git status` shows 0 changes, HEAD `5f0e536`).
**Method:** I read the current `main` source, downloaded the published `@piplabs/cdr-sdk@0.2.2` and `@piplabs/cdr-crypto@0.2.2` tarballs into a scratch folder, and ran an ordinary production build of `main` in a scratch copy (`npm ci --ignore-scripts`, then `next build`) so I could inspect the browser bundle that Next.js actually produces.

**Labels:** **Confirmed** means seen in code or build output. **Inferred** means follows from evidence but was not observed directly. **Hypothesis** means plausible and untested.

---

## 1. Summary

**Most likely root cause (confirmed in build output, not yet observed on your device or in production):**
`await initWasm()` at `lib/protocol/browser-clients.ts:63` throws in the browser bundle, on every call.

The SDK's WASM loader contains `await import(wasmShimSpecifier)`, a dynamic import whose target is computed at runtime. When Next.js 16.3.8 (Turbopack) bundles it for the browser, it compiles that import into a function that always throws:

```
Error("Cannot find module as expression is too dynamic")   // code = "MODULE_NOT_FOUND"
```

That happens inside `createBrowserCdrClient`, which runs *before* the SDK asks Rabby for any transaction. This fits all four of your symptoms: a few seconds of loading, no Rabby prompt, no read-transaction link, and a verification error with no useful detail.

**Why you could not find a diagnostic label:** it exists, but (a) it only shows a category such as `INFRASTRUCTURE_FAILURE`, never the real error text, and (b) it sits inside a collapsed `<details>` further down the card, below the part of the page your two 01:37 screenshots show. Details are in section 4.

**Confidence:** high that this defect exists in the bundle built from `main`. Medium-high that it is the cause of your failure. The remaining gap is that I could not observe your production runtime or read the actual error text. Section 8 gives a safe way to close that gap.

---

## 2. What your two new screenshots show

| Screenshot | Shows |
|---|---|
| Image 2 (1:37 am) | State is `LICENSE_VERIFIED` ("The live condition confirmed the license. Sign the protected read next…"). Wallet balance `0.999923625874908392`, CDR read fee `0`. Receipt shows token `#73228`, a mint transaction link, state "IN PROGRESS". |
| Image 1 (1:37 am) | Status badge `VERIFICATION ERROR`, token field `73228`, button `RETRY VERIFICATION`. The lower part of the card, where the red "Action needs attention" box would be, is below the visible area. |

**Useful deduction (inferred, from code plus the screenshots):**

- `setNativeBalance` and `setProtocolReadFee` are called only inside `preflightRead` (`components/AccessProvider.tsx:518-519`). Minting does not call it.
- Your balance is `0.99992…`, below the `1` shown at 12:52. Gas from the mint has been deducted.
- So a protected-read attempt ran **after** the mint and got through `preflightRead`: chain check, balance, and `readFee` all succeeded.
- That rules out wrong network, RPC outage, and zero balance for that attempt. The failure happened after line 519.

---

## 3. Code path from the click to decryption, with failure points

Click handler: `components/AccessPanel.tsx:73` (`Sign protected read` → `unlockProtectedContent` → `performRead(false)`).

| # | Step | File:line | Result if it fails | Status |
|---|---|---|---|---|
| 1 | Wallet and chain check: `eth_chainId` from Rabby, then RPC `getChainId` | `AccessProvider.tsx:292-307` (`currentWalletOnAeneid`) | `WrongNetworkError` → kind `WRONG_NETWORK` → "Your wallet is not on Aeneid Testnet." Other errors → generic message | Passed (inferred, section 2) |
| 2 | Preflight: chain, balance, `readFee` | `AccessProvider.tsx:501-525` (called at 598) | Zero balance → "Insufficient funds…" text | Passed (inferred) |
| 3 | State guard, then `READ_REQUESTED` (phase becomes `ACCESSING`) | `AccessProvider.tsx:628-636` | `Error("Verify the license before signing…")` → `INFRASTRUCTURE_FAILURE` | Unlikely: button only appears in `LICENSE_VERIFIED` (`AccessPanel.tsx:73`) |
| 4 | Load SDK: `await import("@piplabs/cdr-sdk")` | `lib/protocol/browser-clients.ts:60` | Chunk load error → `INFRASTRUCTURE_FAILURE` | Hypothesis, low |
| 5 | **`await initWasm()`** | **`lib/protocol/browser-clients.ts:63`** | **`Error("Cannot find module as expression is too dynamic")` → `INFRASTRUCTURE_FAILURE`** | **Confirmed defect in build output** |
| 6 | Build `CDRClient` with `apiUrl: "/api/cdr"` | `browser-clients.ts:65-79` | Configuration error. Relative `apiUrl` is safe: the SDK joins it with plain string concatenation (`story-api/transport.js` `joinPath`) | Unlikely |
| 7 | `requestDataKey` → SDK `accessCDR` | `AccessProvider.tsx:640-645`, `lib/protocol/access.ts:158-` | See rows 7a-7e | Not reached if step 5 throws |
| 7a | `observer.getGlobalPubKey()` via the proxy `GET /api/cdr/dkg/global_public_key` | `app/api/cdr/[...path]/route.ts:179-263` | Proxy envelope errors: 404 (path not allowed), 429 (limiter, `route.ts:165-169`), 502 (keeper unreachable or over 512 KB). SDK retries 5xx three times. All become `INFRASTRUCTURE_FAILURE` | Hypothesis, next suspect once step 5 is fixed |
| 7b | `loadAndCheckVault` (vault preflight, before the paid read) | SDK `consumer.js` `accessCDR` | Vault errors → `INFRASTRUCTURE_FAILURE` | Hypothesis |
| 7c | `read()`: **Rabby transaction prompt appears here** | SDK `consumer.js` | Revert containing `"CDR: Read condition not met"` → `REFUSED` → `NO_LICENSE` with "The live CDR read was refused…" (`AccessProvider.tsx:658`). Bare "execution reverted" → `MALFORMED_REQUEST` (`access.ts:103`) | Excluded by "no prompt appeared" |
| 7d | `collectPartials` (polls `/dkg/cdr_partials`, up to the 120 s timeout set at `AccessProvider.tsx:644`) | SDK, `route.ts` | Timeout → `INFRASTRUCTURE_FAILURE` | Excluded (needs a transaction) |
| 7e | `decryptDataKey`, which needs the WASM instance from step 5 | SDK | `WASM module not initialized` | Not reached |
| 8 | Fetch sealed payload (832 bytes), AES-GCM decrypt, SHA-256 check, JSON manifest check | `AccessProvider.tsx:527-579`, `lib/protocol/content-browser.ts` | `DECRYPTION_FAILED` ("The protected read did not produce verified plaintext…") | Excluded (needs a key) |

**What the user sees for a failure at step 5 (confirmed by tracing the code):**
1. The catch block at `AccessProvider.tsx:680-681` calls `failActiveFlow(error)`.
2. `classifyThrownError` (`lib/access/state.ts:165-208`) finds no timeout, revert, funds, or decrypt wording in the message, so the kind is `INFRASTRUCTURE_FAILURE`.
3. The phase moves to `VERIFICATION_ERROR` (badge "Verification error").
4. The status sentence is "We could not establish whether this wallet holds the license…" (`components/LicenseStatus.tsx:65-70`). That is the text you reported.
5. The banner text is the generic fallback at `AccessProvider.tsx:167`: "The network could not establish a reliable result. This is not a licensing decision; try again later."
6. The diagnostic label is `INFRASTRUCTURE_FAILURE` (line 287).

---

## 4. Why the diagnostic label seems missing, and whether the message matches current code

**Confirmed facts:**

1. **The label is deliberately shallow.** `setTechnicalDetail(failure.kind)` (`AccessProvider.tsx:287`). The comment at lines 285-286 says raw errors are kept out of the UI on purpose. So even when it shows, it can only say `INFRASTRUCTURE_FAILURE`, which covers timeouts, proxy faults, WASM faults, chunk-load faults, and unrecognised errors alike.
2. **It is collapsed and below the fold.** It renders only inside the red banner, as `<details><summary>Diagnostic label</summary>…` (`AccessPanel.tsx:138`). That banner appears under the checkpoint list (`AccessPanel.tsx:126-136`), well below the token field where both of your screenshots end. In the earlier 12:52 screenshot the label was visible once expanded.
3. **Tapping "Retry verification" erases it.** `retry` (`AccessProvider.tsx:699-703`) calls `verifyLicense`, which calls `setProblem(null)` and `setTechnicalDetail(null)` (lines 415-416). A successful re-check returns the page to `LICENSE_VERIFIED` with no banner. Your two screenshots are 8 seconds apart (file names `…013748` and `…013756`), and Image 2 shows a clean `LICENSE_VERIFIED` state, so the banner may already have been cleared.
4. **"Retry verification" never retries the read.** It only re-runs the free license check. This is misleading after a failed read.
5. **The messages match current code.** "We could not establish whether this wallet holds the license…" is `LicenseStatus.tsx:67-68`. "The network could not establish a reliable result…" is `AccessProvider.tsx:167`. The deployed UI strings in all screenshots also match `main`.
6. **Transaction links:** `readTxHash` is set only when a data key is granted (`finishWithRecoveredContent`, line 529) and `deniedReadTxHash` only on a decoded refusal with a hash (line 648). A failure before the wallet prompt can never produce a read link. Mint links (`mintTxHash`) are the only ones you can see so far.

**Explains an earlier puzzle (consistent with the code, not proven for those exact screenshots):** in the first evidence report, "NO LICENSE" appeared together with the infrastructure banner. `problem` and `technicalDetail` are separate React state (`AccessProvider.tsx:199-200`) from the reducer's phase. `mintDemoLicense` sets them on failure without changing the phase (lines 491-494). Before `5f0e536`, the mint failed on a fee-parsing bug (`browser-clients.ts:102-121`), which shows as exactly that generic banner while the phase stayed `NO_LICENSE`.

---

## 5. Is the deployed build different from `main`?

**What I could establish:**

- `main` HEAD is `5f0e536`, committed 16 Oct 2026 01:23:59 +01:00: "Fix demo license mint preflight fee parsing". It touches only `lib/protocol/browser-clients.ts` and its test.
- Commit `53cbfdf` (14 Oct 18:17:10 +01:00) introduced the browser flow, including `createBrowserCdrClient` and the `initWasm()` call. `git blame` shows lines 60-63 of `browser-clients.ts` are unchanged since that commit.
- **Correction to my first report:** I said the "6:33 pm" screenshots were probably from 15 Oct. The file names (`Screenshot_20261014-1833…`) show 14 Oct, about 16 minutes after `53cbfdf`.
- Minting working at about 01:37 implies that production contains the `5f0e536` fee fix (inferred).
- Because `initWasm()` has not changed since `53cbfdf`, any production build from `53cbfdf` onward has the same WASM behaviour. A commit mismatch would not change the conclusion.

**What I could not establish:** the exact deployed commit. The GitHub API needs credentials this session does not have (I did not request push access), the shell cannot reach `licensevault.vercel.app`, and the page fetch exposed no build ID. To confirm: Vercel dashboard → Deployments → Production → commit SHA and build time.

---

## 6. Evidence for the root cause

**Source (confirmed):**
- `lib/protocol/browser-clients.ts:60,63`: browser code does `await import("@piplabs/cdr-sdk")` then `await initWasm()`.
- The published `@piplabs/cdr-crypto@0.2.2` `dist/esm/wasm/loader.js` contains `const shim = await import(wasmShimSpecifier)`, where `wasmShimSpecifier = new URL("cb-mpc-tdh2.mjs", import.meta.url).href` (`wasm-paths.js`). This is a computed specifier.
- `next.config.ts:5-12` lists the SDK packages in `serverExternalPackages`. That setting only affects server bundles. Its comment ("keep them out of the browser bundle") does not describe what happens: the build output shows the SDK is bundled into a client chunk.

**Build output from a scratch build of `main` (confirmed):**
- Client chunk `static/chunks/1k21j_rbz4y1u.js` contains the compiled `initWasm`. After the SHA-256 check it reads: `(await Promise.resolve().then(()=>{let e=Error("Cannot find module as expression is too dynamic");throw e.code="MODULE_NOT_FOUND",e})).default`. This always throws.
- The WASM hash check *before* that line should pass: the emitted 5.5 MB WASM file's SHA-256 (`8b5698a8…018256a`) equals the SDK manifest value. That fits "loads for a few seconds, then fails" (fetch plus hash of a 5.5 MB file on a phone).
- **Second layer:** Turbopack also copied the shim as a raw asset, `static/media/cb-mpc-tdh2.0vewfcedp14a7.mjs`. That file still contains `new URL('cb-mpc-tdh2.wasm', import.meta.url)`, but the emitted WASM is named `cb-mpc-tdh2.0b4-abhiwr1kr.wasm`. No un-hashed `cb-mpc-tdh2.wasm` exists. So even if the import were allowed, the shim would request `/_next/static/media/cb-mpc-tdh2.wasm` and get a 404.
- The Turbopack build printed no warning for this, and CI would not catch it.

**Why tests did not catch it:** the repo's tests and tools (`tools/verify-live-read.ts`, `tools/spike/build-vault.ts`, `tools/verify-canonical-run.ts`) run `initWasm()` in Node. Node reads the files from disk and never takes the browser branch. No test runs the browser bundle.

---

## 7. Other possibilities, ranked

| Rank | Hypothesis | Evidence for | Evidence against | How to tell |
|---|---|---|---|---|
| 1 | WASM init throws in the browser bundle (this report) | Confirmed in build output; fits no prompt, a few seconds, generic message | Production runtime not directly observed | Section 8, steps 1-2 |
| 2 | Keeper proxy or keeper (`/api/cdr/dkg/global_public_key`) fails | Occurs before the Rabby prompt; keeper is plain HTTP; repo notes record transient 500s | Never reached if #1 is true | Network tab shows `/api/cdr/...` requests and their status |
| 3 | SDK chunk fails to load on mobile | Mobile network | `initWasm` is in the same chunk; fetch would also fail earlier | Network tab |
| 4 | `loadAndCheckVault` rejects the vault | Occurs before the prompt | Vault is read live elsewhere in the app and in the Node run | Console / network |
| 5 | Rabby provider or gas-estimation problem | Environment is Rabby's in-app browser | Failure happens before any wallet request for the read | A Rabby prompt would appear if this were the case |

Ruled out for the failing attempt (inferred): wrong chain, RPC down, zero balance, unrecognised refusal text (no transaction was sent).

---

## 8. Smallest safe next step (no mint, no extra transaction)

Do **not** mint again. Token `#73228` is already yours and verified.

1. **Static check, no wallet, no gas:** open `https://licensevault.vercel.app/` in desktop Chrome, open DevTools → Console, and run:
   ```js
   (async () => {
     const srcs = [...document.scripts].map(s => s.src).filter(Boolean);
     for (const s of srcs) {
       const t = await (await fetch(s)).text();
       if (t.includes("too dynamic") || t.includes("cb-mpc-tdh2")) {
         console.log(s, { tooDynamic: t.includes("Cannot find module as expression is too dynamic"), wasmRef: t.includes("cb-mpc-tdh2") });
       }
     }
   })();
   ```
   Prediction if this report is right: one chunk prints `tooDynamic: true`. This reads public static files only.
2. **Runtime check with the Rabby desktop extension:** DevTools → Network tab, filter `cdr`, click **Sign protected read** once. Prediction: `cb-mpc-tdh2.<hash>.wasm` loads (about 5.5 MB, status 200), there is **no** `/api/cdr/dkg/global_public_key` request, and no Rabby prompt. If `/api/cdr/...` requests do appear, the failure is downstream (rank 2 or lower) and this report's root cause is wrong for your case.
3. **If a Rabby prompt does appear:** that is the real read. It costs only network gas (read fee is 0). Approve or reject as you wish.
4. Expand **Diagnostic label** (below the checkpoint list, inside the red box) and note the text. Expect `INFRASTRUCTURE_FAILURE`.
5. Optional, only if the above is inconclusive: ask the other assistant to build a one-off debug build that shows the redacted error text, rather than debugging blind.

---

## 9. Recommended changes (described only, nothing implemented)

**A. Fix the cause (smallest first):**
1. **Stop relying on the SDK's computed dynamic import in the browser bundle.** Options, in order of how small they are:
   - *Patch the SDK loader* (for example `patch-package` on `@piplabs/cdr-crypto`) so the shim import is bundler-proof (`/* webpackIgnore: true */` or `/* turbopackIgnore: true */`) and the WASM URL points at stable names.
   - *Self-host the two files* in `public/` under fixed names (copied from `node_modules` at build time), so the shim's `new URL('cb-mpc-tdh2.wasm', import.meta.url)` resolves.
   - *Initialise the module in app code:* import the shim from `public/` natively, fetch and hash-check the WASM, call the factory, and place the resulting `CbMpcWasm` in the global slot the SDK reads (`Symbol.for("@piplabs/cdr-crypto:wasmInstance")`). This relies on an SDK internal and is the most fragile option.
   - Ask Piplabs for a browser-ready build, or move the read into a Web Worker or an externally-built ESM bundle.
   All options need testing in a real browser. I have not tested any of them.
2. **Correct the misleading comment** at `next.config.ts:5-7`: `serverExternalPackages` does not keep these packages out of the client bundle.

**B. Make the next failure diagnosable:**
3. Replace the label with a stage name plus sanitised message, for example `stage: CDR_WASM_INIT` and the redacted text, reusing the existing redaction helper (`lib/access/redact.ts`) so no RPC URL or secret appears. Keep raw SDK bodies out of the UI.
4. Show the `Diagnostic label` details open by default, or place the banner next to the action button so it is not below the fold.
5. Do not clear `problem` and `technicalDetail` silently when Retry succeeds; keep the last failure visible (or log it).
6. Make "Retry verification" retry the failed operation (the read) or rename it ("Re-check license"), since it currently never re-attempts the read.

**C. Prevent regression:**
7. Add a browser-level smoke test (Playwright or similar) against `next build` output that calls `createBrowserCdrClient` and asserts `initWasm` resolves. A cheap guard: fail CI if any built client chunk contains `Cannot find module as expression is too dynamic`.

---

## 10. Limits of this report

- The exact error text from your device was never captured. The banner text and label for your 01:37 failure are not in the two screenshots.
- The production deployment and its commit could not be inspected.
- I built `main` once in a scratch copy with Next.js 16.3.8 and the committed lockfile, which is the version Vercel uses by default for `next build`. I did not run the page in a browser with a wallet.
- Downstream failure points (section 3, rows 7a-8) are untested because the confirmed defect stops execution before them. The Node harness did record a working end-to-end read in October (SHA-256 `46cab4a8…4cd7`), so those later stages are known to work outside the browser.
- No secrets, private keys, or environment-variable values were read or used.
