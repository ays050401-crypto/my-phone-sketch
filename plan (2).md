# Plan: Growing Through Conversation (Phase 2)

<!-- Everything above Steps describes the idea. The agent builds from the Steps,
     one step at a time, and stops after each step so it can be checked on a phone. -->

## Project
Phone as a listener that turns what people say into music, to grow one shared virtual character together, for a small group (2–4 people) talking in the same room.

## The question
Phase 1 asked: how can a phone transform conversation into music? It mapped mouth movements (HOW people speak) directly to notes.
Phase 2 shifts to WHAT people say: can emotional words in an everyday conversation become music and visibly change something the group shares?

It works if:
- People see their own feeling words light up on their phone while they talk.
- A pause sends a "conversation ball" to the laptop, and everyone can hear its melody and see the character react.
- The group notices that positive talk makes the character grow and negative talk stops or shrinks it, and starts talking *to* the character on purpose.

## The experience
1. A laptop on a table shows the shared screen: one glowing 3D sphere (the character), a title, and a QR code.
2. Each person scans the QR with their phone and taps once to start. Their phone shows a small grey ball and "Talk with your group".
3. They hold their phone near their mouth and talk normally, in English. Words appear under the ball; feeling words (happy, love, enjoy, sad, angry …) appear in color and play a short melody. The ball grows and takes the color of the main feeling.
4. When they pause for about 2 seconds, the ball flies up off the phone screen ("sent to the shared screen ↑").
5. On the laptop, the ball flies in from below, enters the character, plays its melody, and the feeling words float up out of the character. Positive → the character grows with a happy bounce. A little negative → it shivers but does not grow. More negative → it shrinks.
6. People keep talking; many balls from many phones keep shaping the same character. The laptop lists the last few balls (who, which words, how the size changed).

People watching see the laptop: the character, balls flying in, rising words, the size counter.

## Input, transformation, output, fallback
- Input: speech through the phone microphone, turned into English text by the browser's speech recognition (Web Speech API, started through p5-phone's speech permission). Touch only for the start tap.
- Transformation:
  - A "section" = everything said until 2000 ms without new words (cut at 20 s max; sections under 2 words are dropped).
  - Each word is checked against a feeling dictionary with 5 feelings: joy (+1), love (+1), enjoy/calm (+1), sad (−1), angry (−1). A negator up to 2 words before ("not", "don't", "never" …) flips it ("not happy" → sad, "not bad" → calm).
  - Score = sum of the feelings (max 8 feeling words per ball). Main feeling = the most frequent one; none → neutral.
  - Ball size = number of words (full size at 40 words). Ball color = main feeling.
  - Only feeling words + word count are sent to the laptop, never the sentence.
  - Character size starts at 1.0, min 0.5, max 3.5. Score > 0 → +0.08 per point. Score −1 → no change (shiver). Score ≤ −2 → −0.06 per point. Max 4 points per ball.
- Output:
  - Phone: the forming ball (shaded sphere, colored by main feeling), the live words with feeling words colored, a short melody per feeling word, the ball flying away on pause, connection status, current character size.
  - Laptop: the character as a shaded 3D-looking sphere with a floor shadow, slowly breathing; balls flying in on a curve; each ball's melodies played in order; feeling words rising; size/count; recent-ball list; color legend; QR code to join.
  - Sound per feeling (generated, no files): joy = bright rising arpeggio; love = warm slow rolled chord; enjoy = gentle 3-note phrase; sad = slow falling minor line; angry = low harsh short chords; neutral = one soft tone. Light echo on everything.
- Fallback:
  - Laptop without `?role=screen` = acts as a phone (speech works in desktop Chrome).
  - Screen with no phones: key T sends a random test ball, R resets the character, F toggles full screen.
  - Browser without speech recognition (some iPhones / in-app browsers): message "open the link in Chrome". Mic refused: message to allow it and reload.
  - Screen not open yet: balls wait on the phone and are sent when it connects.
  - Laptop reload: the character's size, color and count are remembered (browser storage).

## References
| File | Use it as | Take | Leave |
|---|---|---|---|
| references/[layout image — to add] | layout: match this | [where things sit, their size] | [what to ignore] |
| references/[mood image — to add] | inspiration: the feel | [two or three qualities] | [the rest] |

