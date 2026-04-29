# Train Loading Analysis (Cooper E-XX + AREMA Alternate)

A single-file, browser-based analysis tool for railway live loadings on simply-supported spans with optional cantilevers. Computes moving load envelopes, influence lines, and design envelopes for the AREMA Ch.15 Cooper E-XX axle pattern and the AREMA Alternate Live Load, with separate dead load output and a dynamic impact factor.

The entire tool is one self-contained `.html` file. There is no build step, no installation, no server, no account, and no telemetry. Open the file in any modern browser and every input, plot, and table is available immediately. All computation runs locally.

## Features

- **Cooper E-XX live load** with user-selectable E rating (E-50 to E-120) and 1, 2, or 3 locomotives, including the trailing 8 k/ft (E-80 reference) uniform load
- **AREMA Alternate Live Load**, four 100-kip axles at 5 ft, 6 ft, 5 ft spacings (no uniform tail)
- **Moving load envelopes** for moment and shear, with the lead axle swept across the structure in both directions of travel
- **Influence line analysis** at any user-selected section, with the IL plot, the optimal-placement diagram on the beam, and a placement table covering both Cooper and the Alternate
- **Dead load tab** for a uniform DL applied across the entire structure (cantilevers + span); not amplified by impact, per AREMA 1.3.5
- **Dynamic impact factor I** with two modes:
  - AREMA Ch.15 auto-default for *rolling equipment without hammer blow*:
    - I = 1 + (40 - 3·L²/1600) / 100 for L < 80 ft
    - I = 1 + (16 + 600/(L - 30)) / 100 for L ≥ 80 ft
  - User override (manual entry, current AREMA value pre-loaded as a starting point)
- **Combined design envelope** taking the worst case across Cooper and the AREMA Alternate at every section, with per-loading peak comparisons and a governing-loading tag at every peak
- **Pin support at A** (triangle on hatching), **roller support at B** (circle on hatching), with all diagrams sharing a consistent visual style
- **Shear discontinuity at supports** captured by section-flanking points at ±0.001 ft, so the support reaction shows up as a clean visual jump in the shear envelope
- **Print or save as PDF** through the browser's print dialog

## Usage

1. Download `train-loading-analysis.html` from the repository.
2. Open it in any modern browser (Chrome, Firefox, Safari, Edge).
3. Edit the inputs in the top panel: geometry (span and cantilevers), live loads (E rating, number of locomotives, AREMA Alternate inclusion, impact factor), and dead load.
4. Click through the result tabs to view envelopes, influence lines, dead load forces, and the combined design envelope.

The page can be saved as a static HTML record of an analysis run, or printed to PDF directly from the browser.

## Inputs

### Structure
- Span L (between supports), in ft
- Left cantilever length, in ft (0 if none)
- Right cantilever length, in ft (0 if none)

### Live Loads & Impact
- E rating (e.g., 80 for Cooper E-80); axle loads scale linearly as E / 80
- Number of locomotives (1, 2, or 3)
- AREMA Alternate Live Load: include in design envelope (yes/no); the AREMA Envelope tab is always available regardless
- Impact factor I: AREMA Ch.15 auto-default (checked) or manual override

### Dead Load
- Uniform dead load w_DL in k/ft, applied over the entire structure including cantilevers; not amplified by impact

## Outputs

The tool exposes five tabs of output:

### 1. E-{rating} Envelope ({n} loco)
Cooper E-XX moving load envelope with impact factor applied. Includes:
- Moment envelope plot, M+ (sagging) and M- (hogging)
- Shear envelope plot, V+ and V-
- Peak values table (largest M+, M-, V+, V- with their section locations)
- Full per-section envelope table

### 2. AREMA Envelope
Same outputs as above, computed for the four-axle AREMA Alternate Live Load.

### 3. Influence Lines
For a user-selected section x and quantity (M, V left of section, V right of section):
- IL plot with the section marked by a red dashed line
- Beam-with-train diagram showing the governing loading at its optimal lead-axle position and direction of travel
- Optimal placement table for both Cooper and the AREMA Alternate, with maximum positive and minimum negative responses, governing lead-axle positions, and travel directions
- Sampled IL ordinates table

### 4. Dead Load
- M_DL curve over the entire structure
- V_DL curves with the support discontinuities visible
- Peak DL values with their locations
- Full per-section DL table

### 5. Design Envelope
Worst case at every section across the included live loadings (Cooper, plus AREMA Alternate if checked), with impact factor applied:
- Combined moment envelope, with the governing loading tagged at each peak
- Combined shear envelope, similarly tagged
- Per-loading peak comparison cards
- Full design envelope table with governing-loading column

## Analysis Methodology

