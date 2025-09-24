<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>MAE Calculator</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <!-- React 18 + ReactDOM 18 (UMD) + Babel for in-browser JSX -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <!-- Optional font -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet" />
</head>
<body>
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    // Page base styles (light background)
    function GlobalStyles() {
      return (
        <style>{`
          html, body, #root {
            background: #f5f6f8;
            margin: 0;
            padding: 0;
            min-height: 100%;
            font-family: Inter, system-ui, Arial, sans-serif;
          }
          * { box-sizing: border-box; }
        `}</style>
      );
    }

    // Texas Tech-ish light theme
    const TT = {
      red: "#CC0000",
      redDark: "#990000",
      black: "#0b0b0b",
      text: "#111827",
      white: "#ffffff",
      page: "#f5f6f8",
      card: "#ffffff",
      head: "#0b0b0b",
      headText: "#f9fafb",
      border: "#e5e7eb",
      grid: "#d1d5db"
    };

    // ===== App shell (tabs) =====
    function App() {
      const [species, setSpecies] = useState("pork"); // "pork" | "beef" | "reasons"

      return (
        <div style={{ color: TT.text, background: TT.page, minHeight: "100vh" }}>
          <GlobalStyles />
          <div style={{ maxWidth: 1200, margin: "0 auto", padding: "24px 16px 12px" }}>
            <h1 style={{ margin: 0 }}>MAE Calculator</h1>
          </div>

          {/* Top tabs (match cards style) */}
          <div style={{ maxWidth: 1200, margin: "0 auto", padding: "0 16px" }}>
            <div
              style={{
                display: "flex",
                gap: 8,
                background: TT.card,
                padding: 6,
                borderRadius: 12,
                marginBottom: 16,
                border: `1px solid ${TT.border}`
              }}
            >
              <InnerTab active={species === "pork"} onClick={() => setSpecies("pork")}>Pork</InnerTab>
              <InnerTab active={species === "beef"} onClick={() => setSpecies("beef")}>Beef</InnerTab>
              <InnerTab active={species === "reasons"} onClick={() => setSpecies("reasons")}>Reasons</InnerTab>
            </div>
          </div>

          <div style={{ maxWidth: 1200, margin: "0 auto", padding: "0 16px 24px" }}>
            {species === "pork" ? <PorkSection /> : species === "beef" ? <BeefSection /> : <ReasonsSection />}
          </div>
        </div>
      );
    }

    // ===== Reasons (timer) =====
    function ReasonsSection() {
      const [timeLeft, setTimeLeft] = useState(15 * 60);
      const [running, setRunning] = useState(false);
      const intervalRef = useRef(null);
      const warnSound = useRef(null);
      const alarmSound = useRef(null);

      useEffect(() => {
        if (!running) return;
        intervalRef.current = setInterval(() => {
          setTimeLeft((t) => {
            if (t <= 1) {
              clearInterval(intervalRef.current);
              intervalRef.current = null;
              try { alarmSound.current?.play(); } catch {}
              return 0;
            }
            return t - 1;
          });
        }, 1000);
        return () => clearInterval(intervalRef.current);
      }, [running]);

      useEffect(() => {
        if (timeLeft === 8 * 60 || timeLeft === 2 * 60) {
          try { warnSound.current?.play(); } catch {}
        }
      }, [timeLeft]);

      const fmt = (s) => `${Math.floor(s/60)}:${String(s%60).padStart(2,"0")}`;
      const onStartPause = () => setRunning(r => !r);
      const onReset = () => {
        clearInterval(intervalRef.current);
        intervalRef.current = null;
        setTimeLeft(15 * 60);
        setRunning(false);
      };

      return (
        <div>
          <h2 style={{ margin: "4px 0 12px" }}>Reasons Timer</h2>
          <div style={{ fontSize: "3rem", fontWeight: "bold", marginBottom: 20, color: TT.red }}>{fmt(timeLeft)}</div>
          <div style={{ display: "flex", gap: 8 }}>
            <button onClick={onStartPause} style={styles.btnPrimary} type="button">{running ? "Pause" : "Start"}</button>
            <button onClick={onReset} style={styles.btnSecondary} type="button">Reset</button>
          </div>
          <audio ref={warnSound} src="https://actions.google.com/sounds/v1/alarms/beep_short.ogg" preload="auto" />
          <audio ref={alarmSound} src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" preload="auto" />
          <p style={{ color: "#6b7280", marginTop: 8, fontSize: 12 }}>Tip: Some mobile browsers require one tap before audio can play.</p>
        </div>
      );
    }

    // ===== Pork (light table) =====
    function PorkSection() {
      const [tab, setTab] = useState("calc");
      const [rows, setRows] = useState([
        { BW: "", BF: "", LEA: "", DP: "", MS: "" },
        { BW: "", BF: "", LEA: "", DP: "", MS: "" },
        { BW: "", BF: "", LEA: "", DP: "", MS: "" },
        { BW: "", BF: "", LEA: "", DP: "", MS: "" }
      ]);

      const toNum = (v) => {
        const n = parseFloat(String(v ?? "").trim());
        return Number.isFinite(n) ? n : NaN;
      };

      const computeLRBF = (bf) => {
        if (!Number.isFinite(bf)) return NaN;
        let adj = 0;
        if (bf <= 0.7) adj = 0.25;
        else if (bf < 0.8) adj = 0.25;
        else if (bf < 0.9) adj = 0.2;
        else if (bf < 1.0) adj = 0.15;
        else if (bf < 1.1) adj = 0.1;
        else if (bf < 1.2) adj = 0.05;
        else if (bf < 1.3) adj = 0.0;
        else adj = -0.05;
        return bf + adj;
      };

      const dpChart = {
        fflBins: [
          { min: 58, max: Infinity }, { min: 56, max: 57.9 }, { min: 54, max: 55.9 },
          { min: 52, max: 53.9 }, { min: 50, max: 51.9 }, { min: 48, max: 49.9 },
          { min: 46, max: 47.9 }, { min: 44, max: 45.9 }, { min: -Infinity, max: 44 }
        ],
        hcwBins: [
          { min: -Infinity, max: 149 }, { min: 150, max: 159.9 }, { min: 160, max: 169.9 },
          { min: 170, max: 179.9 }, { min: 180, max: 189.9 }, { min: 190, max: 199.9 },
          { min: 200, max: 209.9 }, { min: 210, max: 219.9 }, { min: 220, max: 229.9 },
          { min: 230, max: 239.9 }, { min: 240, max: 250 }, { min: 250.000001, max: Infinity }
        ],
        values: [
          [0.7, 0.7, 0.7, 0.7, 0.7, 0.7, 0.68, 0.68, 0.65],
          [0.93, 0.95, 1.0, 0.97, 0.94, 0.94, 0.92, 0.89, 0.82],
          [0.97, 1.0, 1.0, 1.0, 0.98, 0.97, 0.97, 0.9, 0.83],
          [1.0, 1.02, 1.02, 1.0, 1.0, 1.0, 0.98, 0.91, 0.84],
          [1.0, 1.02, 1.04, 1.02, 1.0, 1.0, 0.98, 0.91, 0.84],
          [1.02, 1.04, 1.04, 1.04, 1.02, 1.0, 0.98, 0.91, 0.84],
          [1.02, 1.04, 1.06, 1.06, 1.04, 1.02, 0.98, 0.91, 0.84],
          [1.02, 1.06, 1.06, 1.06, 1.04, 1.02, 0.98, 0.9, 0.83],
          [1.02, 1.04, 1.04, 1.04, 1.02, 1.0, 0.95, 0.88, 0.81],
          [1.0, 1.02, 1.02, 1.02, 1.0, 0.98, 0.92, 0.85, 0.78],
          [0.95, 1.0, 1.0, 0.98, 0.96, 0.93, 0.88, 0.81, 0.74],
          [0.95, 0.95, 0.94, 0.92, 0.9, 0.88, 0.83, 0.76, 0.69]
        ]
      };

      const binIndex = (bins, x) => {
        if (!Number.isFinite(x)) return -1;
        for (let i = 0; i < bins.length; i++) {
          const { min, max } = bins[i];
          const last = i === bins.length - 1;
          if (x >= min && (last ? x <= max : x < max)) return i;
        }
        return -1;
      };

      const lookupAdjustedDP = (hcw, ffl) => {
        const i = binIndex(dpChart.hcwBins, hcw);
        const j = binIndex(dpChart.fflBins, ffl);
        if (i < 0 || j < 0) return NaN;
        const val = dpChart.values?.[i]?.[j];
        return Number.isFinite(val) ? val : NaN;
      };

      const derive = (r) => {
        const BW = toNum(r.BW);
        const BF = toNum(r.BF);
        const LEA = toNum(r.LEA);
        let DP = toNum(r.DP);
        const MS = toNum(r.MS);
        if (Number.isFinite(DP) && DP > 1) DP = DP / 100;

        const LRBF = computeLRBF(BF);
        const HCW = Number.isFinite(BW) && Number.isFinite(DP) ? BW * DP : NaN;
        const FFL = Number.isFinite(BF) && Number.isFinite(LEA) ? 54.5 - 11.9 * BF + 1.1 * LEA : NaN;
        const USDA = Number.isFinite(LRBF) && Number.isFinite(MS) ? 4 * LRBF - MS : NaN;
        const ADJDP_DEC = Number.isFinite(HCW) && Number.isFinite(FFL) ? lookupAdjustedDP(HCW, FFL) : NaN;

        return {
          LRBF: Number.isFinite(LRBF) ? +LRBF.toFixed(3) : "—",
          HCW: Number.isFinite(HCW) ? +HCW.toFixed(2) : "—",
          FFL: Number.isFinite(FFL) ? +FFL.toFixed(2) : "—",
          USDA: Number.isFinite(USDA) ? +USDA.toFixed(2) : "—",
          ADJDP_DEC,
          ADJDP: Number.isFinite(ADJDP_DEC) ? `${(ADJDP_DEC * 100).toFixed(2)}%` : "—"
        };
      };

      const updateCell = (i, field, val) => {
        setRows(prev => prev.map((r, idx) => (idx === i ? { ...r, [field]: val } : r)));
      };

      const resetCalc = () => setRows([
        { BW: "", BF: "", LEA: "", DP: "", MS: "" },
        { BW: "", BF: "", LEA: "", DP: "", MS: "" },
        { BW: "", BF: "", LEA: "", DP: "", MS: "" },
        { BW: "", BF: "", LEA: "", DP: "", MS: "" }
      ]);

      const [pen, setPen] = useState([]);
      const [pricingInputs, setPricingInputs] = useState({ baseDpPriceCwt: "", standardizedDp: "" });
      const [pricingResults, setPricingResults] = useState(null);

      const addAllToPen = () => {
        const computed = rows
          .map(r => {
            const d = derive(r);
            return d.HCW !== "—" && d.FFL !== "—" && Number.isFinite(d.ADJDP_DEC)
              ? { HCW: d.HCW, FFL: d.FFL, ADJDP: d.ADJDP_DEC }
              : null;
          })
          .filter(Boolean);
        if (!computed.length) {
          alert("Need valid HCW and %FFL (to derive Adj DP from the chart) to add to pen.");
          return;
        }
        setPen(computed);
      };

      const handlePricingChange = (e) => {
        const { name, value } = e.target;
        setPricingInputs(p => ({ ...p, [name]: value }));
      };

      const toNumStrict = (v) => {
        const n = parseFloat(String(v ?? "").trim());
        return Number.isFinite(n) ? n : NaN;
      };

      const onCalculatePricing = () => {
        if (!pen.length) {
          alert("Add animals to the pen from Calculations.");
          return;
        }
        const baseDpPriceCwt = toNumStrict(pricingInputs.baseDpPriceCwt);
        let standardizedDp = toNumStrict(pricingInputs.standardizedDp);
        if (!Number.isFinite(baseDpPriceCwt) || !Number.isFinite(standardizedDp)) {
          alert("Pricing inputs must be numeric.");
          return;
        }
        if (standardizedDp > 1) standardizedDp = standardizedDp / 100;
        const avgAdjDp = pen.reduce((s, a) => s + a.ADJDP, 0) / pen.length;
        const carcassPricePerCwt = baseDpPriceCwt * avgAdjDp;
        const livePricePerCwt = carcassPricePerCwt * standardizedDp;
        const avgHCW = pen.reduce((s, a) => s + a.HCW, 0) / pen.length;
        const carcassValue = (avgHCW / 100) * carcassPricePerCwt;

        setPricingResults({
          avgAdjDp: +(avgAdjDp * 100).toFixed(2),
          carcassPricePerCwt: +carcassPricePerCwt.toFixed(2),
          livePricePerCwt: +livePricePerCwt.toFixed(2),
          carcassValue: +carcassValue.toFixed(2)
        });
      };

      return (
        <div>
          <h2 style={{ margin: "4px 0 12px" }}>Pork</h2>

          {/* Inner tabs */}
          <div
            style={{
              display: "flex",
              gap: 8,
              background: TT.card,
              padding: 6,
              borderRadius: 12,
              marginBottom: 16,
              border: `1px solid ${TT.border}`
            }}
          >
            <InnerTab active={tab === "calc"} onClick={() => setTab("calc")}>Calculations</InnerTab>
            <InnerTab active={tab === "pricing"} onClick={() => setTab("pricing")}>Pricing</InnerTab>
          </div>

          {tab === "calc" && (
            <div>
              <p style={{ marginBottom: 12 }}>
                Spreadsheet-style input for <b>four head</b>. Enter numbers; derived columns update automatically. DP accepts 0.75 or 75.
              </p>

              <LightTableCard>
                <table style={{ width: "100%", borderCollapse: "collapse", minWidth: 980 }}>
                  <thead>
                    <tr>
                      <Th>#</Th><Th>BW (lb)</Th><Th>BF (in)</Th><Th>LEA (in²)</Th><Th>DP</Th><Th>MS</Th>
                      <Th>Adj DP (%)</Th><Th>LRBF (in)</Th><Th>HCW (lb)</Th><Th>%FFL</Th><Th>USDA</Th>
                    </tr>
                  </thead>
                  <tbody>
                    {rows.map((r, i) => {
                      const d = derive(r);
                      return (
                        <tr key={i}>
                          <Td>{i + 1}</Td>
                          <Td><Cell value={r.BW}  onChange={(v) => updateCell(i, "BW", v)}  placeholder="280" /></Td>
                          <Td><Cell value={r.BF}  onChange={(v) => updateCell(i, "BF", v)}  placeholder="0.85" /></Td>
                          <Td><Cell value={r.LEA} onChange={(v) => updateCell(i, "LEA", v)} placeholder="6.5" /></Td>
                          <Td><Cell value={r.DP}  onChange={(v) => updateCell(i, "DP", v)}  placeholder="0.75 or 75" /></Td>
                          <Td><Cell value={r.MS}  onChange={(v) => updateCell(i, "MS", v)}  placeholder="2.5" /></Td>
                          <Td>{d.ADJDP}</Td><Td>{d.LRBF}</Td><Td>{d.HCW}</Td><Td>{d.FFL}</Td><Td>{d.USDA}</Td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </LightTableCard>

              <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
                <button onClick={resetCalc} style={styles.btnSecondary} type="button">Clear Table</button>
                <button onClick={addAllToPen} style={styles.btnPrimary} type="button">Add All to Pen</button>
              </div>
            </div>
          )}

          {tab === "pricing" && (
            <div>
              <p style={{ marginBottom: 8 }}>Average DP = mean of the Adj DP (%) from the chart for animals in the pen.</p>
              <h3 style={{ marginTop: 8 }}>Pen</h3>
              {!pen.length && <p style={{ color: "#6b7280" }}>No animals yet. On Calculations → Add All to Pen.</p>}
              {pen.length > 0 && (
                <LightTableCard>
                  <table style={{ width: "100%", borderCollapse: "collapse", minWidth: 560 }}>
                    <thead>
                      <tr><Th>#</Th><Th>HCW (lb)</Th><Th>%FFL</Th><Th>Adj DP (%)</Th></tr>
                    </thead>
                    <tbody>
                      {pen.map((a, i) => (
                        <tr key={i}><Td>{i + 1}</Td><Td>{a.HCW}</Td><Td>{a.FFL}</Td><Td>{(a.ADJDP * 100).toFixed(2)}%</Td></tr>
                      ))}
                    </tbody>
                  </table>
                </LightTableCard>
              )}

              <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(260px, 1fr))", gap: 12, marginTop: 16, maxWidth: 820 }}>
                <LabeledInput name="baseDpPriceCwt" value={pricingInputs.baseDpPriceCwt} onChange={handlePricingChange} label="Base DP Price ($/cwt)" placeholder="90" />
                <LabeledInput name="standardizedDp" value={pricingInputs.standardizedDp} onChange={handlePricingChange} label="Standardized DP (decimal or %)" placeholder="0.75 or 75" />
              </div>

              <div style={{ marginTop: 12 }}>
                <button onClick={onCalculatePricing} style={styles.btnPrimary} type="button">Calculate Pricing</button>
              </div>

              {pricingResults && (
                <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(240px, 1fr))", gap: 12, marginTop: 18 }}>
                  <Card title="Avg Adjusted DP (%)" value={pricingResults.avgAdjDp} />
                  <Card title="Carcass Price ($/cwt)" value={pricingResults.carcassPricePerCwt} />
                  <Card title="Live Price ($/cwt)" value={pricingResults.livePricePerCwt} />
                  <Card title="Carcass Value ($/head)" value={pricingResults.carcassValue} />
                </div>
              )}
            </div>
          )}
        </div>
      );
    }

    // ===== Beef (light table) =====
    function BeefSection() {
      const [tab, setTab] = useState("calculations");
      const [rows, setRows] = useState([
        { BW: "", FT: "", AREA: "", KPH: "", QG: "" },
        { BW: "", FT: "", AREA: "", KPH: "", QG: "" },
        { BW: "", FT: "", AREA: "", KPH: "", QG: "" },
        { BW: "", FT: "", AREA: "", KPH: "", QG: "" }
      ]);

      const toNum = (v) => {
        const n = parseFloat(String(v ?? "").trim());
        return Number.isFinite(n) ? n : NaN;
      };

      const ftOptions = [];
      for (let x = 0.2; x <= 1.320001; x += 0.04) ftOptions.push(x.toFixed(2));

      const qgOptions = [
        { v: "", label: "—" },
        { v: "Sl-", label: "Select- (Sl-)" },
        { v: "Sl+", label: "Select+ (Sl+)" },
        { v: "Ch-", label: "Choice- (Sm/Ch-)" },
        { v: "ChAvg", label: "Choice avg (Mt/Ch0)" },
        { v: "Ch+", label: "Choice+ (Md/Ch+)" },
        { v: "Pr-", label: "Prime- (Pr-)" },
        { v: "PrAvg", label: "Prime avg (Pr0)" }
      ];

      const computeBREA = (bw) => {
        const n = toNum(bw);
        if (!Number.isFinite(n)) return NaN;
        return (n - 950) / 130 + 11;
      };

      const computePYG = (ft) => {
        const n = toNum(ft);
        if (!Number.isFinite(n)) return NaN;
        const steps = Math.max(0, Math.round((n - 0.2) / 0.04));
        return +(2.5 + steps * 0.1).toFixed(1);
      };

      const reaAdj = (BREA, AREA) => {
        const b = toNum(BREA);
        const a = toNum(AREA);
        if (![b, a].every(Number.isFinite)) return NaN;
        return (b - a) / 3;
      };

      const kphAdj = (KPH) => {
        const k = toNum(KPH);
        if (!Number.isFinite(k)) return NaN;
        return (k - 3.5) * 0.2; // 3.0 => -0.1 ; 4.0 => +0.1
      };

      const computeFYG = ({ BREA, AREA, KPH, PYG }) => {
        const p = toNum(PYG);
        const r = reaAdj(BREA, AREA);
        const k = kphAdj(KPH);
        if (![p, r, k].every(Number.isFinite)) return NaN;
        return +(p + r + k).toFixed(2);
      };

      const update = (i, field, val) => {
        setRows(prev => prev.map((r, idx) => (idx === i ? { ...r, [field]: val } : r)));
      };

      return (
        <div>
          <h2 style={{ margin: "4px 0 12px" }}>Beef</h2>

          <div
            style={{
              display: "flex",
              gap: 8,
              background: TT.card,
              padding: 6,
              borderRadius: 12,
              marginBottom: 16,
              border: `1px solid ${TT.border}`
            }}
          >
            <InnerTab active={tab === "calculations"} onClick={() => setTab("calculations")}>Calculations</InnerTab>
            <InnerTab active={tab === "pricing"} onClick={() => setTab("pricing")}>Pricing</InnerTab>
          </div>

          {tab === "calculations" && (
            <div>
              <p style={{ marginBottom: 12 }}>
                Enter BW, choose FT (0.04 steps). BREA &amp; PYG auto-calc. Enter AREA &amp; KPH (%).
                FYG = PYG + (BREA − AREA)/3 + (KPH − 3.5) × 0.2.
              </p>

              <LightTableCard>
                <table style={{ width: "100%", borderCollapse: "collapse", minWidth: 1100 }}>
                  <thead>
                    <tr>
                      <Th>#</Th><Th>BW (lb)</Th><Th>FT (in)</Th><Th>PYG</Th><Th>BREA (in²)</Th>
                      <Th>AREA (in²)</Th><Th>REA Δ/3</Th><Th>KPH (%)</Th><Th>KPH Adj</Th><Th>FYG</Th><Th>QG</Th>
                    </tr>
                  </thead>
                  <tbody>
                    {rows.map((r, i) => {
                      const BREA = computeBREA(r.BW);
                      const PYG = computePYG(r.FT);
                      const REA_DELTA_THIRDS = reaAdj(BREA, r.AREA);
                      const KPH_ADJ = kphAdj(r.KPH);
                      const FYG = computeFYG({ BREA, AREA: r.AREA, KPH: r.KPH, PYG });

                      return (
                        <tr key={i}>
                          <Td>{i + 1}</Td>
                          <Td><input type="text" inputMode="decimal" value={r.BW} onChange={(e)=>update(i,"BW",e.target.value)} placeholder="1200" style={inputStyle} /></Td>
                          <Td>
                            <select value={r.FT} onChange={(e)=>update(i,"FT",e.target.value)} style={selectStyle}>
                              <option value="">—</option>
                              {ftOptions.map(ft => <option key={ft} value={ft}>{ft}</option>)}
                            </select>
                          </Td>
                          <Td>{Number.isFinite(PYG) ? PYG.toFixed(1) : "—"}</Td>
                          <Td>{Number.isFinite(BREA) ? BREA.toFixed(2) : "—"}</Td>
                          <Td><input type="text" inputMode="decimal" value={r.AREA} onChange={(e)=>update(i,"AREA",e.target.value)} placeholder="13.4" style={inputStyle} /></Td>
                          <Td>{Number.isFinite(REA_DELTA_THIRDS) ? REA_DELTA_THIRDS.toFixed(2) : "—"}</Td>
                          <Td><input type="text" inputMode="decimal" value={r.KPH} onChange={(e)=>update(i,"KPH",e.target.value)} placeholder="3.5" style={inputStyle} /></Td>
                          <Td>{Number.isFinite(KPH_ADJ) ? KPH_ADJ.toFixed(2) : "—"}</Td>
                          <Td>{Number.isFinite(FYG) ? FYG.toFixed(2) : "—"}</Td>
                          <Td>
                            <select value={r.QG} onChange={(e)=>update(i,"QG",e.target.value)} style={selectStyle}>
                              {qgOptions.map(q => <option key={q.v} value={q.v}>{q.label}</option>)}
                            </select>
                          </Td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </LightTableCard>
            </div>
          )}

          {tab === "pricing" && (
            <div style={{ color: "#6b7280" }}>
              Pricing for beef will go here after we lock calculations and any grid/discount tables.
            </div>
          )}
        </div>
      );
    }

    // ===== Reusable UI =====
    function LightTableCard({ children }) {
      return (
        <div
          style={{
            overflowX: "auto",
            border: `1px solid ${TT.border}`,
            borderRadius: 10,
            background: TT.card,
            padding: 8,
            boxShadow: "0 1px 2px rgba(0,0,0,0.04), 0 4px 16px rgba(0,0,0,0.03)"
          }}
        >
          {children}
        </div>
      );
    }

    const Th = ({ children }) => (
      <th style={{
        border: `1px solid ${TT.grid}`,
        padding: 10,
        background: TT.head,
        color: TT.headText,
        textAlign: "left",
        whiteSpace: "nowrap",
        fontSize: 13,
        position: "sticky",
        top: 0
      }}>{children}</th>
    );

    const Td = ({ children }) => (
      <td style={{
        border: `1px solid ${TT.grid}`,
        padding: 8,
        verticalAlign: "middle",
        color: TT.text,
        background: TT.white
      }}>{children}</td>
    );

    const inputStyle = {
      width: "100%",
      padding: "6px 8px",
      border: `1px solid ${TT.grid}`,
      borderRadius: 8,
      background: TT.white,
      color: TT.text
    };

    const selectStyle = {
      width: "100%",
      padding: "6px 8px",
      border: `1px solid ${TT.grid}`,
      borderRadius: 8,
      background: TT.white,
      color: TT.text
    };

    function InnerTab({ active, onClick, children }) {
      return (
        <button
          onClick={onClick}
          type="button"
          style={{
            padding: "8px 14px",
            border: active ? `2px solid ${TT.redDark}` : `1px solid ${TT.border}`,
            background: active ? TT.red : "transparent",
            color: active ? "#ffffff" : TT.text,
            fontWeight: 800,
            borderRadius: 8,
            boxShadow: active ? "inset 0 -2px 0 rgba(0,0,0,0.45)" : "none",
            cursor: "pointer"
          }}
        >
          {children}
        </button>
      );
    }

    const Cell = ({ value, onChange, placeholder }) => (
      <input
        type="text"
        inputMode="decimal"
        value={value}
        onChange={(e) => onChange(e.target.value)}
        placeholder={placeholder}
        style={inputStyle}
      />
    );

    const LabeledInput = ({ label, name, value, onChange, placeholder }) => (
      <div>
        <label style={{ display: "block", fontWeight: 600, marginBottom: 4 }}>{label}</label>
        <input name={name} value={value} onChange={onChange} inputMode="decimal" placeholder={placeholder} style={inputStyle} />
      </div>
    );

    const Card = ({ title, value }) => (
      <div style={{ padding: 14, border: `1px solid ${TT.border}`, borderRadius: 8, background: TT.card, color: TT.text }}>
        <div style={{ fontSize: 12, color: "#6b7280", marginBottom: 4 }}>{title}</div>
        <div style={{ fontSize: 20, fontWeight: 700 }}>{value}</div>
      </div>
    );

    const styles = {
      btnPrimary: {
        padding: "8px 14px",
        borderRadius: 8,
        border: `2px solid ${TT.redDark}`,
        background: TT.red,
        color: "#ffffff",
        fontWeight: 700,
        cursor: "pointer"
      },
      btnSecondary: {
        padding: "8px 14px",
        borderRadius: 8,
        border: `1px solid ${TT.border}`,
        background: TT.card,
        color: TT.text,
        cursor: "pointer"
      }
    };

    // Mount
    const root = ReactDOM.createRoot(document.getElementById("root"));
    root.render(<App />);
  </script>
</body>
</html>
