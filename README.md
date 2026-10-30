**[English](#lichess-equivalent)** · **[Türkçe](#turkce)**

# Lichess-equivalent

A chess arbiter that plays by **lichess's rules** rather than FIDE's, in a single HTML file of **2,804 bytes** — and the same arbiter stripped of its board, in **1,088**. Two players, one screen. No libraries, no build step, no server. Download a file, double-click, play.

The two rulebooks are close, but they are not the same book, and every place they part ways is written down below.

Part of the [Golfstack](https://www.fidelite.art/) project.

## Play

| file | interface | bytes | GitHub Pages | project site |
| --- | --- | --- | --- | --- |
| `index.html` | clickable board, clock, lichess colours | 2,804 | [Lichess-equivalent](https://cuneytinann.github.io/Lichess-equivalent/) | [Lichess-equivalent.html](https://www.fidelite.art/special/Lichess-equivalent.html) |
| `numerical_packed.html` | square numbers typed into a `prompt()` box, no board | 1,088 | [numerical_packed.html](https://cuneytinann.github.io/Lichess-equivalent/numerical_packed.html) | [Lichess-eqv_min.html](https://www.fidelite.art/builds/special/Lichess-eqv_min.html) |

On the project site both builds live under `special/`, off to the side of the `L1`–`L3` ladder. They are not another rung on it; they follow a different rulebook.

Anything from late 2020 onwards will run them: Chrome 85+, Firefox 79+, Safari 14+. Three things set that floor — BigInt, which the starting position is written with; the `safe` keyword in `place-content`, which keeps the board reachable on a narrow screen; and the `||=` operator, which the packed build uses.

**On the board.** Click a piece, then click where it should go. Legal squares pick up a dot, a piece you can take picks up a ring, the square you came from and the square you landed on stay tinted, and a king in check glows red. After every move the board turns around to face whoever is to play. When a pawn reaches the last rank the file it landed on turns into the picker: the four promotion choices sit stacked on the board itself, each on a grey disc, and you click the one you want. `½` offers a draw, or claims one when you are entitled to it, and it lights up on its own when the offer is yours to answer or the position has come round for the third time; `⚐` resigns. The two controls sit side by side and look alike on purpose — same height, white on colour, green for the draw and red for the resignation. The clock starts at ten minutes and hands back five seconds a move.

**Without the board.** `numerical_packed.html` draws nothing at all. A square is two digits, rank first and file second, each counted from 0: a1 is `00`, e2 is `14`, h8 is `77` — the engine's own 0x88 index, written in hexadecimal. A move is the two squares written end to end: e2–e4 is `1434`. An `8` or a `9` in either digit points off the board, and the arbiter refuses it like any other illegal move. To promote, add a fifth digit — `0` bishop, `1` rook, `2` knight, anything else queen. A draw is offered the same way, by hanging a non-digit on the end of the move: `1434x` plays e2–e4 with an offer attached. Type the non-digit on its own and the offer still stands, but you owe a move afterwards. The opponent accepts by answering in kind; a plain move declines it and wipes it off. If the position has already come up three times, that same non-digit claims the threefold outright. To resign, leave the box empty or press Cancel.

That is the one place the two builds part company, and it is a difference of gesture rather than of rule. On the board a draw offer is a standing flag you raise with `½` and lower the same way; in the dialog it rides along with a move. Either way the arbiter reads the same two bits, and either way the opponent's plain move turns the offer down. The dialog title carries White's clock, Black's clock and the last move you got past the arbiter, so when an illegal move is quietly refused you will see that the last one never changed. Both clocks start at 900,000 milliseconds — fifteen minutes, no increment — and they keep running while the dialog is open, so thinking costs you what it would over the board. Keeping track of the position is your job.

## Two files, one arbiter

These are not a full version and a cut-down one. Same game, same rules, same core — and **1,088 bytes is what that core really costs.** Everything on top of it exists for you, not for chess.

The 1,716 bytes `index.html` spends beyond the packed file buy a board you can see, pieces you can click, a clock that ticks, a promotion picker drawn onto the board, colours that tell you which squares are open. Strip all of it away and not one rule moves. Mate is still mate, en passant is still en passant, the fifty-move counter still fills on exactly the same ply.

So the two files answer two different questions. *What does it cost to referee a game of chess?* Eleven hundred bytes. *What does it cost to let a human enjoy one?* Another seventeen hundred.

That is a claim worth measuring rather than asserting, so both builds were run side by side. First their rule layers, move for move: 21,410 plies, with the legal-move list of every piece belonging to the side to play compared square by square, and the board, the side to move, the en passant square, the castling rights, the halfmove clock and the material verdicts compared after each one. 142,253 comparisons, not a single disagreement. Then fifty complete games, played by clicking in Chrome and replayed digit by digit through the `prompt()` box — 16,630 plies, the same result code every time. When both builds moved onto the 0x88 engine in October 2026 the second test was run again on the new pair: 30 games clicked out in Chrome, 10,377 plies, replayed digit by digit — the same result code and the same final board every time.

## Why lichess and not FIDE

Chess has a rulebook, and then chess servers have rulebooks of their own. They agree on almost everything and disagree in a handful of interesting places. This arbiter follows lichess deliberately, and the decision was made by reading lichess's own source rather than by guessing at its behaviour.

| ending | FIDE | lichess | here |
| --- | --- | --- | --- |
| checkmate, stalemate | — | — | ✓ |
| insufficient material | automatic | automatic | automatic |
| fivefold repetition | automatic | automatic | automatic |
| threefold repetition | **claim** | **claim** | **claim** |
| fifty moves | **claim** | **automatic** | **automatic** |
| seventy-five moves | automatic | *never reached* | *never reached* |
| draw by agreement | — | — | ✓ |
| flag fall | loss | loss | loss |
| flag fall, opponent cannot mate | **draw** | **draw** | **draw** |
| resignation | loss | loss | loss |
| resignation, opponent cannot mate | **draw** | **draw** | **draw** |

The bold rows are where the interesting part lives.

**Fifty moves.** FIDE lets a player *claim* the draw at fifty and has the arbiter declare it unasked at seventy-five. Lichess does not wait that long: `Variant.autoDraw` checks `halfMoveClock >= 100` and ends the game right there. Which means the seventy-five-move rule never gets a chance to fire on lichess, and it has no reason to exist here either. The game simply ends as `50`.

**Resigning against a bare king.** Article 5.1.2 says a resignation loses the game — *unless* the opponent could not deliver mate by any series of legal moves, in which case it is a draw. Most servers skip this. Lichess does not: in `RoundAsyncActor`, a resignation from a player who `cannotLose` is recorded as `InsufficientMaterialClaim`, a draw. This file does the same and calls it `RM`.

**Running out of time against a bare king.** Article 6.9, the same idea one article along. Lichess's `Finisher.outOfTime` hands over the win only `ifFalse(position.opponentHasInsufficientMaterial)`. Here that ending is `TM`.

**Blocked positions.** A position can be dead with pieces to spare: the pawns interlock, and mate becomes impossible even if both players set out to arrange one. Article 5.2.2 calls that a draw on the spot. Lichess does not look for it, and neither does this file — both play on until repetition or the fifty-move counter closes the game instead. Taking that detector out is most of what separates this build from its FIDE sibling on the project site.

**En passant, and the mistake nearly everyone makes.** For repetition, two positions count as the same only if a legal en passant capture is available in both or in neither. Plenty of engines write the en passant square after every double pawn push, whether or not anything can actually take. Nothing looks broken, because the capture gets refused anyway — but the repetition key comes out different, and a threefold that should have triggered arrives late or never arrives at all. Lichess gets this right: its position hash uses `enPassantSquare`, which has already been filtered through `isLegalEnPassant`, not the raw `potentialEpSquare`. So does this file, and it is stricter about it — the square goes in only after the legal move generator has confirmed that a neighbouring pawn really can take.

*Read from [scalachess](https://github.com/lichess-org/scalachess) (`Variant.scala`, `Position.scala`, `Hash.scala`) and [lila](https://github.com/lichess-org/lila) (`Finisher.scala`, `RoundGame.scala`) in September 2026. Lichess can change its mind later; this file cannot.*

## Result codes

Fourteen ways a game can end: six decisive, eight drawn.

| | | | |
| --- | --- | --- | --- |
| `W#` White mates | `B#` Black mates | `SM` stalemate | `IM` insufficient material |
| `WT` White wins on time | `BT` Black wins on time | `TM` flag fall, mate impossible | `50` fifty-move rule |
| `WR` White wins by resignation | `BR` Black wins by resignation | `RM` resignation, mate impossible | `5R` fivefold repetition |
| `3R` threefold claim | `DA` draw by agreement | | |

A code beginning with `W` is a win for White and one beginning with `B` a win for Black; everything ending in `M`, beginning with `D`, or carrying a digit is a draw. `numerical_packed.html` reports the same fourteen endings as numbers, `1` through `15`, with `4` left unused — that slot belonged to the seventy-five-move rule.

The FIDE build on the project site has fifteen codes and calls insufficient material `DP`, dead position, because it detects blocked ones as well. Here the code is `IM`, which names what the file actually tests.

## What's in it

- **Every piece's movement**, worked out with arithmetic on a 0x88 board. No direction tables, no offset arrays.
- **Full legality.** A move that would leave your own king in check is never let through.
- **Castling** on both wings, with all of it checked: the right still standing, the rook's path clear, the king not in check, not crossing an attacked square, not landing on one.
- **En passant**, including the repetition-key subtlety above.
- **Promotion** to queen, rook, bishop or knight — a picker on the board, a fifth digit without it.
- **A running clock.** Ten minutes plus five seconds a move on the board, fifteen flat minutes in the dialog.
- **Draw offers and claims**, resignation, flag fall, and the two impossible-mate exceptions that turn a loss into a draw.
- **All fourteen endings**, told apart from one another.

## What's not in it

- No blocked-position detection, as above. Lichess has none either.
- No engine, no takebacks, no FEN in or out, no PGN.
- No coordinates around the board. It turns around every ply, and the status line keeps out of the way.

For the full FIDE arbiter — fifteen codes, dead positions, eight front ends — see [fidelite.art](https://www.fidelite.art/).

## Read it in the browser

`index.html` is not packed and not minified. **Right-click → View Page Source** (`Ctrl` `U`, or `⌥` `⌘` `U` on macOS) puts the whole program on screen, and that is what this repository is for: open the page, play a few moves, then go and read the source that just refereed them.

`numerical_packed.html` is the exception. It unpacks itself, and there is a section further down on how to open it up.

---

## Anatomy

Every byte of `index.html`, by part.

| part | bytes | |
| --- | --- | --- |
| markup and CSS | 598 | board, panel, controls, colours, layout |
| `<script>` tags | 17 | |
| state | 122 | the 0x88 board, castling rights, en passant square, counters, selection |
| `C` | 29 | which castling right a square forfeits |
| `G` | 237 | can this piece reach that square |
| `T` | 53 | is this square attacked — the king's, unless told otherwise |
| `U` | 135 | make the move on a copy: promotion, en passant victim, rook hop |
| `L` | 55 | is this move legal — make it, ask, take it back |
| `M` | 110 | play it for real: counter, castling rights, side to move, en passant square, position key |
| `V` | 134 | can the other side still mate, on material alone |
| `s`, `J` | 52 | the verdict on flag fall and resignation, the claims |
| repetition table and clock | 42 | |
| `j` | 52 | the clock |
| aliases and setup | 109 | `innerHTML`, the piece glyphs, the 64 cells generated at load |
| `F` | 683 | draw the board, and the promotion picker on it |
| `E`, `K` | 345 | play the move and give the verdict, the buttons, the click |
| first draw and interval | 31 | |
| **total** | **2,804** | |

Sliced the other way (the way the project site colours the source, with the verdict after each move counted as rules), the rules come to **1,097** bytes and the page that shows them to **1,707**. The referee is cheap and the stage is expensive, which is exactly the argument the packed file makes.

The names are those of FideLite's `L3`: a letter does the same job in both, and `C`, `G`, `T`, `U`, `L` and `M` are byte for byte the functions in FideLite's `engine.js`. The table's id is `u` rather than `T`, because `T` is now the check test.

## Verification

Both builds were checked by running them rather than by reading them. `index.html` was driven through real Chrome over the DevTools protocol and again under jsdom; `numerical_packed.html` ran in Node against a stubbed `prompt()`.

- **perft**, on both builds, over the five standard positions: 197,281 from the starting position at depth 4; 97,862 from Kiwipete at depth 3; 43,238 from CPW position 3 at depth 4; 9,467 from position 4 at depth 3; 62,379 from position 5 at depth 3. The move generator is FideLite's, and FideLite's own suite runs it far deeper.
- **All fourteen endings**, each one reached and checked — `TM` and `RM` among them, and the automatic `50` firing on ply 100 exactly.
- **The two builds against each other**, twice over: 21,410 plies of step-by-step comparison on the rule layer, taking in 142,253 legal-move lists and the full state after every ply; then 50 complete games clicked out in Chrome and replayed digit by digit in the numerical build, 16,630 plies. Neither run turned up a difference.
- **Markup**: the W3C Nu Html Checker returns zero errors and zero warnings on `index.html`, before the 0x88 rewrite and after it. Standards mode, UTF-8, no BOM, not a single line break in the file.
- **Rendering**, cell by cell in Chrome: a 568×568 board of 71×71 squares, the last move and the selection tinted correctly on light and dark squares alike, the check glow on the king's square and nowhere else, and the promotion discs landing on the right four squares of the right file with all four pieces in the mover's colour, whatever stands underneath them.
- **Earlier rounds.** The switch to 0–63 square numbers and the rule optimizations of September and early October 2026 were each checked against the file before them, on the 8×8 engine both builds used at the time. The rewrite below replaced that engine.
- **The 0x88 rewrite of October 2026.** Both builds dropped their own 8×8 engine for FideLite's 0x88 one, written with `L3`'s names. The material test `V` is new — the material half of FideLite's dead-position test — and it was run against the old `I` and `H` on 600,000 random positions, 300,000 sparse and 300,000 crowded: the same insufficient-material verdict, and the same answer to *can the opponent still mate* for either side, every time. `index.html` was then run in Chrome side by side with its previous version, both pages clicked on the same screen squares. After every step the 64 cells (piece, colour, outline, background), the status line, both clocks, the offer light and the radio button were compared, and so were the board, the side to move, the castling rights, the counters, the repetition count and the offers: 6,080 comparisons over 18 games, from the starting position and from ten special positions (promotions, castling, en passant, bare kings, minor pieces), 17 promotions through the picker, 98 screenshots identical byte for byte. Scripted endings — `W#`, `B#`, `SM`, `TM`, `BT`, `RM`, `BR`, `IM`, and `50` on ply 100 — came out the same on both. `numerical_packed.html` ran in lock step against the previous file — the old one fed 0–63, the new one the same squares as rank and file digits, with illegal, off-board and junk input mixed in — over 180 games and 5,755 legal moves, comparing every dialog title and the result code: no difference, all fourteen codes reached. The sizes went from 3,018 to 2,838 bytes and from 1,201 to 1,118.
- **The engine update of October 2026.** Both builds took the rule optimizations of FideLite's new `engine.js`, so `C`, `G`, `T`, `U`, `L` and `M` in `index.html` are again its functions byte for byte: the pawn branch closes with `?…:0` and `>b[f]` instead of `&&(…)` and `&!b[f]`, castling asks `T()+T(i+s)<S()`, the rook hop sits in `U`'s return value, `d-8?d-1024?8:Q:U(r,Q)`, and the starting position is laid out with `for(j in s=…)` instead of a `.map()`. The material test writes `!m|!W>(…)` for `!m|!W&!(…)`. No rule changed. `numerical_packed.html` took the pawn branch, the castling test and the material test; the next section says why the other two stayed out. The sizes went from 2,838 to 2,828 bytes and from 1,118 to 1,113. Each build was run in lock step against its previous version. `index.html` in Node, the same squares clicked in both, comparing after every click the board, side to move, castling rights, en passant square, counters, repetition count, offers, selection, status line and every cell's piece and style: 24 games, 4,249 comparisons, 7 promotions through the picker, flag falls ending in `TM` and `BT`, and `IM` and `B#` endings. `numerical_packed.html` against a stubbed `prompt()`, with junk input, offers and resignations mixed in: 12 games, 1,395 inputs, the same result and final state every time. The material test `V` against its previous form on 20,000 random positions, both sides: no difference.
- **The renamed packing of October 2026.** `numerical_packed.html` was packed again after the letter search described under *Packing*, and it now carries the `for…in` setup and the rook hop as well. The size went from 1,113 to 1,088 bytes. The new file was run in lock step against the previous one, under a stubbed `prompt()` and a stubbed clock that now and then jumps far enough to drop a flag, with illegal, off-board and junk input, offers and resignations mixed in, and with captures favoured so that games run down to bare material: 360 games, 44,280 inputs, every dialog title and every result code the same — ten of the fourteen codes came up: both mates, `IM`, `DA`, both flag falls, both resignations, `TM` and `RM`. Unpacking the new file gives back its source exactly; RegPack 5.0.1 and 5.0.4 both rebuild it from that source byte for byte; and `reassignVars` changes nothing at any of the 350 settings.

## Unpacking

`numerical_packed.html` unpacks itself. Its script is a RegPack decompression loop that ends in `eval(_)`, so to get the plain source back you replace that one call:

```js
eval(_)   →   console.log(_)
```

Nothing in the loop touches the game, so it is safe to do this in Node. What falls out is 1,212 bytes of source.

**Download the file rather than copying it out of the browser.** The dictionary keys are control characters from the `\x01`–`\x1f` range, and the clipboard — or any editor that tidies up line endings — will quietly destroy them.

## Packing

[RegPack 5.0.4](https://github.com/Siorki/RegPack); 5.0.1 produces the identical file. These settings rebuild `numerical_packed.html` from that source **byte for byte**:

| option | value |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0.5` |
| `crushLengthFactor` | `2` |
| `crushCopiesFactor` | `1` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Stage 2 wins, the regexp character class: `[\x01-\x1fP-Y]`, 41 tokens. The bytes land like this:

```
   8 B  <script>
1071 B  packed payload
   9 B  </script>
----
1088 B
```

Turning `reassignVars` on changes nothing: 1,088 either way, at every one of the 350 settings below. The renaming it would do has already been done, as the next section explains.

350 crusher combinations were tried — gain 0 to 3, length 0 to 2, copies 0 to 2 in steps of a half, tiebreaker 0 or 1. Two of them tie at 1,071; the settings above are the first in that order, and the worst of the 350 lands at 1,101.

### The packed source is renamed for the packer

The source above runs to 1,212 bytes, and it is not the source anyone would write by hand. It began as the same 0x88 core `index.html` runs, with `L3_numerical`'s dialog around it, and a straight port packs to 1,153. Then it was cut down for this build alone. There is no search to feed, so the legal-move iterator `O` and the verdict function `Z` are folded into the one place each is used; the clocks live on `G` beside the position keys; `U` no longer records `X`, which only the clickable build reads; and the offer path no longer works out a verdict the move has already given. That takes the payload to 1,108.

Then the names were moved. RegPack's dictionary tokens are characters the source does not use, and every capital letter a name takes up is one the packer cannot have. So the names went onto lower-case letters, shared out by scope wherever two of them never meet. No name is left on a capital: the only capitals in the source are the `D` of `Date` and the `O` of `indexOf`. That leaves capitals free for the dictionary — RegPack takes `P`–`S`, four more tokens — and brings the payload to 1,101.

The engine update of October 2026 — the pawn branch closed with `?…:0` and `>b[f]`, the castling test `p()+p(i+f)<a()` and the material test's `!k>` — took the payload to 1,096. Its other two items stayed out at first, because on those names they packed larger: the starting position laid out by `for…in` by 3 bytes, the rook hop folded into `m`'s return value by 2.

Last, the letters were handed out again, by search rather than by hand. Every name — each global, and each parameter inside its own function — was tried on every letter it could take: the lower-case ones, `_` and `$`; the capitals stay with the dictionary. A move was kept only if the file then packed smaller, and only if the program still meant the same thing: after every move the source was parsed again and every identifier had to point to the same variable as before, so two names never land on one letter where both are in reach. Single moves and pairwise swaps took turns until neither found anything more. The search lands on letters RegPack likes better, and on those letters the two items that stayed out pay after all: with `for…in` and the rook hop the source is 1,212 bytes and packs to 1,071; without them, searched the same way, 1,216 bytes pack to 1,073. So both are in. The dictionary now takes `P`–`Y`, ten capitals. The names stand like this:

| straight port (`L3_numerical`'s names) | packed source | |
| --- | --- | --- |
| `G` | `a` | reach; here also the repetition table and the clocks |
| `T` | `p` | is this square attacked |
| `U` | `m` | make the move on a copy |
| `L` | `l` | is this move legal |
| `M` | `s` | play it |
| `V` | `t` | can the other side still mate |
| `C` | `j` | castling rights |
| `R` | `w` | how many times this position has come up |
| `t` | `q` | the side to move |
| `K`, `Y` | `d`, `x` | the typed move, the target square |
| `P`, `S` in `G`; `P`, `Q` in `U` | `z`, `a`; `a`, `d` | piece type, the square walker; piece type, the square in between |
| `A`, `W` in `V` | `a`, `k` | material by side, pawns on the board |

Two variables could also share one letter outright, wherever one is finished with before the other starts. That was tried and left out: test games cannot prove that two variables are never alive at the same time, and one fold that passed them had put the fifty-move counter, the position key and the repetition count on the same letter. RegPack's own `reassignVars` finds nothing left to do.

---

## Related

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — the full FIDE arbiter, fifteen result codes, eight front ends
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — the other direction entirely: dead positions carried to 99.97%
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — the same rules a level down, letters and numbers side by side
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — the bare rules, packed, in 1,023 bytes

## License

MIT

---
---

<a id="turkce"></a>

# Lichess-equivalent (Türkçe)

FIDE'nin değil, **lichess'in kurallarıyla** hükmeden bir satranç hakemi; tek bir HTML dosyasında **2.804 bayt** — ve aynı hakemin tahtasından soyulmuş hâli, **1.088 baytta**. İki oyuncu, tek ekran. Kütüphane yok, derleme adımı yok, sunucu yok. Dosyayı indirin, çift tıklayın, oynayın.

İki kural kitabı birbirine yakındır ama aynı kitap değildir; yolların ayrıldığı her nokta aşağıda tek tek yazılı.

[Golfstack](https://www.fidelite.art/) projesinin bir parçasıdır.

## Oyna

| dosya | arayüz | bayt | GitHub Pages | proje sitesi |
| --- | --- | --- | --- | --- |
| `index.html` | tıklanabilir tahta, saat, lichess renkleri | 2.804 | [Lichess-equivalent](https://cuneytinann.github.io/Lichess-equivalent/) | [Lichess-equivalent.html](https://www.fidelite.art/special/Lichess-equivalent.html) |
| `numerical_packed.html` | `prompt()` kutusuna yazılan kare numaraları, tahta yok | 1.088 | [numerical_packed.html](https://cuneytinann.github.io/Lichess-equivalent/numerical_packed.html) | [Lichess-eqv_min.html](https://www.fidelite.art/builds/special/Lichess-eqv_min.html) |

Proje sitesinde iki sürüm de `special/` altında, `L1`–`L3` merdiveninin bir kenarında durur. O merdivenin bir basamağı değildirler; başka bir kural kitabını izlerler.

2020 sonu ve sonrasının her tarayıcısı çalıştırır: Chrome 85+, Firefox 79+, Safari 14+. Bu tabanı üç şey belirliyor — başlangıç dizilişinin yazıldığı BigInt; dar ekranda tahtanın erişilebilir kalmasını sağlayan `place-content`'teki `safe` anahtar sözcüğü; ve paketli sürümün kullandığı `||=` işleci.

**Tahtayla.** Bir taşa tıklayın, sonra gitmesini istediğiniz kareye. Yasal kareler birer nokta alır, alabileceğiniz taş bir halka; çıktığınız ve indiğiniz kare boyalı kalır; şah altındaki şahın karesi kırmızı parlar. Her hamleden sonra tahta dönüp sırası gelene bakar. Bir piyon son yatayı bulduğunda indiği sütun seçiciye dönüşür: dört terfi seçeneği tahtanın üstünde, her biri gri bir diskin içinde alt alta durur, istediğinize tıklarsınız. `½` beraberlik teklif eder, hakkınız varsa talep eder — ve cevap sırası sizdeyken ya da konum üçüncü kez geldiğinde kendiliğinden yanar; `⚐` terk eder. İki denetim yan yana ve bilerek birbirine benzer: aynı boy, renk üstüne beyaz, beraberlikte yeşil, terkte kırmızı. Saat on dakikadan başlar ve her hamlede beş saniye geri verir.

**Tahtasız.** `numerical_packed.html` hiçbir şey çizmez. Bir kare iki basamaktır, önce yatay sonra sütun, ikisi de 0'dan sayılır: a1 `00`, e2 `14`, h8 `77` — motorun kendi 0x88 indeksi, on altılık tabanda yazılmış hâliyle. Hamle, iki karenin uç uca yazılmış hâlidir: e2–e4 `1434` olur. Basamaklardan birinde `8` ya da `9` tahtanın dışını gösterir; hakem onu da her yasadışı hamle gibi reddeder. Terfi için beşinci bir basamak eklersiniz — `0` fil, `1` kale, `2` at, başka bir şey vezir. Beraberlik de aynı yoldan teklif edilir: hamlenin sonuna rakam olmayan bir karakter iliştirirsiniz, `1434x` yazmak e2–e4'ü teklifle birlikte oynar. O karakteri tek başına yazarsanız teklif yine geçerlidir ama sıra sizde kalır, arkasından bir hamle borcunuz olur. Rakip aynı şekilde yanıt verirse beraberlik olur; düz bir hamle oynarsa teklifi reddetmiş ve silmiş olur. Konum daha önce üç kez oluştuysa aynı karakter üçlü tekrarı doğrudan talep eder. Terk etmek için kutuyu boş bırakın ya da İptal'e basın.

İki sürümün ayrıştığı tek yer burası ve bu bir kural farkı değil, bir jest farkı. Tahtada beraberlik teklifi `½` ile kaldırıp yine `½` ile indirdiğiniz, havada duran bir bayraktır; iletişim kutusunda ise hamleye binerek gider. İki durumda da hakem aynı iki biti okur ve iki durumda da rakibin düz hamlesi teklifi reddeder. İletişim kutusunun başlığında Beyaz'ın saati, Siyah'ın saati ve hakemden geçirebildiğiniz son hamle durur; yasadışı bir hamle sessizce reddedildiğinde sonuncunun hiç değişmediğini oradan anlarsınız. İki saat de 900.000 milisaniyeden başlar — artırımsız on beş dakika — ve kutu açıkken işlemeyi sürdürür, yani düşünmek size tahta başındaki kadara mal olur. Konumu takip etmek ise tümüyle sizin işiniz.

## İki dosya, tek hakem

Bunlar tam sürüm ile kırpılmış sürüm değil. Aynı oyun, aynı kurallar, aynı öz — ve **o özün gerçek bedeli 1.088 bayt.** Üstüne binen her şey satranç için değil, sizin için var.

`index.html`'in paketli dosyanın ötesinde harcadığı 1.716 bayt şunu satın alıyor: gördüğünüz bir tahta, tıkladığınız taşlar, işleyen bir saat, tahtanın üstüne çizilen bir terfi seçicisi, hangi karenin açık olduğunu söyleyen renkler. Hepsini soyun, tek bir kural yerinden oynamaz. Mat yine mattır, geçerken alma yine geçerken almadır, elli hamle sayacı yine tam aynı yarım hamlede dolar.

Yani iki dosya iki ayrı soruya yanıt veriyor. *Bir satranç oyununu yönetmenin bedeli nedir?* Bin yüz bayt. *Bir insanın o oyundan keyif almasının bedeli nedir?* Bin yedi yüz bayt daha.

Bu, iddia edilmektense ölçülmeyi hak eden bir cümle; o yüzden iki sürüm yan yana koşturuldu. Önce kural katmanları, hamle hamle: 21.410 yarım hamle boyunca sırası gelen tarafın her taşının yasal hamle listesi kare kare karşılaştırıldı, her hamleden sonra da tahta, sıra, geçerken alma karesi, rok hakları, yarım hamle sayacı ve materyal hükümleri. 142.253 karşılaştırma, tek bir ayrılık yok. Sonra elli tam oyun: Chrome'da tıklanarak oynandı ve `prompt()` kutusuna basamak basamak yeniden girildi — 16.630 yarım hamle, her seferinde aynı sonuç kodu. Ekim 2026'da iki sürüm de 0x88 motoruna geçince ikinci sınama yeni çift üzerinde yeniden yapıldı: Chrome'da tıklanarak oynanan 30 oyun, 10.377 yarım hamle, basamak basamak yeniden girildi — her seferinde aynı sonuç kodu, aynı son tahta.

## Neden lichess, neden FIDE değil

Satrancın bir kural kitabı vardır, satranç sunucularının da kendi kural kitapları. Neredeyse her şeyde anlaşır, birkaç ilginç noktada ayrılırlar. Bu hakem bilerek lichess'i izliyor ve bu karar, davranışını tahmin ederek değil, lichess'in kendi kaynağı okunarak verildi.

| bitiş | FIDE | lichess | burada |
| --- | --- | --- | --- |
| mat, pat | — | — | ✓ |
| yetersiz materyal | otomatik | otomatik | otomatik |
| beş kez tekrar | otomatik | otomatik | otomatik |
| üç kez tekrar | **talep** | **talep** | **talep** |
| elli hamle | **talep** | **otomatik** | **otomatik** |
| yetmiş beş hamle | otomatik | *hiç ulaşılmaz* | *hiç ulaşılmaz* |
| anlaşmalı beraberlik | — | — | ✓ |
| süre bitimi | yenilgi | yenilgi | yenilgi |
| süre bitimi, rakip mat edemiyor | **beraberlik** | **beraberlik** | **beraberlik** |
| terk | yenilgi | yenilgi | yenilgi |
| terk, rakip mat edemiyor | **beraberlik** | **beraberlik** | **beraberlik** |

İşin ilginç yanı kalın satırlarda.

**Elli hamle.** FIDE ellinci hamlede oyuncuya *talep* hakkı tanır, yetmiş beşinci hamlede ise hakem sormadan ilan eder. Lichess o kadar beklemez: `Variant.autoDraw` doğrudan `halfMoveClock >= 100` koşuluna bakar ve oyunu orada bitirir. Dolayısıyla yetmiş beş hamle kuralı lichess'te tetiklenme fırsatı bile bulamaz; burada da bulunmasının bir anlamı yok. Oyun yalnızca `50` ile kapanır.

**Çıplak şaha karşı terk.** 5.1.2. madde terkin oyunu kaybettirdiğini söyler — *ancak* rakip hiçbir yasal hamle dizisiyle mat edemiyorsa beraberliktir. Çoğu sunucu bu ayrıntıyı atlar. Lichess atlamaz: `RoundAsyncActor`'da `cannotLose` durumundaki bir oyuncunun terki `InsufficientMaterialClaim` olarak, yani beraberlik olarak kaydedilir. Bu dosya da aynısını yapıyor ve buna `RM` diyor.

**Çıplak şaha karşı süre bitimi.** 6.9. madde, bir madde ötede aynı fikir. Lichess'in `Finisher.outOfTime` işlevi galibiyeti yalnızca `ifFalse(position.opponentHasInsufficientMaterial)` koşuluyla teslim eder. Buradaki karşılığı `TM`.

**Kilitli pozisyonlar.** Bir konum, taş fazlasıyla yeterliyken bile ölü olabilir: piyonlar birbirine kenetlenir ve iki oyuncu mat kurmak için anlaşsa bile mat kurulamaz. 5.2.2. madde bunu olduğu anda beraberlik sayar. Lichess bunu aramaz, bu dosya da aramaz — ikisi de tekrar ya da elli hamle sayacı oyunu kapatana kadar oynamayı sürdürür. Bu sürümü proje sitesindeki FIDE kardeşinden ayıran şeyin büyük bölümü, o dedektörün sökülmüş olması.

**Geçerken alma ve neredeyse herkesin düştüğü hata.** Tekrar açısından iki konum, ancak yasal bir geçerken alma imkânı ikisinde de varsa ya da ikisinde de yoksa aynı sayılır. Pek çok motor, gerçekten alabilen bir piyon olsun olmasın, her çift adımdan sonra geçerken alma karesini yazar. Dışarıdan bozuk bir şey görünmez, çünkü alma zaten reddedilir — ama tekrar anahtarı farklı çıkar ve tetiklenmesi gereken üçlü tekrar ya geç gelir ya hiç gelmez. Lichess bu ayrımı doğru yapıyor: konum karması, ham `potentialEpSquare`'i değil, `isLegalEnPassant` süzgecinden geçmiş `enPassantSquare`'i kullanıyor. Bu dosya da öyle, hatta biraz daha katı davranıyor — kare, ancak yasal hamle üreticisi komşu bir piyonun gerçekten alabildiğini onayladıktan sonra yazılıyor.

*Eylül 2026'da [scalachess](https://github.com/lichess-org/scalachess) (`Variant.scala`, `Position.scala`, `Hash.scala`) ve [lila](https://github.com/lichess-org/lila) (`Finisher.scala`, `RoundGame.scala`) dosyalarından okundu. Lichess ileride fikrini değiştirebilir; bu dosya değiştiremez.*

## Sonuç kodları

Bir oyunun bitebileceği on dört yol: altısı kesin sonuç, sekizi beraberlik.

| | | | |
| --- | --- | --- | --- |
| `W#` Beyaz mat eder | `B#` Siyah mat eder | `SM` pat | `IM` yetersiz materyal |
| `WT` Beyaz süreyle kazanır | `BT` Siyah süreyle kazanır | `TM` süre bitti, mat imkânsız | `50` elli hamle kuralı |
| `WR` Beyaz terkle kazanır | `BR` Siyah terkle kazanır | `RM` terk, mat imkânsız | `5R` beş kez tekrar |
| `3R` üç kez tekrar talebi | `DA` anlaşmalı beraberlik | | |

`W` ile başlayan kod Beyaz'ın, `B` ile başlayan Siyah'ın galibiyetidir; `M` ile biten, `D` ile başlayan ya da rakam taşıyan her şey beraberliktir. `numerical_packed.html` aynı on dört bitişi sayı olarak bildirir, `1`'den `15`'e; `4` boş kalır — o yuva yetmiş beş hamle kuralınındı.

Proje sitesindeki FIDE sürümünde on beş kod vardır ve yetersiz materyale `DP`, yani ölü pozisyon der; çünkü kilitli olanları da tespit eder. Burada kod `IM` — dosyanın gerçekten sınadığı şeyin adı.

## İçinde neler var

- **Bütün taş hareketleri**, 0x88 tahtada aritmetikle çıkarılmış. Yön tablosu yok, ofset dizisi yok.
- **Tam yasallık.** Kendi şahınızı açıkta bırakacak bir hamle hiçbir zaman geçmez.
- **İki kanatta rok**, hepsi denetlenerek: hak hâlâ duruyor mu, kalenin yolu boş mu, şah tehdit altında mı, geçtiği kare tehdit altında mı, indiği kare tehdit altında mı.
- **Geçerken alma**, yukarıdaki tekrar anahtarı inceliğiyle birlikte.
- **Terfi**: vezir, kale, fil, at — tahtada bir seçiciyle, tahtasız sürümde beşinci basamakla.
- **İşleyen bir saat.** Tahtada on dakika artı hamle başına beş saniye, iletişim kutusunda artırımsız on beş dakika.
- **Beraberlik teklifi ve talebi**, terk, süre bitimi ve yenilgiyi beraberliğe çeviren iki mat-imkânsızlığı istisnası.
- **On dört bitişin hepsi**, birbirinden ayrılmış hâlde.

## İçinde neler yok

- Kilitli pozisyon tespiti yok, yukarıda anlattığımız gibi. Lichess'te de yok.
- Motor yok, geri alma yok, FEN alışverişi yok, PGN yok.
- Tahtanın çevresinde koordinat yok. Tahta her yarım hamlede dönüyor, durum satırı da yoldan çekiliyor.

Tam FIDE hakemi için — on beş kod, ölü pozisyonlar, sekiz arayüz — [fidelite.art](https://www.fidelite.art/) adresine bakın.

## Tarayıcıda okuyun

`index.html` paketlenmiş de küçültülmüş de değil. **Sağ tık → Sayfa Kaynağını Görüntüle** (`Ctrl` `U`, macOS'ta `⌥` `⌘` `U`) programın tamamını ekrana getirir; bu depo zaten bunun için var: sayfayı açın, birkaç hamle oynayın, sonra o hamleleri az önce yöneten kaynağı okuyun.

`numerical_packed.html` bunun istisnası. Kendi kendini açar; nasıl açılacağı aşağıda anlatılıyor.

---

## Anatomi

`index.html`'in her baytı, parça parça.

| parça | bayt | |
| --- | --- | --- |
| markup ve CSS | 598 | tahta, panel, denetimler, renkler, yerleşim |
| `<script>` etiketleri | 17 | |
| durum | 122 | 0x88 tahta, rok hakları, geçerken alma karesi, sayaçlar, seçim |
| `C` | 29 | bir karenin düşürdüğü rok hakkı |
| `G` | 237 | bu taş o kareye ulaşabilir mi |
| `T` | 53 | bu kare tehdit altında mı — aksi söylenmedikçe şahın karesi |
| `U` | 135 | hamleyi bir kopyada yap: terfi, geçerken alınan piyon, kale sıçraması |
| `L` | 55 | bu hamle yasal mı — yap, sor, geri al |
| `M` | 110 | hamleyi gerçekten oyna: sayaç, rok hakları, sıra, geçerken alma karesi, konum anahtarı |
| `V` | 134 | öbür taraf hâlâ mat edebilir mi, yalnızca materyale bakarak |
| `s`, `J` | 52 | süre bitimi ve terk hükmü, talepler |
| tekrar tablosu ve saat | 42 | |
| `j` | 52 | saat |
| takma adlar ve kurulum | 109 | `innerHTML`, taş glifleri, açılışta üretilen 64 hücre |
| `F` | 683 | tahtayı ve üstündeki terfi seçicisini çiz |
| `E`, `K` | 345 | hamleyi oyna ve hükmü ver, düğmeler, tıklama |
| ilk çizim ve interval | 31 | |
| **toplam** | **2.804** | |

Başka türlü bölersek (proje sitesinin kaynağı boyadığı gibi, her hamleden sonraki hüküm de kurala sayılarak) kurallar **1.097** bayt, onları gösteren sayfa **1.707**. Hakem ucuz, sahne pahalı — paketli dosyanın öne sürdüğü şey tam olarak bu.

Adlar FideLite'ın `L3`'ündeki adlar: bir harf ikisinde de aynı işi görüyor; `C`, `G`, `T`, `U`, `L` ve `M` de FideLite'ın `engine.js`'indeki işlevlerin bayt bayt aynısı. Tablonun kimliği `T` değil `u`, çünkü `T` artık şah denetimi.

## Doğrulama

İki sürüm de okunarak değil, koşturularak sınandı. `index.html` hem DevTools protokolü üzerinden gerçek Chrome'da hem de jsdom altında sürüldü; `numerical_packed.html` Node'da, taklit bir `prompt()` karşısında çalıştı.

- **perft**, iki sürümde de, beş standart pozisyon üzerinde: başlangıç dizilişinden 4. derinlikte 197.281; Kiwipete'ten 3. derinlikte 97.862; CPW 3. pozisyondan 4. derinlikte 43.238; 4. pozisyondan 3. derinlikte 9.467; 5. pozisyondan 3. derinlikte 62.379. Hamle üretici FideLite'ınki; FideLite'ın kendi test takımı onu çok daha derine sürüyor.
- **On dört bitişin hepsi**, tek tek üretilip sınandı — aralarında `TM` ile `RM` ve tam 100. yarım hamlede tetiklenen otomatik `50` de var.
- **İki sürüm birbirine karşı**, iki ayrı biçimde: kural katmanında 21.410 yarım hamlelik adım adım karşılaştırma, 142.253 yasal hamle listesi ve her hamleden sonra tam durum; ardından Chrome'da tıklanarak oynanıp sayısal sürümde basamak basamak tekrarlanan 50 tam oyun, 16.630 yarım hamle. İki koşuda da fark çıkmadı.
- **Markup**: W3C Nu Html Checker `index.html` için sıfır hata, sıfır uyarı veriyor; 0x88'e geçişten önce de sonra da. Standart mod, UTF-8, BOM yok, dosyada tek bir satır sonu bile yok.
- **Çizim**, Chrome'da hücre hücre: 71×71 karelerden oluşan 568×568 tahta; son hamle ve seçim açık ve koyu karelerde ayrı ayrı doğru tonda; şah parlaması yalnız şahın karesinde; terfi diskleri doğru sütunun doğru dört karesine oturuyor ve dört taş da altlarında ne olursa olsun hamleyi yapanın renginde çıkıyor.
- **Önceki turlar.** 0–63 kare numaralarına geçiş ile Eylül ve Ekim başı 2026 kural optimizasyonları, her biri kendinden önceki dosyaya karşı, iki sürümün o sırada kullandığı 8×8 motor üzerinde sınandı. Aşağıdaki yeniden yazım o motorun yerini aldı.
- **Ekim 2026 0x88 yeniden yazımı.** İki sürüm de kendi 8×8 motorunu bırakıp FideLite'ın 0x88 motoruna geçti, `L3`'ün adlarıyla. Materyal sınaması `V` yeni — FideLite'ın ölü pozisyon sınamasının materyal yarısı — ve eski `I` ile `H`'ye karşı 600.000 rastgele pozisyonda koşturuldu, 300.000'i seyrek, 300.000'i kalabalık: yetersiz materyal hükmü de, iki taraf için *rakip hâlâ mat edebilir mi* yanıtı da her seferinde aynı. Ardından `index.html` Chrome'da önceki sürümüyle yan yana koşturuldu; iki sayfaya da ekranın aynı karelerinden tıklandı. Her adımdan sonra 64 hücre (taş, renk, kontur, arka plan), durum satırı, iki saat, teklif ışığı ve radyo düğmesi karşılaştırıldı; tahta, sıra, rok hakları, sayaçlar, tekrar sayısı ve teklifler de: başlangıç dizilişinden ve on özel pozisyondan (terfiler, rok, geçerken alma, çıplak şahlar, hafif taşlar) 18 oyunda 6.080 karşılaştırma, seçici üzerinden 17 terfi, bayt bayt aynı 98 ekran görüntüsü. Senaryolu bitişler — `W#`, `B#`, `SM`, `TM`, `BT`, `RM`, `BR`, `IM` ve tam 100. yarım hamlede `50` — ikisinde de aynı çıktı. `numerical_packed.html` önceki dosyayla kilit adımlı koşturuldu — eskisine 0–63, yenisine aynı kareler yatay ve sütun basamaklarıyla verildi, araya yasadışı, tahta dışı ve anlamsız girdiler karıştırıldı — 180 oyunda 5.755 yasal hamle boyunca her iletişim kutusu başlığı ve sonuç kodu karşılaştırıldı: fark yok, on dört kodun hepsine ulaşıldı. Boyutlar 3.018'den 2.838 bayta, 1.201'den 1.118'e indi.
- **Ekim 2026 motor güncellemesi.** İki sürüm de FideLite'ın yeni `engine.js`'inin kural optimizasyonlarını aldı; böylece `index.html`'deki `C`, `G`, `T`, `U`, `L` ve `M` yine onun işlevlerinin bayt bayt aynısı: piyon dalı `&&(…)` ve `&!b[f]` yerine `?…:0` ve `>b[f]` ile kapanıyor, rok `T()+T(i+s)<S()` diye soruyor, kale sıçraması `U`'nun dönüş değerinde duruyor (`d-8?d-1024?8:Q:U(r,Q)`), başlangıç konumu `.map()` yerine `for(j in s=…)` ile yerleşiyor. Materyal sınaması `!m|!W&!(…)` yerine `!m|!W>(…)` yazıyor. Hiçbir kural değişmedi. `numerical_packed.html` piyon dalını, rok testini ve materyal sınamasını aldı; öbür ikisinin neden dışarıda kaldığını bir sonraki bölüm anlatıyor. Boyutlar 2.838 bayttan 2.828'e, 1.118'den 1.113'e indi. Her sürüm bir öncekiyle adım adım koşturuldu. `index.html` Node'da, ikisinde de aynı karelere tıklanarak; her tıklamadan sonra tahta, sıra, rok hakları, geçerken alma karesi, sayaçlar, tekrar sayısı, teklifler, seçim, durum satırı ve her hücrenin taşı ve stili karşılaştırıldı: 24 oyun, 4.249 karşılaştırma, seçiciyle 7 terfi, `TM` ve `BT` ile biten bayrak düşüşleri, `IM` ve `B#` bitişleri. `numerical_packed.html` taklit bir `prompt()`'a karşı, arada anlamsız girdiler, teklifler ve terklerle: 12 oyun, 1.395 girdi, her seferinde aynı sonuç ve aynı son durum. Materyal sınaması `V` önceki biçimine karşı 20.000 rastgele pozisyonda, iki taraf için: fark yok.
- **Ekim 2026 yeniden adlandırılmış paket.** `numerical_packed.html`, *Paketleme* bölümünde anlatılan harf aramasından sonra yeniden paketlendi; artık `for…in` yerleşimini ve kale sıçramasını da taşıyor. Boyut 1.113 bayttan 1.088'e indi. Yeni dosya öncekiyle kilit adımlı koşturuldu: taklit bir `prompt()` ve arada bir bayrak düşürecek kadar ileri sıçrayan taklit bir saat karşısında, araya yasadışı, tahta dışı ve anlamsız girdiler, teklifler ve terkler karıştırılarak, alışlar öne çıkarılarak ki oyunlar çıplak materyale kadar insin: 360 oyun, 44.280 girdi, her iletişim kutusu başlığı ve her sonuç kodu aynı — on dört kodun onu görüldü: iki mat, `IM`, `DA`, iki bayrak düşüşü, iki terk, `TM` ve `RM`. Yeni dosyanın paketi açılınca kaynağı birebir geri geliyor; RegPack 5.0.1 de 5.0.4 de o kaynaktan dosyayı bayt bayt aynı kuruyor; `reassignVars` da 350 ayarın hiçbirinde bir şey değiştirmiyor.

## Paketi açma

`numerical_packed.html` kendi kendini açar. Betiği `eval(_)` ile biten bir RegPack açma döngüsüdür; düz kaynağı geri almak için o tek çağrıyı değiştirmeniz yeter:

```js
eval(_)   →   console.log(_)
```

Döngünün içinde oyuna dokunan hiçbir şey yok, bunu Node'da yapmak güvenli. Dökülen şey 1.212 baytlık kaynak.

**Dosyayı tarayıcıdan kopyalamak yerine indirin.** Sözlük anahtarları `\x01`–`\x1f` aralığındaki denetim karakterleridir; pano ya da satır sonlarını derleyip toplayan herhangi bir editör onları sessizce yok eder.

## Paketleme

[RegPack 5.0.4](https://github.com/Siorki/RegPack); 5.0.1 birebir aynı dosyayı üretiyor. Şu ayarlar `numerical_packed.html`'i o kaynaktan **bayt bayt** yeniden kurar:

| ayar | değer |
| --- | --- |
| `reassignVars` | `false` |
| `crushGainFactor` | `0.5` |
| `crushLengthFactor` | `2` |
| `crushCopiesFactor` | `1` |
| `crushTiebreakerFactor` | `0` |
| `useES6` | `true` |

Kazanan aşama 2, yani düzenli ifade karakter sınıfı: `[\x01-\x1fP-Y]`, 41 token. Baytlar şöyle yerleşiyor:

```
   8 B  <script>
1071 B  paketli yük
   9 B  </script>
----
1088 B
```

`reassignVars`'ı açmak hiçbir şey değiştirmiyor: aşağıdaki 350 ayarın her birinde iki durumda da 1.088. Onun yapacağı yeniden adlandırma zaten yapıldı; bir sonraki bölüm anlatıyor.

350 crusher kombinasyonu denendi — gain 0–3, length 0–2, copies 0–2 yarımlık adımlarla, tiebreaker 0 ya da 1. İkisi 1.071'de berabere; yukarıdaki ayar bu sıradaki ilki, 350'nin en kötüsü 1.101'e iniyor.

### Paket kaynağı paketleyici için yeniden adlandırıldı

Yukarıdaki kaynak 1.212 bayt tutuyor ve kimsenin elle yazacağı bir kaynak değil. `index.html`'in koşturduğu aynı 0x88 özüyle, çevresinde `L3_numerical`'ın iletişim kutusuyla başladı; düz bir aktarım 1.153'e paketleniyor. Sonra yalnız bu sürüm için kırpıldı. Besleyecek bir arama yok; bu yüzden yasal hamle gezgini `O` ile hüküm işlevi `Z`, her biri kullanıldığı tek yere gömüldü; saatler konum anahtarlarının yanında, `G`'nin üstünde duruyor; `U` artık yalnız tıklanan sürümün okuduğu `X`'i yazmıyor; teklif yolu da hamlenin zaten verdiği hükmü bir daha hesaplamıyor. Bu, yükü 1.108'e indiriyor.

Sonra adlar taşındı. RegPack'in sözlük tokenleri kaynağın kullanmadığı karakterlerdir ve bir adın tuttuğu her büyük harf, paketleyicinin alamadığı bir harftir. Bu yüzden adlar küçük harflere taşındı; hiç karşılaşmayan adlar aynı harfi kapsama göre paylaştı. Büyük harfte hiçbir ad kalmadı: kaynaktaki büyük harfler yalnız `Date`'in `D`'si ile `indexOf`'un `O`'su. Büyük harfler sözlüğe kalıyor — RegPack `P`–`S`'yi, dört token daha, alıyor — ve yük 1.101'e iniyor.

Ekim 2026 motor güncellemesi — piyon dalının `?…:0` ve `>b[f]` ile kapanması, `p()+p(i+f)<a()` rok testi ve materyal sınamasının `!k>`'sı — yükü 1.096'ya indirdi. Güncellemenin öbür iki maddesi önce dışarıda kaldı, çünkü o adlarla daha büyük paketleniyorlardı: başlangıç konumunu `for…in` ile yerleştirmek 3 bayt, kale sıçramasını `m`'nin dönüş değerine katlamak 2 bayt.

Son olarak harfler bir kez daha dağıtıldı, bu sefer elle değil, aramayla. Her ad — her global ve her işlevin kendi parametreleri — alabileceği her harfte denendi: küçük harfler, `_` ve `$`; büyük harfler sözlükte kalıyor. Bir taşıma ancak dosya daha küçük paketleniyorsa ve program hâlâ aynı şeyi söylüyorsa tutuldu: her taşımadan sonra kaynak yeniden ayrıştırıldı ve her tanımlayıcının öncekiyle aynı değişkene işaret etmesi şart koşuldu; böylece ikisinin de erişebildiği bir yerde iki ad asla aynı harfe düşmüyor. Tekli taşımalar ve ikili takaslar, ikisi de bir şey bulamayana dek sırayla döndü. Arama RegPack'in daha çok sevdiği harflere varıyor ve o harflerle dışarıda kalan iki madde de kazandırıyor: `for…in` ve kale sıçramasıyla kaynak 1.212 bayt ve 1.071'e paketleniyor; onlarsız, aynı aramadan geçmiş 1.216 bayt 1.073'e. Bu yüzden ikisi de içeride. Sözlük artık `P`–`Y`'yi, on büyük harfi alıyor. Adlar şöyle duruyor:

| düz aktarım (`L3_numerical`'ın adları) | paket kaynağı | |
| --- | --- | --- |
| `G` | `a` | ulaşım; burada ayrıca tekrar tablosu ve saatler |
| `T` | `p` | bu kare tehdit altında mı |
| `U` | `m` | hamleyi bir kopyada yap |
| `L` | `l` | bu hamle yasal mı |
| `M` | `s` | oyna |
| `V` | `t` | öbür taraf hâlâ mat edebilir mi |
| `C` | `j` | rok hakları |
| `R` | `w` | bu konumun kaçıncı kez geldiği |
| `t` | `q` | sıradaki taraf |
| `K`, `Y` | `d`, `x` | yazılan hamle, hedef kare |
| `G`'de `P`, `S`; `U`'da `P`, `Q` | `z`, `a`; `a`, `d` | taş türü, kare yürüyücüsü; taş türü, aradaki kare |
| `V`'de `A`, `W` | `a`, `k` | taraf başına materyal, tahtada piyon var mı |

İki değişken, biri işini bitirmeden öbürü başlamıyorsa, aynı harfi doğrudan da paylaşabilirdi. Bu denendi ve dışarıda bırakıldı: deneme oyunları iki değişkenin hiçbir zaman aynı anda canlı olmadığını kanıtlayamaz, ve onlardan geçen bir birleştirme elli hamle sayacını, konum anahtarını ve tekrar sayısını aynı harfe koymuştu. RegPack'in kendi `reassignVars`'ı yapacak bir şey bulamıyor.

---

## İlgili

- [FideLite](https://github.com/cuneytinann/FideLite) · [fidelite.art](https://www.fidelite.art/) — tam FIDE hakemi, on beş sonuç kodu, sekiz arayüz
- [Chess LUX](https://github.com/cuneytinann/Chess_LUX) — tam ters yön: ölü pozisyonlar %99,97'ye taşınmış
- [chessarbiter2kb](https://github.com/cuneytinann/chessarbiter2kb) — aynı kurallar bir seviye aşağıda, harfler ve sayılar yan yana
- [chess1023byte](https://github.com/cuneytinann/chess1023byte) — yalnız çıplak kurallar, paketlenmiş, 1.023 baytta

## Lisans

MIT
