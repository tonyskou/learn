# Διαβάζω! — Ελληνική έκδοση (learn-gr)

Greek word-learning PWA built for Antonios's daughter. Teaches Greek reading/syllabification.

## Deployment
- **GitHub repo:** `tonyskou/learn`, hosted on GitHub Pages at `https://tonyskou.github.io/learn/`
- **Firebase project:** `diabazo` (separate from the Chinese version's `learn-cn-80ac0`)
- **Firestore security rules:** already set to permanent production rules (not test mode):
  ```
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {
      match /users/{userId}/{document=**} {
        allow read, write: if request.auth != null && request.auth.uid == userId;
      }
    }
  }
  ```

## Files
`index.html`, `words.js`, `vocabulary.js`, `stories.js`, `manifest.json`, `sw.js`, `icon-180/192/512.png`, `README.md`

## Content
- **317 sets × 10 words = 3170 words** (grew from an original 70-set/700-word `words.js` core, extended via `vocabulary.js` which appends `EXTRA_WORD_SETS` + `EXTRA_WORD_SETS_FINAL` into the same `WORD_SETS` array)
- Word object shape: `{w:"ΚΕ-ΦΑ-ΛΙ-ΚΟ",s:["κε","φα","λι","κό"]}` — `w` is uppercase hyphenated (no accents), `s` is the array of lowercase accented syllables shown on the card
- Color-coded syllabification (alternating colors per syllable), **no images/emoji on word cards** (forces reading, not guessing)
- Card shows the word twice: large uppercase (no accents) + smaller lowercase (with accents) underneath
- Set titles were originally in Chinese in `vocabulary.js` (sets 71–317); all 247 were translated to Greek
- Retains the **child-profile flow** (unlike the CN version) — parent logs in, then selects/creates a child profile, progress is stored per child

## Fixes already applied (do not re-apply blindly — check current file state first)
1. **Diphthong-split syllabification bug** (306 words): a stressed diphthong (αι, ει, οι, ου, αυ, ευ, ηυ, υι) was being split across two syllables instead of staying in one (e.g. "ο-ί-νος" → should be "οί-νος"). Fixed via script that detects an unaccented vowel ending one syllable followed by an accented ί/ύ starting the next.
2. **Final-sigma bug** (34 words): in two-word phrases (space-separated entries), the first word's final σ should be ς. Detected via: syllable ending in σ immediately followed by a space marker in the `s` array.
3. **5 merged-word entries** with no space between two Greek words (detected via mid-word ς appearing non-finally — a strong signal since real words never have σ mid-word):
   - κάλυμμα κρεβατιού (was "καλυμμακρεβατιου")
   - γραμμωτός κώδικας (was "γραμμωτοσκωδικας")
   - αλογάκι της Παναγιάς (was "αλογακιτησπαναγιας")
   - αξονικός τομογράφος (was "αξονικοστομογραφος")
   - ιδιωτικός χώρος (was "ιδιωτικοσχωρος")

## Known unresolved issues
- Some `vocabulary.js` set titles don't match their actual word contents (e.g. a set titled "seafood" contains flowers) — flagged, not fixed, low priority.
- Set 42, word 9 (originally "ΣΚΑΛΙΣΤΡΙ") looks garbled/not a real Greek word — never resolved for this version.
- `README.md` still had outdated stats (referenced "700 words / 70 sets" and "30 stories") as of the last check — **needs a rewrite** with current numbers (3170 words / 317 sets — verify actual current story count in `stories.js` before writing, don't assume 40 matches the CN version).

## Style notes for future edits
- All fixes were applied via small Node.js scripts doing targeted regex/string replacement on the raw `.js` text (not by rewriting the whole file), to avoid disturbing formatting or accidentally corrupting entries. Prefer the same approach: read exact current text first, build old_str/new_str pairs, verify with `node -c` after.
- When editing `vocabulary.js`, remember it's split into two arrays internally (`EXTRA_WORD_SETS` then `EXTRA_WORD_SETS_FINAL`) that both get pushed onto `WORD_SETS` — a naive block-regex across `{ id:N, ... }` can fail to match nested brackets; split the file into per-set chunks first (e.g. `text.split(/(?=\{ id:\d+,)/)`) rather than using one big regex.