Until images are added: dark background (grey 10), flat bright feeling colors (joy #FFEA27, love #EC66FF, enjoy #A9FF66, sad #66BEFF, angry #FF6666, neutral #9A9A9A), Chakra Petch for titles/labels, Helvetica for small text — same visual language as Phase 1.

## Limits
- Change only sketch.js.
- English only.
- One shared screen per room; 2–4 phones.
- Not now: a real character shape (it stays a sphere), growth stages that change its look, Korean words, a smarter emotion model than the word list, saving the conversation's music.

## How I will check it
- On my laptop: open the link with `?role=screen&room=test`, click once. I see the sphere, title, QR and legend. Pressing T sends test balls: they fly in, play music, words rise, the size and the list change.
- On my phone: scan the QR, tap once, allow the mic. Saying "I'm so happy today, I love this" shows yellow/pink words, plays short melodies, and after a 2 s pause the ball flies away. The laptop plays it and the character grows. Saying "I'm sad and angry" makes it shrink. Saying "I'm not happy" counts as sad.

## Steps
<!-- Each step is small enough to check on a phone. Stop after each one. -->

Rules for every step (from the p5-phone and p5.js 2 skills):
- p5.js 2.x: `async function setup()` with `await` for loading (no `preload()`); 2.x input names (no `touchStarted`/`keyCode`).
- `lockGestures()` in setup. All permissions from ONE p5-phone tap (`enablePermissionsTap`), never on load. Wait for `userSetupComplete()`.
- Gate on the p5-phone flags (`speechEnabled`, `soundEnabled`).
- Sound through p5.sound (`p5.Oscillator`, `p5.Envelope`, `p5.Reverb`), not a hand-written Web Audio graph. No sound files.
- Wake lock: call `navigator.wakeLock` directly from `mouseReleased()`, in try/catch, re-request on `visibilitychange`.
- Keep all numbers that matter as named constants at the top of sketch.js, with a plain-English comment each.
- Keep `showDebug()` on and log every important event with `debug()`.

1. **Two roles, one file.** Read `?room=` and `?role=screen` from the link. Draw a dark screen with the role and room name in the middle. Phone role: `showDesktopQr()`. Check: the link with and without `role=screen` shows the right label.

2. **Start tap.** Phone: one p5-phone tap asking for speech + sound. Screen: one tap asking for sound. Log the flags in `userSetupComplete()`. Check: the tap screen appears; after tapping, the debug log shows both flags.

3. **Phone hears words.** Create the browser's `SpeechRecognition` (en-US, continuous, interim results) after the tap. Restart it when it ends. Show the live words in the middle of the phone. Show a clear message if the browser has no speech recognition or the mic was refused. Check on an Android phone in Chrome: my words appear while I talk and keep appearing after a long silence.

4. **Sections.** Collect words into a section; 2000 ms without new words (or 20 s total) ends it. Drop sections under 2 words. Log each finished section. Handle Android's habit of repeating earlier words in new results. Check: each pause logs exactly one section, without repeated words.

5. **Feeling words.** Add the 5-feeling dictionary and the negator rule as constants. For each section compute the feeling words, score and main feeling, and log them. Color feeling words in the live text. Check: "I'm not happy" logs sad, "not bad" logs calm.

6. **Feeling sounds.** One short generated melody per feeling (see Output) with light reverb. The phone plays a feeling word's melody once, softly, when the word is final. Check: each feeling sounds clearly different; no sound repeats while I keep talking.

7. **The phone ball.** A shaded sphere (light from top-left, highlight, soft floor shadow) in the upper middle. Size grows with the section's word count, color fades to the main feeling, small bounce when a feeling word lands. On section end the ball flies up off the screen. Check: talk → it grows and changes color; pause → it flies away and a new small grey ball starts.

8. **Shared screen character.** On the screen role, draw the character: the same shaded sphere, centered, slowly breathing, size from `charScale`. Add key R (reset) and F (full screen). Check on the laptop: a calm glowing sphere with a shadow.

9. **Test balls on the screen.** Key T makes a random ball. It flies in from below on a curve, enters the character, plays its melodies in order (balls queue, never overlap), feeling words rise out in time with the music, and the size follows the growth rules (grow / shiver / shrink). Check: pressing T many times shows all three reactions and the size never leaves 0.5–3.5.

10. **Connect phones to the screen.** Use PeerJS (loaded from unpkg inside sketch.js, public server, STUN + PeerJS relay). The screen takes the id `[prefix]-[room]-screen`; each phone connects to it and retries every 3 s. The screen numbers phones 1, 2, 3 … Phones send balls `{id, words, hits, score, main}`; balls made while not connected wait and are sent later. The screen rebuilds every ball from known feelings only (never trusts the message) and ignores repeated ids. Check: phone status shows "screen connected"; a pause on the phone makes a ball arrive on the laptop.

11. **Screen information.** Status (room, number of phones), title "GROWING THROUGH CONVERSATION", counter (conversations, size), the last 6 balls (person, words, change), the color legend, and a QR code of the phone link (bottom left). After each ball the screen sends the new size to all phones; the phone shows "character ×1.40". Check: everything is readable from 2 m away; the phone's size matches the laptop's.

12. **Remember and keep awake.** The screen saves size, color and count in browser storage and loads them on reload. Wake lock on both roles. Bottom hint line that tells people what to do next. Check: reload the laptop → same character; phone screen does not go dark while talking.

13. **Group test.** 3 phones + laptop in one room. Tune constants only (silence time, dictionary, growth amounts, volumes). Note what was changed in Changes.

## Changes
<!-- One line each time this plan changes, and why. -->
- 2026-10-27: First version, written from the Phase 2 draft sketch.js. Moved sound to p5.sound and wake lock to `mouseReleased()` to follow the p5-phone / p5.js 2 skills.