### Coordinate System and Conventions
- x = 0 at the left support (A), x = L at the right support (B)
- Optional cantilevers extend over [-Cl, 0] on the left and [L, L + Cr] on the right
- Positive M is sagging
- Positive V is the upward force on the left side of the section cut
- Lead axle moving "left to right" means the lead is the rightmost axle at any instant; "right to left" means the lead is the leftmost

### Cooper E-XX Train Geometry (from lead axle backward)
- Locomotive: 5 axles, 40 + 80 + 80 + 80 + 80 kips at 8, 5, 5, 5 ft spacings (E-80 magnitudes; scale by E/80 for other ratings)
- Driver-to-tender gap: 9 ft
- Tender: 4 axles at 52 kips each with 5, 6, 5 ft spacings
- Tender-to-next-locomotive gap: 8 ft (when multiple locomotives are used)
- Last tender to start of uniform load: 5 ft
- Trailing uniform load: 8 k/ft (E-80 reference; scales linearly by E/80)

### AREMA Alternate Live Load Geometry
- Four 100-kip axles at 5 ft, 6 ft, 5 ft spacings
- No uniform tail

### Solver
- Section forces are computed in closed form as the sum of axle contributions and a uniform-load contribution, with reactions computed by equilibrium for each placed configuration
- Off-bridge axles are filtered out before reactions and section forces are computed, so consist positions during entry and exit do not produce spurious reactions
- The lead axle is swept across the structure in both directions, with the envelope tracking the maximum and minimum M and V at every section
- The default sweep step is 1.0 ft for the lead axle and 2.0 ft for the section grid
- Influence line evaluation places point loads at IL critical positions and integrates the uniform tail using the trapezoidal rule with explicit zero-crossing detection so sign changes in the IL are handled correctly
- Optimal placement search sweeps the lead-axle position in both directions and reports the placement giving the largest positive and most negative IL response at the section

### Shear Discontinuity at Supports
The section grid drops any base-grid point that falls within ±0.0004 ft of a support and inserts flanking points at ±0.001 ft on each side. The internal side-of-cut offset in `computeShear` is set to 1e-9 ft so that each flanking section evaluates cleanly on its own side of the discontinuity. The result is a shear envelope that shows the support reaction as a near-vertical jump between the two flanking sections rather than as an interpolated slope or an artifactual deep-negative spike on the cantilever side.

### Impact Factor Application
- Live-load envelope values (Cooper, AREMA Alternate, and the combined design envelope) are post-multiplied by the impact factor I before display
- Optimal-placement values reported in the IL tab are also multiplied by I
- IL ordinates themselves are unit-load quantities and are not scaled
- Dead load is never amplified by I

## Technical Implementation

- Single `.html` file, approximately 75 KB
- React 18 loaded via CDN (`react.production.min.js`, `react-dom.production.min.js`)
- Babel standalone for in-browser JSX transformation
- Tailwind CSS via the play CDN for utility styling
- All plots, structure diagrams, axle arrows, and influence-line graphics rendered directly with SVG; no charting library
- Self-contained JavaScript analysis engine, no external numerical packages

The file can be inlined for fully offline use by replacing the CDN script tags with the corresponding library contents.

## References

- American Railway Engineering and Maintenance-of-Way Association, *AREMA Manual for Railway Engineering*, Chapter 15 (Steel Structures)
  - Section 1.3.3: Cooper E-XX live load
  - Section 1.3.4: Alternate Live Load
  - Section 1.3.5: Dynamic impact for rolling equipment without hammer blow
- Higgins, C. (2004), original Fortran moving load analysis program (precursor)

## Author

**Christopher C. Higgins, PhD, PE**
School of Civil and Construction Engineering
Oregon State University

## Attribution

This tool is based on the original 2004 Fortran moving load analysis program developed by the author for teaching and research in bridge engineering. The browser version reimplements and extends that engine in JavaScript, adds the AREMA Cooper and Alternate loadings, and provides interactive plots, influence-line analysis, and a combined design envelope.

## Disclaimer

This tool is provided for engineering exploration and educational purposes only. **Use at your own risk.** No warranty, express or implied, is made regarding the correctness, accuracy, completeness, or fitness for any particular use of the analysis results, source code, or documentation. Results produced by this tool must be independently verified by a licensed professional engineer before being used for design, evaluation, rating, construction, or any decision affecting public safety or property.

The author and Oregon State University assume no responsibility or liability for any errors, omissions, or consequences arising from the use of this software.

## License

Released under the MIT License unless noted otherwise. See the `LICENSE` file in the repository for full text.

## Feedback

Issues, pull requests, and suggestions are welcome through the GitHub issue tracker on this repository.
