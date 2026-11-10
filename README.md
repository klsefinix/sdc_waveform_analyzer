# SDC Clock Waveform Analyzer

A single-file, browser-based tool for visualizing Synopsys Design Constraints (SDC) clock definitions and analyzing cross-clock-domain (CDC) edge relationships. Load your `.sdc`/`.tcl` constraints, see the clock waveforms drawn to scale, and inspect launch/capture edge timing for setup and hold — including multicycle-path and timing-exception effects.

No build step, no dependencies, no server. Open the HTML file in any modern browser and go.

---

## Features

### Constraint parsing
Parses the SDC clocking constructs directly in the browser:

- **`create_clock`** — period, named or auto-named, with full support for:
  - `-waveform {rise fall}` including **phase-shifted and non-50% duty-cycle** clocks (e.g. `-waveform {0.090 0.490}`)
  - `-name`, `[get_ports]` / `[get_pins]` / `[get_nets]` / `[get_registers]` targets
  - Bare-node references (`create_clock -period 10 mipi_clk`)
  - Virtual clocks (no physical target node)
- **`create_generated_clock`** — resolved against its master clock, with:
  - `-divide_by`, `-multiply_by`, `-duty_cycle`, `-phase`, `-invert`
  - `-edges {…}` / `-edge_shift {…}` edge-based generation
  - `-master_clock` (explicit) and `-source` (pin/port) master resolution
- **`set_multicycle_path`** — `-setup` / `-hold`, `-start` / `-end`, per `-from` / `-to` clock pair
- **`set_max_delay`** / **`set_min_delay`** — timing exceptions applied to the selected clock pair

Line continuations (`\`), comments, and multi-command files are all handled.

### Clock waveform rendering
- Waveforms drawn to scale on an HTML canvas with a labeled time axis and grid.
- Correct handling of phase offsets, arbitrary duty cycles, and clocks that start high or low — edges are generated across a full guard window and the level at *t=0* is inferred from edge history, so no false glitches at the origin.
- Per-clock info cards: period, frequency (MHz), rise/fall edge times, target node/type, and (for generated clocks) master clock, divide/multiply factors.
- Clear badges distinguishing **Base**, **Base (bare)**, **Virtual**, and **Generated** clocks.

### Cross-domain timing analysis
Select any two clocks (**Clock A** and **Clock B**) and choose an analysis mode:

- **Setup (Max)** — worst-case launch→capture window.
- **Hold (Min)** — hold relationship at/after launch.
- **Bidirectional CDC** — both A→B and B→A, setup and hold, in one view.

The tool reports:
- **Derived (natural) clock relationships** — the true edge timing with no exceptions applied.
- **Adjusted relationships** — the same paths with multicycle paths and max/min-delay exceptions applied, plus the **timing impact** (relaxed/tightened, in ns).
- A **CDC summary** with per-direction setup/hold windows and the effective setup round-trip.

Launch and capture edges are annotated directly on the waveforms with directional markers and connecting arrows.

### Auto time-scale
When time scale and cycle count are left on **auto**, the tool computes the LCM of the two clock periods and extends the window until it captures the worst-case (smallest) edge separation — so the most critical alignment is always on screen. Manual override of time scale and cycle count is available at any time.

### Multicycle-path controls
Dedicated UI for A→B and B→A multicycle paths (setup/hold multiplier and `-start`/`-end` reference). Values are auto-populated from matching `set_multicycle_path` constraints in the loaded SDC, and can be edited live to explore "what-if" scenarios.

### File handling
- **Load** one or more `.sdc` / `.tcl` files, or **drag and drop** them onto the editor.
- Multiple files are concatenated in alphabetical order (use numeric prefixes like `01_clocks.sdc`, `02_generated.sdc` to control sequence).
- Built-in **Example** design demonstrates base clocks, generated clocks, multicycle paths, and timing exceptions.
- Live editor with a sync indicator showing when the analysis is up to date with the current text.

---

## Usage

1. Open `sdc_waveform_viewer.html` in a browser.
2. Click **Load Files** (or drag in your `.sdc`/`.tcl`), or paste constraints into the editor, or click **Example**.
3. Click **Analyze** (or `Ctrl`/`Cmd`+`Enter`).
4. Pick **Clock A**, **Clock B**, and an analysis mode. Adjust multicycle paths or time scale as needed.

Everything runs locally in your browser — no constraint data leaves your machine.

---

## Supported constraint reference

| Construct | Notable options honored |
|---|---|
| `create_clock` | `-period`, `-name`, `-waveform {rise fall}`, `[get_*]` targets, bare node, virtual |
| `create_generated_clock` | `-divide_by`, `-multiply_by`, `-duty_cycle`, `-phase`, `-invert`, `-edges`, `-edge_shift`, `-master_clock`, `-source` |
| `set_multicycle_path` | `-setup`, `-hold`, `-start`, `-end`, `-from`/`-to` clocks |
| `set_max_delay` / `set_min_delay` | delay value, `-from`/`-to` clocks |

> **Note on `-waveform`:** per the SDC standard, the edge list is an even number of alternating rising/falling edges within one period. `{rise fall}` fully defines any duty cycle; the tool renders arbitrary duty cycles and phase offsets correctly. (Odd-length lists such as `{rise fall rise}` are not SDC-conformant — a third edge would need a matching fall to form a complete additional pulse.)

---

## Notes & limitations

- Timing is **edge-relationship / ideal-clock** analysis — it models launch/capture edges, multicycle paths, and delay exceptions, not net/cell propagation delays, clock skew, or jitter. It is a reasoning and visualization aid, not a sign-off STA tool.
- `set_clock_groups` and other advanced exceptions are not currently modeled.
- Multi-pulse-per-period `-waveform` edge lists (four or more edges) are not yet supported; standard single-pulse `{rise fall}` is.

---

## License

Released under the **MIT License** — free to use, modify, and redistribute,
including in commercial and proprietary products. See [`LICENSE`](LICENSE) for
the full text.

This is a free utility intended to help engineers and students better understand
static timing analysis and SDC constraints.

---

## Trademarks & attribution

SDC (Synopsys Design Constraints) is a constraint format that originated with
Synopsys, Inc. "Synopsys" and related names are trademarks of their respective
owners. This project is an independent, unaffiliated utility that reads the
textual SDC constraint format; it is **not** affiliated with, sponsored by, or
endorsed by Synopsys. It contains no Synopsys software, source code, or SDC API
implementation — it parses the standard constraint-file text format supported
across the EDA industry.

---

## Credits

Built by Keith Spencer, with development assistance from **Claude (Opus 4.8)** by
Anthropic — including SDC semantics, the phase-shift / non-50%-duty-cycle
`-waveform` rendering fix, and timing-analysis logic.

🤖 Built with [Claude](https://claude.com/claude-code)
