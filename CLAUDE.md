# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A coursework deliverable, not a software project: an LTspice simulation of the PYD 1378 dual-element PIR (pyroelectric) motion sensor with its amplifier + low-pass signal-conditioning circuit, and the Vietnamese LaTeX report documenting it (HUST, course "Cảm biến và xử lý tín hiệu đo", author Nguyễn Thái Sơn). There is no package manager, linter or test suite. The report and the guide are written in Vietnamese.

`FirstPrompt.md` is the original brief and holds the hard requirements: sensor model + motion input signals + op-amp amplifier + low-pass filter all in **one** LTspice project; 3.3 V single supply; output within 0–3 V; output at 1.5 V when there is no motion.

| Path | What it is |
|---|---|
| `MoPhong_LTspice/PIR_PYD1378.asc` | The whole design in one schematic. Source of truth for every circuit, model and scenario value |
| `MoPhong_LTspice/HUONG_DAN_MO_PHONG.md` | Operating guide: run, probe nodes, AC mode, write motion scenarios, expected `.meas` results, troubleshooting. Read it before changing the simulation |
| `BaoCao/` | LaTeX report: `main.tex` + chapters, `data/` (pgfplots tables), `hinh/` (TikZ schematic redraws) |
| `LHI778.PDF` | PerkinElmer datasheet the sensor model was fitted to (file name is the sensor's old name; it is the same part as PYD 1378) |
| `Demo mô phỏng.mp4` | ~100 MB screen recording, tracked in git; don't open it |

A root `*.rar` is git-ignored.

## Commands

**Build the report.** Needs XeLaTeX and the system fonts that `main.tex` hard-codes (Times New Roman, Arial, Consolas). From `BaoCao/`:

```
latexmk -xelatex main.tex      # or: xelatex main.tex, run twice
```

Takes ~100 s. A plain build rewrites tracked files (`main.pdf`, `main.{aux,toc,lof,lot,log}`). For a verify-only build that leaves the tree untouched, run from the repo root: `latexmk -xelatex -cd -outdir=<absolute dir> BaoCao/main.tex`. The file `BaoCao/Nguyen Thai Son 20252373M - Cam bien va xu ly tin hieu do.pdf` is a byte-identical renamed copy of `main.pdf` (the submission copy); re-copy it after a deliberate rebuild.

**Run the simulation.** Open `PIR_PYD1378.asc` in LTspice (26.1.1 was used; the guide says ≥ 24) → Simulate → Run (~15–30 s). `.meas` results appear in View → SPICE Error Log and in the plain-text `PIR_PYD1378.log`. The repo documents no CLI invocation; if the `ltspice` MCP server is connected (`mcp__ltspice__*`), it can open, run and read the design instead.

**Validate a change.** There are no tests; the `.meas` block is the regression check. With default parameters the log must reproduce the table in HUONG_DAN §2 (`VOUT_NGHI` ≈ 1.499 V, `VOUT_MIN`/`VOUT_MAX` ≈ 0.32/2.55 V, `SK1_PP`…`SK5_PP` ≈ 1.27/2.23/0.54/0.07/0.67 V). Rest level 1.5 V and OUT staying inside 0–3 V are the pass/fail requirements.

## Architecture

**Signal chain in the schematic.** Node names are what you probe and what `.save` keeps.

```
IRA/IRB(time) → PA,PB [1 V = 1 µW] → thermal RC → TA,TB [1 V = 1 K]
  → pyro current sources I = p·A·dT/dt, two elements in anti-series (only A−B survives; B scaled by 1−MISM)
  → G (R_G = 68 GΩ) → JFET source follower J1 → S (R_S = 47 k; ~0.81 V DC + mV signal) → + noise BN → SN
  → U1A (K1 = 101, C1 blocks DC) → A1 → AC-couple R3/C3 to VREF → U1B (K2 = 16) → A2
  → U1C Sallen-Key LPF (f0 ≈ 10.7 Hz) → A3 → R8/R9 divider (×0.909) → OUT
VREF = VDD/2 = 1.65 V (R10/R11 + U1D follower). U1A–U1D are one AD8608 quad on a single 3.3 V rail.
Rest level = 0.909 × 1.65 V = 1.5 V.
```

Model parameters, motion scenarios, noise, `.options`, the analysis line and the `.meas` lines are SPICE-directive **text blocks** on the sheet, named in unaccented ASCII: `THONG SO MO HINH PYD1378`, `KICH BAN CHUYEN DONG`, `NHIEU`, `MO PHONG`, `DO LUONG AC`, `DO LUONG`. Motion is `.param`/`.func` expressions (`gp` Gaussian pulse, `walk` pulse train, `sstep` smooth step → `IRA`/`IRB`). In the walking scenarios the same pulses hit A at `S` and B at `S+T/2` (A leads); swapping the offsets reverses direction.

The sensor parameters (τ_T = 0.6 s, τ_E = 2.5 s, R_G = 68 GΩ, p = 3.7e-4, …) are **fitted** to datasheet curve points, not manufacturer data. The derivation is in `c2b_mohinh.tex`, and the datasheet-match table `tab:kiemchung` depends on them. The amplifier/filter derivations are in `c2c_mach.tex`.

**Report ↔ simulation coupling.** The report is synchronised by hand; nothing is generated at build time.

- `BaoCao/data/*.dat` are checked-in LTspice exports (header row; pgfplots reads columns by name, e.g. `table[x=t,y=out]`, in `c2b_mohinh.tex` and `c2d_ketqua.tex`). Columns are post-processed, not raw node voltages (e.g. `s` and `g` are in mV and `s` is DC-subtracted; `ac.dat` has `R` in kV/W and `G`, `G1`–`G3` in dB). No script in the repo regenerates them, so compare against an existing file before overwriting one.
- SK6 (reverse direction) exists only as the recipe in HUONG_DAN §5.5. The shipped `.asc` has just SK1–SK5, so regenerating `dir_sk6.dat` means re-applying that recipe.
- `BaoCao/hinh/ltspice_{cb,mach,vref}.tex` are TikZ redraws of the schematic (uniformly scaled `.asc` coordinates, no generator script in the repo), shown as three subfigures in `c2d_ketqua.tex`. If the schematic changes they go stale.
- Numbers are typed in, not computed. Component values appear in the `.asc`, in `c2c_mach.tex` (BOM `tab:bom` and derivations) and in the `hinh/ltspice_*.tex` labels. The model and scenario `.param`/`.func` blocks are also copied verbatim into `phuluc.tex`, and the scenario parameters into HUONG_DAN §5.3. Headline results (rest level 1.4992 V, 62.4 dB at 1 Hz, OUT 0.32–2.55 V, 47 dB / 137 dB attenuation at 50 Hz / 1 kHz, 4.06 kV/W) appear in `c2d_ketqua.tex`, `chuong3.tex`, `c2b_mohinh.tex`, `c2c_mach.tex` and the guide. After any circuit or parameter change: re-run, check against HUONG_DAN §2, then sweep all of these. `tab:danhgia` (end of `c2d_ketqua.tex`) maps each requirement of the brief to its measured result.
- Report layout: `main.tex` (preamble, macros) inputs `bia`, `loinoidau`, `kyhieu`, `chuong1` (overview), `chuong2`, `chuong3` (conclusion), `tailieu` (a hand-written `thebibliography`) and `phuluc`. `chuong2.tex` is only a container for `c2a_nguyenly` (principle, sensor structure), `c2b_mohinh` (sensor model, input signals), `c2c_mach` (circuit design) and `c2d_ketqua` (results). Those four files hold sections, not chapters.

## LTspice editing gotchas

These come from the guide (§5.7, §7) and the report (`c2b_mohinh.tex`, "Cài đặt mô hình").

- In the `.asc`, each directive block is one `TEXT x y Left n !…` line whose SPICE lines are separated by a literal `\n`. `!` marks a SPICE directive and `;` a comment. To switch between `.tran 0 40 0 0.1m` (default) and `.ac dec 100 0.01 1k`, swap the `!`/`;` on those two lines. Only one may be active, and the `.meas AC` lines only act under `.ac`.
- All sheet text is unaccented ASCII and the guide cites blocks by those names, so keep both.
- Keep `.options gmin=1e-15`. With the default gmin, ~pA leakage into the 68 GΩ gate shifts V(S) by ~0.26 V.
- No `**` in expressions; it breaks on negative bases and gives "Time step too small". Write `(x-c)*(x-c)`. Don't name a `.param` `tr`.
- Each `.meas TRAN` has a hard-coded `FROM … TO …` window per scenario (SK1 2.5–9.5 s … SK5 30.5–40 s). Keep scenarios ≥ 4–5 s apart. If you move one, update its window, and extend the `.tran` stop time if it runs past 40 s.
- `.save` limits which nodes are stored; delete that line to probe others. `PIR_PYD1378.plt` must stay beside the `.asc` for the 4-pane plot to open.
- `PIR_PYD1378.raw` (~27 MB), `.log`, `.net`, `.db` and `.op.raw` are tracked and rewritten by every run, so a check-only run leaves binary diffs.

## Report conventions

- Vietnamese with diacritics, in the author's first-person student voice ("em").
- Decimal comma and a non-breaking `~` before units (`1,4992~V`). `\num{3.7e-4}` takes a period and prints a comma. In math mode `icomma` handles the comma.
- Write the sensor as `\PYD{}` (prints "PYD 1378"). Other macros and colours are in `main.tex`: `\VREF`, `\dd`, plot colours `c1`–`c4`, `cgray`, `bkred`, and `tabularx` column types `Y` (centred) and `L` (left).
