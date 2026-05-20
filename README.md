<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5, viewport-fit=cover">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="FOSB Calculator">
<meta name="theme-color" content="#000000">
<title>FOSB's Agent Calculator</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
<style>
  /* ============ REVOLUT-INSPIRED REDESIGN ============
     Two-mode canvas: pure white catalogue + black hero bands.
     Display type: Aeonik-style (heavy, tight). Body: Inter.
     Pill buttons (fully rounded). 20px rounded cards. 12px rounded inputs.
     Single red accent (#D23C35 — from FOSB logo). */

  :root {
    /* Canvas */
    --canvas-light: #ffffff;       /* white catalogue mode */
    --canvas-dark: #000000;        /* black hero mode */
    --surface-soft: #f6f6f6;       /* subtle off-white surface */
    --surface-elev: #fafafa;       /* slightly elevated surface */

    /* Text */
    --ink: #0a0a0a;                /* primary text on light */
    --ink-soft: #6b6b6b;            /* secondary text */
    --ink-faint: #a3a3a3;           /* tertiary / placeholder */
    --on-dark: #ffffff;             /* text on black */
    --on-dark-soft: rgba(255,255,255,0.65);

    /* Hairlines */
    --line: #ececec;
    --line-strong: #d4d4d4;

    /* Brand red (extracted from FOSB logo) */
    --brand: #D23C35;
    --brand-deep: #9F2A24;
    --brand-soft: #fef0ef;
    --brand-hover: #B8332C;

    /* Functional */
    --success: #15803d;
    --success-soft: #dcfce7;
    --danger: #b91c1c;
    --warn-soft: #fef3c7;
    --warn-deep: #92400e;

    /* Radii — Revolut uses pills, 20px cards, 12px chips */
    --r-pill: 999px;
    --r-card: 20px;
    --r-chip: 12px;
    --r-input: 12px;

    /* Shadows — very subtle */
    --shadow-xs: 0 1px 2px rgba(0,0,0,0.04);
    --shadow-sm: 0 2px 8px rgba(0,0,0,0.04);
    --shadow-md: 0 4px 20px rgba(0,0,0,0.06);
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  html, body {
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    background: #F5F5F5;
    color: var(--ink);
    font-size: 15px;
    line-height: 1.5;
    -webkit-font-smoothing: antialiased;
    text-rendering: optimizeLegibility;
    letter-spacing: -0.005em;
    min-height: 100vh;
  }

  .page {
    max-width: 1280px;
    margin: 0 auto;
    padding: 32px 32px 80px;
  }

  /* ============ HERO HEADER (dark band) ============ */
  header {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 24px;
    align-items: center;
    padding: 28px 32px;
    background: var(--canvas-dark);
    color: var(--on-dark);
    border-radius: var(--r-card);
    margin-bottom: 28px;
    overflow: hidden;
    position: relative;
  }
  header::before {
    content: '';
    position: absolute;
    inset: 0;
    background:
      radial-gradient(circle at 90% 50%, rgba(210,60,53,0.25), transparent 50%),
      radial-gradient(circle at 5% 100%, rgba(210,60,53,0.12), transparent 40%);
    pointer-events: none;
  }
  .brand {
    display: flex; align-items: center; gap: 16px;
    position: relative; z-index: 1;
  }
  .brand .logo-img {
    width: 56px; height: 56px;
    background: var(--canvas-light);
    border-radius: var(--r-chip);
    padding: 6px;
    display: flex; align-items: center; justify-content: center;
    flex-shrink: 0;
  }
  .brand .logo-img img { width: 100%; height: 100%; object-fit: contain; }
  .brand .brand-text { display: flex; flex-direction: column; gap: 2px; }
  .brand .eyebrow {
    font-size: 11px;
    text-transform: uppercase;
    letter-spacing: 0.12em;
    color: var(--on-dark-soft);
    font-weight: 500;
  }
  .brand h1 {
    font-family: 'Inter', sans-serif;
    font-weight: 700;
    font-size: 32px;
    letter-spacing: -0.035em;
    line-height: 1;
    color: var(--on-dark);
  }
  .brand h1 em,
  .brand h1 .wordmark {
    font-style: normal;
    color: var(--brand);
    font-weight: 700;
  }
  .meta {
    display: flex; gap: 12px; align-items: center;
    font-size: 12px;
    color: var(--on-dark-soft);
    position: relative; z-index: 1;
    flex-wrap: wrap;
    justify-content: flex-end;
  }
  .meta > span {
    display: inline-flex; align-items: center; gap: 6px;
    padding: 6px 14px;
    background: rgba(255,255,255,0.08);
    border-radius: var(--r-pill);
    backdrop-filter: blur(8px);
  }
  .meta .dot {
    width: 6px; height: 6px; border-radius: 50%;
    background: var(--success);
    box-shadow: 0 0 0 3px rgba(21,128,61,0.25);
  }

  /* ============ LAYOUT ============ */
  .layout {
    display: grid;
    grid-template-columns: 1fr 380px;
    gap: 20px;
    align-items: start;
  }
  @media (max-width: 980px) {
    .layout { grid-template-columns: 1fr; }
  }
  .stack { display: flex; flex-direction: column; gap: 16px; }

  /* ============ CARDS ============ */
  .card {
    background: var(--canvas-light);
    border: 1px solid var(--line);
    border-radius: var(--r-card);
    overflow: hidden;
    transition: border-color 200ms, box-shadow 200ms;
    box-shadow: var(--shadow-xs);
  }
  .card:hover { border-color: var(--line-strong); box-shadow: var(--shadow-sm); }
  .card-header {
    display: flex; align-items: center; justify-content: space-between;
    padding: 18px 24px;
    border-bottom: 1px solid var(--line);
  }
  .card-title {
    font-size: 13px;
    font-weight: 600;
    color: var(--ink);
    letter-spacing: -0.01em;
  }
  .card-title strong { color: var(--ink); }
  .card-title.muted { color: var(--ink-faint); font-weight: 500; }
  .card-subtitle {
    font-size: 12px;
    color: var(--ink-faint);
    margin-left: 8px;
    font-weight: 400;
  }
  .card-body { padding: 4px 0; }

  /* ============ EMPTY STATE ============ */
  .empty-state {
    padding: 64px 24px;
    text-align: center;
    color: var(--ink-faint);
    font-size: 14px;
  }
  .empty-state .big {
    font-size: 24px;
    font-weight: 700;
    letter-spacing: -0.02em;
    color: var(--ink);
    margin-bottom: 8px;
  }

  /* Inline "Add another product" CTA at end of quote list */
  .quote-addmore {
    padding: 20px 24px;
    display: flex; justify-content: center;
    background: var(--surface-soft);
    border-top: 1px dashed var(--line-strong);
  }
  .quote-addmore-btn {
    display: inline-flex; align-items: center; gap: 8px;
    padding: 10px 22px;
    background: var(--canvas-light);
    border: 1px dashed var(--line-strong);
    border-radius: var(--r-pill);
    color: var(--ink-soft);
    font-family: 'Inter', sans-serif;
    font-size: 13px; font-weight: 600;
    cursor: pointer;
    transition: all 160ms;
    letter-spacing: -0.005em;
  }
  .quote-addmore-btn:hover {
    border-color: var(--brand);
    color: var(--brand);
    background: var(--brand-soft);
    border-style: solid;
  }
  .quote-addmore-btn .icon {
    font-size: 16px; font-weight: 700; line-height: 1;
  }

  /* ============ BUTTONS (pill shaped) ============ */
  .btn {
    font-family: 'Inter', sans-serif;
    font-size: 14px;
    font-weight: 600;
    padding: 10px 22px;
    border: 1px solid var(--line-strong);
    border-radius: var(--r-pill);
    background: var(--canvas-light);
    color: var(--ink);
    cursor: pointer;
    transition: all 160ms;
    display: inline-flex; align-items: center; gap: 8px;
    line-height: 1;
    height: 40px;
    letter-spacing: -0.005em;
  }
  .btn:hover {
    background: var(--surface-soft);
    border-color: var(--ink);
  }
  .btn-primary {
    background: var(--canvas-dark);
    color: var(--on-dark);
    border-color: var(--canvas-dark);
  }
  .btn-primary:hover {
    background: #1a1a1a;
    border-color: #1a1a1a;
  }
  .btn-brand {
    background: var(--brand);
    color: var(--on-dark);
    border-color: var(--brand);
  }
  .btn-brand:hover {
    background: var(--brand-hover);
    border-color: var(--brand-hover);
  }
  .btn-lg { padding: 12px 28px; font-size: 15px; height: 46px; }
  .btn .icon {
    font-size: 16px; line-height: 1; font-weight: 700;
  }
  .actions-bar {
    display: flex; gap: 8px;
    padding: 18px 24px;
    background: var(--surface-soft);
    border-top: 1px solid var(--line);
    flex-wrap: wrap;
  }

  /* ============ QUOTE LIST ============ */
  .quote-list { padding: 0; }
  .quote-item {
    border-bottom: 1px solid var(--line);
    padding: 24px;
    transition: background 160ms;
  }
  .quote-item:last-child { border-bottom: none; }
  .quote-item:hover { background: var(--surface-elev); }

  .quote-item-head {
    display: grid;
    grid-template-columns: auto 1fr auto;
    gap: 16px;
    align-items: center;
    margin-bottom: 16px;
  }
  .quote-item-head .num {
    width: 36px; height: 36px;
    border-radius: var(--r-pill);
    background: var(--canvas-dark);
    color: var(--on-dark);
    display: flex; align-items: center; justify-content: center;
    font-size: 12px; font-weight: 700;
    letter-spacing: -0.01em;
  }
  .quote-item-head .head-title { min-width: 0; }
  .quote-item-head .name {
    font-weight: 700; font-size: 18px;
    letter-spacing: -0.02em;
    overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
    color: var(--ink);
  }
  .quote-item-head .name .plus {
    display: inline-block;
    font-size: 11px; font-weight: 600;
    color: var(--brand-deep);
    background: var(--brand-soft);
    padding: 3px 10px;
    border-radius: var(--r-pill);
    margin-left: 8px;
    vertical-align: middle;
    letter-spacing: 0;
  }
  .quote-item-head .qty-line {
    font-size: 13px;
    color: var(--ink-soft);
    margin-top: 8px;
    display: flex; align-items: center; gap: 10px;
  }
  .quote-item-head .qty-line .qty-label {
    text-transform: uppercase;
    letter-spacing: 0.08em;
    font-size: 10px;
    color: var(--ink-faint);
    font-weight: 600;
  }
  .quote-item-head .qty-line .qty-unit {
    color: var(--ink-faint); font-size: 12px;
  }
  .quote-item-head .qty-edit {
    display: inline-flex; align-items: stretch;
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    overflow: hidden;
    background: var(--canvas-light);
    transition: border-color 160ms, box-shadow 160ms;
  }
  .quote-item-head .qty-edit:focus-within {
    border-color: var(--brand);
    box-shadow: 0 0 0 4px var(--brand-soft);
  }
  .quote-item-head .qty-edit button {
    background: transparent; border: none;
    width: 32px; height: 34px;
    cursor: pointer;
    font-size: 16px; line-height: 1;
    color: var(--ink-soft);
    font-weight: 600;
    transition: background 160ms, color 160ms;
  }
  .quote-item-head .qty-edit button:hover {
    background: var(--surface-soft); color: var(--ink);
  }
  .quote-item-head .qty-edit-input {
    border: none; background: transparent;
    width: 60px; padding: 0 6px;
    font-family: 'Inter', sans-serif;
    font-size: 15px; font-weight: 700;
    text-align: center;
    color: var(--ink);
    -moz-appearance: textfield;
    letter-spacing: -0.01em;
  }
  .quote-item-head .qty-edit-input::-webkit-inner-spin-button,
  .quote-item-head .qty-edit-input::-webkit-outer-spin-button {
    -webkit-appearance: none; margin: 0;
  }
  .quote-item-head .qty-edit-input:focus { outline: none; }
  .quote-item-head .actions { display: flex; gap: 6px; }

  /* ============ BREAKDOWN ============ */
  .breakdown {
    background: var(--surface-soft);
    border-radius: var(--r-chip);
    padding: 6px 0;
    margin-bottom: 16px;
  }
  .bd-row {
    display: grid;
    grid-template-columns: 1fr auto;
    align-items: center;
    gap: 20px;
    padding: 12px 18px;
    border-bottom: 1px solid var(--line);
  }
  .bd-row:last-child { border-bottom: none; }
  .bd-comp {
    display: flex; align-items: center; gap: 12px;
    min-width: 0;
  }
  .bd-tag {
    font-size: 9px;
    text-transform: uppercase; letter-spacing: 0.1em;
    padding: 4px 10px; border-radius: var(--r-pill);
    font-weight: 700;
    flex-shrink: 0;
    line-height: 1;
  }
  .bd-tag.base {
    background: var(--canvas-dark); color: var(--on-dark);
  }
  .bd-tag.addon {
    background: var(--brand-soft); color: var(--brand-deep);
  }
  .bd-name {
    font-size: 14px; font-weight: 600; color: var(--ink);
    overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
    min-width: 0; letter-spacing: -0.005em;
  }
  .bd-code {
    font-size: 11px; font-weight: 500;
    color: var(--ink-faint);
    flex-shrink: 0;
    padding: 2px 8px;
    background: var(--canvas-light);
    border-radius: var(--r-pill);
    border: 1px solid var(--line);
  }
  .bd-calc {
    display: flex; align-items: center; gap: 8px;
    font-variant-numeric: tabular-nums;
    font-size: 13px;
    color: var(--ink-soft);
    flex-shrink: 0;
  }
  .bd-unit { color: var(--ink); font-weight: 600; }
  /* Clickable add-on unit price (editable for discount) */
  button.bd-unit-editable {
    background: transparent;
    border: 1px dashed transparent;
    color: var(--ink); font-weight: 600;
    padding: 2px 8px;
    border-radius: var(--r-pill);
    cursor: pointer;
    font-family: inherit; font-size: inherit;
    line-height: 1.2;
    transition: all 140ms;
  }
  button.bd-unit-editable:hover {
    border-color: var(--brand);
    background: var(--brand-soft);
    color: var(--brand-deep);
  }
  button.bd-unit-editable:hover::after {
    content: ' ✎';
    font-size: 10px;
    opacity: 0.7;
  }
  .bd-row.overridden button.bd-unit-editable {
    color: #15803d; font-weight: 700;
  }
  /* Inline input shown when editing */
  input.bd-unit-input {
    width: 80px;
    padding: 4px 10px;
    border: 1px solid var(--brand);
    border-radius: var(--r-pill);
    background: var(--canvas-light);
    color: var(--ink);
    font-family: inherit;
    font-size: 13px;
    font-weight: 700;
    text-align: center;
    box-shadow: 0 0 0 4px var(--brand-soft);
    -moz-appearance: textfield;
  }
  input.bd-unit-input:focus { outline: none; }
  input.bd-unit-input::-webkit-inner-spin-button,
  input.bd-unit-input::-webkit-outer-spin-button {
    -webkit-appearance: none; margin: 0;
  }
  .bd-x, .bd-eq { color: var(--ink-faint); font-size: 12px; }
  .bd-qty { color: var(--ink); font-weight: 600; }
  .bd-sub {
    color: var(--ink); font-weight: 700;
    font-size: 15px;
    min-width: 100px; text-align: right;
    letter-spacing: -0.01em;
  }
  /* Count badge */
  .bd-count-badge {
    display: inline-block;
    font-size: 10px; font-weight: 700;
    padding: 3px 9px; border-radius: var(--r-pill);
    background: var(--canvas-dark); color: var(--on-dark);
    margin-left: 8px; vertical-align: middle;
    letter-spacing: 0;
  }
  /* Inline qty editor in breakdown (compact) */
  .bd-qty-edit {
    display: inline-flex; align-items: stretch;
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    overflow: hidden;
    background: var(--canvas-light);
    flex-shrink: 0;
    transition: border-color 160ms, box-shadow 160ms;
  }
  .bd-qty-edit:focus-within {
    border-color: var(--brand);
    box-shadow: 0 0 0 4px var(--brand-soft);
  }
  .bd-qty-edit button {
    background: transparent; border: none;
    width: 26px; height: 28px;
    cursor: pointer;
    font-size: 13px; line-height: 1;
    color: var(--ink-soft); font-weight: 600;
    transition: background 160ms, color 160ms;
  }
  .bd-qty-edit button:hover {
    background: var(--surface-soft); color: var(--ink);
  }
  .bd-qty-input {
    border: none; background: transparent;
    width: 50px; padding: 0 4px;
    font-family: 'Inter', sans-serif;
    font-size: 13px; font-weight: 700;
    text-align: center;
    color: var(--ink);
    -moz-appearance: textfield;
  }
  .bd-qty-input::-webkit-inner-spin-button,
  .bd-qty-input::-webkit-outer-spin-button {
    -webkit-appearance: none; margin: 0;
  }
  .bd-qty-input:focus { outline: none; }

  /* Reset / remove buttons on breakdown rows */
  .bd-reset, .bd-remove {
    background: transparent;
    border: 1px solid transparent;
    color: var(--ink-faint);
    cursor: pointer;
    width: 26px; height: 26px;
    margin-left: 4px;
    border-radius: var(--r-pill);
    font-size: 13px; line-height: 1;
    display: inline-flex; align-items: center; justify-content: center;
    transition: all 160ms;
    flex-shrink: 0;
    font-weight: 600;
  }
  .bd-row.addon-row:hover .bd-remove,
  .bd-row.overridden .bd-reset {
    border-color: var(--line);
    background: var(--canvas-light);
  }
  .bd-reset:hover {
    color: var(--ink); border-color: var(--ink); background: var(--canvas-light);
  }
  .bd-remove:hover {
    color: var(--canvas-light) !important;
    border-color: var(--danger) !important;
    background: var(--danger) !important;
  }

  /* Add-more dashed button */
  .bd-addmore-row {
    padding: 8px 18px !important;
    grid-template-columns: 1fr !important;
  }
  .bd-addmore {
    background: transparent;
    border: 1px dashed var(--line-strong);
    color: var(--ink-soft);
    padding: 8px 14px;
    border-radius: var(--r-pill);
    font-size: 12px; font-weight: 600;
    cursor: pointer;
    transition: all 160ms;
    width: 100%;
  }
  .bd-addmore:hover {
    border-color: var(--brand);
    color: var(--brand);
    background: var(--brand-soft);
    border-style: solid;
  }

  /* ============ FOOT UNIT + TOTAL ============ */
  .quote-item-foot {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
  }
  .foot-unit, .foot-total {
    display: flex; flex-direction: column;
    padding: 16px 20px;
    border-radius: var(--r-chip);
  }
  .foot-unit {
    background: var(--surface-soft);
    border: 1px solid var(--line);
  }
  .foot-total {
    background: var(--canvas-dark);
    color: var(--on-dark);
  }
  .foot-unit .lbl, .foot-total .lbl {
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    font-weight: 600;
    margin-bottom: 4px;
  }
  .foot-unit .lbl { color: var(--ink-faint); }
  .foot-total .lbl { color: var(--on-dark-soft); }
  .foot-unit .val, .foot-total .val {
    font-family: 'Inter', sans-serif;
    font-variant-numeric: tabular-nums;
    font-weight: 700;
    letter-spacing: -0.025em;
    line-height: 1;
  }
  .foot-unit .val { font-size: 22px; color: var(--ink); }
  .foot-total .val { font-size: 28px; color: var(--on-dark); }

  /* Icon buttons in head */
  .icon-btn {
    background: transparent;
    border: 1px solid var(--line);
    color: var(--ink-soft);
    cursor: pointer;
    width: 36px; height: 36px;
    border-radius: var(--r-pill);
    font-size: 14px; line-height: 1;
    display: inline-flex; align-items: center; justify-content: center;
    transition: all 160ms;
  }
  .icon-btn:hover {
    color: var(--ink); border-color: var(--ink);
    background: var(--surface-soft);
  }
  .icon-btn.danger:hover {
    color: var(--canvas-light);
    border-color: var(--danger);
    background: var(--danger);
  }

  /* ============ UPSELL BANNERS ============ */
  .upsell-banner {
    display: flex;
    align-items: center;
    gap: 14px;
    padding: 14px 18px;
    margin: 0 0 14px 0;
    border-radius: var(--r-chip);
    font-size: 13px;
    animation: fadeIn 200ms ease-out;
  }
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(-4px); }
    to   { opacity: 1; transform: translateY(0); }
  }
  .upsell-banner.savings {
    background: var(--success-soft);
    border: 1px solid #86efac;
    color: #14532d;
  }
  .upsell-banner.tip {
    background: var(--warn-soft);
    border: 1px solid #fde68a;
    color: var(--warn-deep);
  }
  .upsell-icon { font-size: 20px; flex-shrink: 0; }
  .upsell-msg {
    flex: 1; line-height: 1.5;
    min-width: 200px;
  }
  .upsell-msg strong { font-weight: 700; }
  .upsell-actions { display: flex; gap: 6px; flex-shrink: 0; flex-wrap: wrap; }
  .upsell-btn {
    font-family: 'Inter', sans-serif;
    font-size: 12px; font-weight: 600;
    padding: 6px 14px;
    border-radius: var(--r-pill);
    cursor: pointer;
    flex-shrink: 0;
    transition: all 160ms;
    white-space: nowrap;
    line-height: 1;
  }
  .upsell-banner.savings .upsell-btn {
    background: #15803d; color: white; border: 1px solid #15803d;
  }
  .upsell-banner.savings .upsell-btn:hover { background: #14532d; }
  .upsell-banner.savings .upsell-btn.alt {
    background: transparent; color: #14532d; border-color: #86efac;
  }
  .upsell-banner.savings .upsell-btn.alt:hover {
    background: #86efac; color: #14532d;
  }
  .upsell-banner.tip .upsell-btn {
    background: var(--warn-deep); color: white; border: 1px solid var(--warn-deep);
  }
  .upsell-banner.tip .upsell-btn:hover { background: #7c2d12; }
  .upsell-banner.tip .upsell-btn.alt {
    background: transparent; color: var(--warn-deep); border-color: #fde68a;
  }
  .upsell-banner.tip .upsell-btn.alt:hover {
    background: #fde68a; color: var(--warn-deep);
  }
  .upsell-dismiss {
    background: transparent; border: none; cursor: pointer;
    color: currentColor; opacity: 0.5;
    font-size: 18px; line-height: 1; padding: 4px 6px;
    border-radius: var(--r-pill);
    flex-shrink: 0;
    font-weight: 600;
  }
  .upsell-dismiss:hover { opacity: 1; }

  /* Override badge */
  .override-badge {
    display: inline-block;
    font-size: 9px; font-weight: 700;
    text-transform: uppercase; letter-spacing: 0.08em;
    padding: 3px 8px; border-radius: var(--r-pill);
    background: #15803d; color: #fff;
    margin-left: 8px;
    vertical-align: middle;
    line-height: 1;
  }
  .bd-row.overridden { background: rgba(34, 197, 94, 0.04); }
  .bd-row.overridden .bd-unit { color: #15803d; font-weight: 700; }

  /* ============ SUMMARY SIDEBAR ============ */
  .summary {
    background: var(--canvas-light);
    border: 1px solid var(--line);
    border-radius: var(--r-card);
    overflow: hidden;
    position: sticky; top: 24px;
    align-self: start;
    box-shadow: var(--shadow-xs);
  }
  .summary-row {
    display: flex; justify-content: space-between; align-items: baseline;
    padding: 18px 24px;
    border-bottom: 1px solid var(--line);
  }
  .summary-row:last-child { border-bottom: none; }
  .summary-row .label {
    font-size: 12px;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    color: var(--ink-soft);
    font-weight: 600;
  }
  .summary-row .value {
    font-family: 'Inter', sans-serif;
    font-size: 22px;
    font-weight: 700;
    font-variant-numeric: tabular-nums;
    letter-spacing: -0.025em;
    color: var(--ink);
  }
  .summary-row.subtotal { background: var(--surface-soft); }
  .summary-row.subtotal .value {
    font-size: 16px; color: var(--ink-soft); font-weight: 600;
  }
  /* Profit row — green highlight, money you take home */
  .summary-row.profit-row {
    background: var(--success-soft);
    border-bottom: 1px solid #bbf7d0;
  }
  .summary-row.profit-row .label {
    color: #14532d;
    display: flex; align-items: center; gap: 6px;
  }
  .summary-row.profit-row .label::before {
    content: '↑';
    font-size: 14px; font-weight: 800;
    color: #15803d;
    line-height: 1;
  }
  .summary-row.profit-row .value {
    color: #14532d;
    font-size: 22px;
    font-weight: 800;
  }

  /* Markup control */
  .markup-control {
    padding: 20px 24px;
    border-bottom: 1px solid var(--line);
    background: var(--surface-soft);
  }
  .markup-control label {
    display: flex; justify-content: space-between; align-items: center;
    font-size: 11px;
    text-transform: uppercase; letter-spacing: 0.1em;
    color: var(--ink-soft); font-weight: 700;
    margin-bottom: 12px;
  }
  .markup-control label span {
    color: var(--ink); font-size: 13px;
    letter-spacing: -0.01em;
  }
  .markup-control .markup-row { display: flex; gap: 8px; align-items: center; }
  .markup-control .preset-btns { display: flex; gap: 4px; flex: 1; }
  .preset-btn {
    flex: 1; padding: 8px 4px;
    font-family: 'Inter', sans-serif;
    font-size: 12px; font-weight: 600;
    background: var(--canvas-light);
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    cursor: pointer; color: var(--ink-soft);
    transition: all 160ms;
  }
  .preset-btn:hover { border-color: var(--ink); color: var(--ink); }
  .preset-btn.active {
    background: var(--canvas-dark); color: var(--on-dark); border-color: var(--canvas-dark);
  }
  .markup-control input[type="number"] {
    text-align: center; font-weight: 700; font-size: 15px;
    padding: 8px 10px; width: 76px;
    background: var(--canvas-light);
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    font-family: 'Inter', sans-serif;
    -moz-appearance: textfield;
  }
  .markup-control input[type="number"]:focus {
    outline: none; border-color: var(--brand);
    box-shadow: 0 0 0 4px var(--brand-soft);
  }
  .markup-control input[type="number"]::-webkit-inner-spin-button,
  .markup-control input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none; margin: 0;
  }

  /* Suggested price hero */
  .suggest-row {
    background: var(--canvas-dark);
    color: var(--on-dark);
    position: relative;
    overflow: hidden;
  }
  .suggest-row::before {
    content: '';
    position: absolute; inset: 0;
    background: radial-gradient(circle at 100% 100%, rgba(210,60,53,0.3), transparent 60%);
    pointer-events: none;
  }
  .suggest-row .label { color: var(--on-dark-soft); position: relative; z-index: 1; }
  .suggest-row .value {
    color: var(--on-dark); font-size: 32px;
    position: relative; z-index: 1;
  }

  /* Platform picker */
  .platform-control {
    padding: 18px 24px;
    border-bottom: 1px solid var(--line);
    background: var(--canvas-light);
  }
  .platform-header { margin-bottom: 12px; }
  .platform-header .ph-title {
    font-size: 11px; text-transform: uppercase;
    letter-spacing: 0.1em; color: var(--ink-soft);
    font-weight: 700; display: block; margin-bottom: 2px;
  }
  .platform-header .ph-hint {
    font-size: 12px; color: var(--ink-faint);
    line-height: 1.4;
  }
  .platform-pills {
    display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 6px;
  }
  .platform-pill {
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    gap: 6px;
    padding: 12px 6px;
    border: 1px solid var(--line);
    border-radius: var(--r-chip);
    background: var(--canvas-light);
    color: var(--ink-soft);
    cursor: pointer;
    font-family: 'Inter', sans-serif;
    font-size: 12px; font-weight: 600;
    transition: all 160ms;
  }
  .platform-pill:hover { border-color: var(--line-strong); background: var(--surface-soft); }
  .platform-pill .pp-icon {
    font-size: 14px; font-weight: 700;
    width: 26px; height: 26px;
    border-radius: var(--r-pill);
    display: inline-flex; align-items: center; justify-content: center;
    background: var(--line); color: var(--ink-soft);
    transition: all 160ms;
  }
  .platform-pill .pp-icon.shopee { background: #ee4d2d; color: white; }
  .platform-pill .pp-icon.lazada { background: #0f146d; color: white; }
  .platform-pill .pp-rate {
    font-size: 10px; color: var(--ink-faint);
    font-weight: 600;
    letter-spacing: 0.02em;
  }
  .platform-pill[data-platform="none"].active {
    background: var(--canvas-dark); color: var(--on-dark); border-color: var(--canvas-dark);
  }
  .platform-pill[data-platform="none"].active .pp-icon {
    background: var(--canvas-light); color: var(--ink);
  }
  .platform-pill[data-platform="none"].active .pp-rate { color: rgba(255,255,255,0.7); }
  .platform-pill[data-platform="shopee"].active {
    background: #ee4d2d; border-color: #ee4d2d; color: white;
  }
  .platform-pill[data-platform="shopee"].active .pp-icon { background: white; color: #ee4d2d; }
  .platform-pill[data-platform="shopee"].active .pp-rate { color: rgba(255,255,255,0.85); }
  .platform-pill[data-platform="lazada"].active {
    background: #0f146d; border-color: #0f146d; color: white;
  }
  .platform-pill[data-platform="lazada"].active .pp-icon { background: white; color: #0f146d; }
  .platform-pill[data-platform="lazada"].active .pp-rate { color: rgba(255,255,255,0.85); }

  .platform-rate-edit {
    margin-top: 12px;
    display: flex; align-items: center; gap: 10px;
    padding: 10px 14px;
    background: var(--surface-soft);
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
  }
  .platform-rate-edit .rate-lbl {
    flex: 1; font-size: 11px;
    color: var(--ink-soft); font-weight: 600;
    text-transform: uppercase; letter-spacing: 0.06em;
  }
  .platform-rate-edit input {
    width: 64px; padding: 5px 10px;
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    text-align: center;
    font-family: 'Inter', sans-serif;
    font-size: 13px; font-weight: 700;
    background: var(--canvas-light);
    -moz-appearance: textfield;
  }
  .platform-rate-edit input:focus {
    outline: none; border-color: var(--brand);
    box-shadow: 0 0 0 4px var(--brand-soft);
  }
  .platform-rate-edit input::-webkit-inner-spin-button,
  .platform-rate-edit input::-webkit-outer-spin-button {
    -webkit-appearance: none; margin: 0;
  }
  .platform-rate-edit .rate-pct {
    font-size: 12px; color: var(--ink-soft); font-weight: 600;
  }

  .platform-row.shopee { background: #fff5f1; }
  .platform-row.shopee .label,
  .platform-row.shopee .value { color: #c2410c; }
  .platform-row.shopee .value { font-size: 16px; font-weight: 700; }
  .platform-row.lazada { background: #eef0fa; }
  .platform-row.lazada .label,
  .platform-row.lazada .value { color: #0f146d; }
  .platform-row.lazada .value { font-size: 16px; font-weight: 700; }
  .platform-final-row.shopee {
    background: #ee4d2d; color: white;
    position: relative; overflow: hidden;
  }
  .platform-final-row.lazada {
    background: #0f146d; color: white;
    position: relative; overflow: hidden;
  }
  .platform-final-row .label { color: rgba(255,255,255,0.85); }
  .platform-final-row .value { color: var(--on-dark); font-size: 22px; }

  .tier-hint {
    font-size: 11px;
    color: var(--ink-faint);
    padding: 14px 24px;
    background: var(--surface-soft);
    border-top: 1px solid var(--line);
    line-height: 1.6;
  }

  /* ============ MODAL ============ */
  .modal-backdrop {
    position: fixed;
    top: 0; right: 0; bottom: 0; left: 0;
    background: rgba(0,0,0,0.6);
    display: none;
    z-index: 100;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
  }
  .modal-backdrop.show {
    display: block;
    animation: fadeBackdrop 200ms ease-out;
  }
  @keyframes fadeBackdrop {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  .modal {
    background: var(--canvas-light);
    border-radius: var(--r-card);
    width: calc(100% - 40px);
    max-width: 760px;
    margin: 60px auto;
    box-shadow: var(--shadow-md);
    overflow: hidden;
    position: relative;
  }
  .modal-header {
    padding: 24px 28px;
    border-bottom: 1px solid var(--line);
    display: flex; justify-content: space-between; align-items: center;
  }
  .modal-header h2 {
    font-family: 'Inter', sans-serif;
    font-size: 24px; font-weight: 700;
    letter-spacing: -0.025em;
  }
  .modal-header .close {
    background: transparent; border: none; cursor: pointer;
    font-size: 26px; color: var(--ink-faint);
    width: 40px; height: 40px;
    border-radius: var(--r-pill);
    display: flex; align-items: center; justify-content: center;
    transition: all 160ms;
  }
  .modal-header .close:hover {
    color: var(--ink); background: var(--surface-soft);
  }
  .modal-body { padding: 28px; max-height: 70vh; overflow-y: auto; }
  .modal-footer {
    padding: 18px 28px;
    border-top: 1px solid var(--line);
    background: var(--surface-soft);
    display: flex; justify-content: space-between; align-items: center;
    gap: 16px;
  }
  .modal-footer .preview {
    font-size: 12px; color: var(--ink-soft);
    line-height: 1.6;
  }
  .modal-footer .preview > div {
    display: flex; align-items: baseline; gap: 8px;
  }
  .modal-footer .preview strong {
    color: var(--ink); font-size: 16px;
    font-weight: 700; margin-left: 6px;
    letter-spacing: -0.01em;
  }
  .modal-footer .preview > div:last-child strong {
    font-size: 20px;
  }

  /* Form steps */
  .step { margin-bottom: 32px; }
  .step:last-child { margin-bottom: 0; }
  .step-label {
    font-size: 11px;
    text-transform: uppercase; letter-spacing: 0.1em;
    color: var(--ink-soft); margin-bottom: 14px;
    display: flex; align-items: center; gap: 10px;
    font-weight: 700;
  }
  .step-num {
    width: 24px; height: 24px;
    background: var(--canvas-dark); color: var(--on-dark);
    border-radius: var(--r-pill);
    display: inline-flex; align-items: center; justify-content: center;
    font-size: 11px; font-weight: 700;
  }

  /* Search box */
  .search-box {
    width: 100%; padding: 13px 18px;
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    font-family: 'Inter', sans-serif; font-size: 14px;
    background: var(--canvas-light);
    margin-bottom: 14px;
  }
  .search-box:focus {
    outline: none; border-color: var(--brand);
    box-shadow: 0 0 0 4px var(--brand-soft);
  }

  /* Category pills */
  .cat-pills {
    display: flex; flex-wrap: wrap; gap: 6px;
    margin-bottom: 16px;
  }
  .cat-pill {
    font-family: 'Inter', sans-serif; font-size: 12px; font-weight: 600;
    padding: 7px 14px;
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    background: var(--canvas-light); color: var(--ink-soft);
    cursor: pointer; transition: all 160ms;
  }
  .cat-pill:hover { border-color: var(--ink); color: var(--ink); }
  .cat-pill.active {
    background: var(--canvas-dark); color: var(--on-dark); border-color: var(--canvas-dark);
  }
  .cat-pill .count {
    font-size: 10px; opacity: 0.6; margin-left: 4px;
    font-weight: 500;
  }

  /* Product list */
  .product-grid {
    display: grid; gap: 6px;
    max-height: 300px; overflow-y: auto;
    padding: 6px;
    border: 1px solid var(--line);
    border-radius: var(--r-chip);
    background: var(--surface-soft);
  }
  .product-option {
    display: flex; justify-content: space-between; align-items: center;
    padding: 12px 14px;
    background: var(--canvas-light);
    border: 1px solid transparent;
    border-radius: var(--r-chip);
    cursor: pointer; transition: all 160ms;
  }
  .product-option:hover { border-color: var(--line-strong); }
  .product-option.selected {
    border-color: var(--brand);
    background: var(--brand-soft);
  }
  .product-option .left { display: flex; align-items: center; gap: 14px; }
  .product-option .code {
    font-size: 11px; font-weight: 600;
    color: var(--ink-faint); width: 60px;
    padding: 3px 8px; background: var(--surface-soft);
    border-radius: var(--r-pill); text-align: center;
  }
  .product-option .name { font-weight: 600; font-size: 14px; letter-spacing: -0.005em; }
  .product-option .brand-tag {
    font-size: 10px; color: var(--ink-faint);
    font-weight: 500; text-transform: uppercase;
    letter-spacing: 0.05em;
  }
  .product-option .from {
    font-size: 11px; color: var(--ink-soft); font-weight: 500;
  }
  .product-option .from strong {
    color: var(--ink); font-size: 15px;
    font-weight: 700; margin-left: 4px;
    letter-spacing: -0.01em;
  }
  .empty-msg { padding: 40px; text-align: center; color: var(--ink-faint); font-size: 13px; }

  /* Qty input in modal */
  .qty-input-wrap {
    display: flex; align-items: stretch;
    border: 1px solid var(--line);
    border-radius: var(--r-pill);
    overflow: hidden;
    max-width: 220px;
    background: var(--canvas-light);
  }
  .qty-input-wrap:focus-within {
    border-color: var(--brand);
    box-shadow: 0 0 0 4px var(--brand-soft);
  }
  .qty-input-wrap button {
    background: var(--surface-soft); border: none;
    width: 44px; cursor: pointer; font-size: 18px;
    color: var(--ink); font-weight: 600;
    transition: background 160ms;
  }
  .qty-input-wrap button:hover { background: var(--line); }
  .qty-input-wrap input {
    flex: 1; border: none; padding: 12px;
    font-family: 'Inter', sans-serif;
    font-size: 18px; text-align: center; font-weight: 700;
    background: var(--canvas-light);
    -moz-appearance: textfield;
    letter-spacing: -0.02em;
  }
  .qty-input-wrap input:focus { outline: none; }
  .qty-input-wrap input::-webkit-inner-spin-button,
  .qty-input-wrap input::-webkit-outer-spin-button {
    -webkit-appearance: none; margin: 0;
  }
  .qty-quick { display: flex; gap: 6px; margin-top: 10px; flex-wrap: wrap; }
  .qty-quick button {
    background: var(--canvas-light); border: 1px solid var(--line);
    padding: 6px 14px; border-radius: var(--r-pill);
    font-family: 'Inter', sans-serif;
    font-size: 12px; font-weight: 600;
    color: var(--ink-soft); cursor: pointer;
    transition: all 160ms;
  }
  .qty-quick button:hover {
    border-color: var(--brand); color: var(--brand);
    background: var(--brand-soft);
  }

  /* Add-on selection cards */
  .addon-list {
    display: grid; gap: 6px;
    grid-template-columns: 1fr 1fr;
  }
  @media (max-width: 600px) { .addon-list { grid-template-columns: 1fr; } }
  .addon-card-opt {
    display: flex; align-items: center; gap: 12px;
    padding: 12px 14px;
    border: 1px solid var(--line);
    border-radius: var(--r-chip);
    background: var(--canvas-light);
    cursor: pointer; transition: all 160ms;
  }
  .addon-card-opt:hover { border-color: var(--line-strong); }
  .addon-card-opt.selected {
    border-color: var(--brand);
    background: var(--brand-soft);
  }
  .addon-card-opt input { cursor: pointer; accent-color: var(--brand); flex-shrink: 0; }
  .addon-card-opt .label-text { flex: 1; font-size: 13px; }
  .addon-card-opt .label-text .name { font-weight: 600; letter-spacing: -0.005em; }
  .addon-card-opt .label-text .from {
    display: block; font-size: 10px; color: var(--ink-soft);
    margin-top: 2px; text-transform: uppercase; letter-spacing: 0.05em;
    font-weight: 500;
  }

  /* Toast */
  .toast {
    position: fixed; bottom: 32px; left: 50%;
    transform: translateX(-50%) translateY(20px);
    background: var(--canvas-dark); color: var(--on-dark);
    padding: 12px 24px; border-radius: var(--r-pill);
    font-size: 13px; font-weight: 600;
    box-shadow: var(--shadow-md); opacity: 0;
    transition: all 240ms cubic-bezier(0.16, 1, 0.3, 1);
    pointer-events: none; z-index: 1000;
    letter-spacing: -0.005em;
  }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

  footer {
    margin-top: 40px; text-align: center;
    font-size: 11px;
    color: var(--ink-faint); letter-spacing: 0.02em;
  }

  /* ============ MOBILE (iPhone & small screens) ============ */
  @media (max-width: 640px) {
    /* Tighter page padding */
    .page { padding: 16px 12px 60px; }

    /* Header: stack and compact */
    header {
      grid-template-columns: 1fr;
      gap: 16px;
      padding: 20px;
      border-radius: var(--r-chip);
      margin-bottom: 16px;
    }
    .brand { gap: 12px; }
    .brand .logo-img { width: 48px; height: 48px; }
    .brand h1 { font-size: 22px; line-height: 1.1; }
    .brand .eyebrow { font-size: 10px; }
    .meta {
      justify-content: flex-start;
      gap: 6px;
      font-size: 10px;
      flex-wrap: wrap;
    }
    .meta > span { padding: 5px 10px; font-size: 10px; }

    /* Layout: single column with smaller gap */
    .layout { gap: 12px; }
    .card { border-radius: var(--r-chip); }
    .card-header { padding: 14px 16px; }
    .card-title { font-size: 12px; }

    /* Quote item: tighter padding, rearranged head */
    .quote-item { padding: 16px 14px; }
    .quote-item-head {
      grid-template-columns: auto 1fr;       /* number + title only */
      grid-template-rows: auto auto;
      gap: 10px 12px;
    }
    .quote-item-head .num { width: 30px; height: 30px; font-size: 11px; }
    .quote-item-head .name { font-size: 16px; white-space: normal; }
    .quote-item-head .name .plus {
      display: inline-block; margin-top: 4px;
    }
    /* Action buttons move to a 2nd row, full width */
    .quote-item-head .actions {
      grid-column: 1 / -1;
      justify-self: end;
    }
    .quote-item-head .qty-line {
      grid-column: 1 / -1;
      flex-wrap: wrap;
      margin-top: 0;
    }
    .quote-item-head .qty-edit button { width: 38px; height: 38px; font-size: 18px; }
    .quote-item-head .qty-edit-input { width: 64px; font-size: 16px; }

    /* Breakdown: vertical stack — name on top, math below, right-aligned */
    .bd-row {
      grid-template-columns: 1fr;
      gap: 6px;
      padding: 10px 12px;
    }
    .bd-comp { flex-wrap: wrap; gap: 8px; }
    .bd-name { white-space: normal; font-size: 13px; }
    .bd-code { font-size: 10px; }
    .bd-calc {
      justify-content: flex-end;
      flex-wrap: wrap;
      font-size: 12px;
      gap: 6px;
    }
    .bd-sub { font-size: 14px; min-width: auto; }
    /* Touch-friendly steppers */
    .bd-qty-edit button { width: 32px; height: 32px; font-size: 15px; }
    .bd-qty-input { width: 56px; font-size: 16px; height: 32px; }
    button.bd-unit-editable { padding: 6px 10px; min-height: 32px; }
    .bd-reset, .bd-remove { width: 32px; height: 32px; }

    /* Foot: stack vertically */
    .quote-item-foot { grid-template-columns: 1fr; gap: 8px; }
    .foot-unit, .foot-total { padding: 12px 16px; }
    .foot-unit .val { font-size: 18px; }
    .foot-total .val { font-size: 22px; }

    /* Actions bar wraps */
    .actions-bar {
      padding: 12px 14px;
      flex-wrap: wrap;
      gap: 6px;
    }
    .actions-bar > div[style*="flex:1"] { display: none; }
    .actions-bar .btn-primary { order: -1; width: 100%; justify-content: center; }
    .btn { padding: 9px 16px; font-size: 13px; }
    .btn-lg { padding: 11px 20px; font-size: 14px; height: 42px; }

    /* Summary sidebar */
    .summary { position: static; border-radius: var(--r-chip); }
    .summary-row { padding: 14px 16px; }
    .summary-row .value { font-size: 18px; }
    .summary-row.subtotal .value { font-size: 14px; }
    .suggest-row .value { font-size: 24px; }
    .summary-row.profit-row .value { font-size: 18px; }

    /* Markup control: stack */
    .markup-control { padding: 14px 16px; }
    .markup-control .markup-row { flex-wrap: wrap; gap: 6px; }
    .preset-btn { padding: 8px 8px; font-size: 12px; }
    .markup-control input[type="number"] { width: 70px; font-size: 16px; }

    /* Platform pills */
    .platform-control { padding: 14px 16px; }
    .platform-pill { padding: 10px 4px; font-size: 11px; }
    .platform-pill .pp-icon { width: 22px; height: 22px; font-size: 12px; }
    .platform-pill .pp-rate { font-size: 9px; }
    .platform-final-row .value { font-size: 18px; }

    /* Modal — on mobile: full-width, near-fullscreen */
    .modal-backdrop {
      overflow-y: auto;
    }
    .modal {
      width: 100%;
      max-width: 100%;
      margin: 0;
      margin-top: 16px;
      min-height: calc(100vh - 16px);
      border-radius: var(--r-card) var(--r-card) 0 0;
    }
    .modal-header { padding: 18px 20px; }
    .modal-header h2 { font-size: 20px; }
    .modal-header .close { width: 36px; height: 36px; font-size: 22px; }
    .modal-body { padding: 18px 16px; }
    .modal-footer {
      padding: 14px 16px;
      flex-direction: column-reverse;
      align-items: stretch;
      gap: 10px;
    }
    .modal-footer .preview {
      display: flex; gap: 16px;
      justify-content: space-between;
    }
    .modal-footer .preview > div { flex-direction: column; align-items: flex-start; gap: 0; }
    .modal-footer .preview strong { margin-left: 0; }
    .modal-footer > div[style*="display:flex"] { width: 100%; }
    .modal-footer .btn { flex: 1; justify-content: center; }

    /* Step labels & search */
    .step { margin-bottom: 24px; }
    .step-label { font-size: 10px; margin-bottom: 10px; }
    .search-box { padding: 12px 16px; font-size: 15px; } /* >=16px prevents iOS zoom */
    .cat-pills { gap: 4px; }
    .cat-pill { font-size: 11px; padding: 6px 10px; }
    .product-grid { max-height: 240px; }
    .product-option { padding: 10px 12px; }
    .product-option .left { gap: 10px; }
    .product-option .code { width: 50px; font-size: 10px; }
    .product-option .name { font-size: 13px; }
    .product-option .from strong { font-size: 13px; }

    /* Qty input in modal — bigger touch targets */
    .qty-input-wrap { max-width: 100%; }
    .qty-input-wrap button { width: 48px; font-size: 20px; min-height: 48px; }
    .qty-input-wrap input { font-size: 18px; padding: 14px; min-height: 48px; }
    .qty-quick { gap: 4px; }
    .qty-quick button { padding: 8px 12px; font-size: 12px; min-height: 36px; }

    /* Add-on selection: single column with bigger tap targets */
    .addon-list { grid-template-columns: 1fr; gap: 6px; }
    .addon-card-opt { padding: 12px 14px; min-height: 50px; }
    .addon-card-opt input { width: 20px; height: 20px; }

    /* Upsell banners */
    .upsell-banner { padding: 12px 14px; flex-wrap: wrap; gap: 10px; }
    .upsell-msg { flex-basis: 100%; font-size: 12px; }
    .upsell-actions { flex-basis: 100%; }
    .upsell-btn { padding: 7px 12px; font-size: 11px; flex: 1; text-align: center; }

    /* Inline add-more product CTA */
    .quote-addmore { padding: 14px 16px; }
    .quote-addmore-btn { width: 100%; justify-content: center; padding: 12px 18px; }

    /* Inline addon price editor input */
    input.bd-unit-input { width: 90px; font-size: 16px; min-height: 36px; padding: 8px 12px; }

    /* Toast */
    .toast { bottom: 16px; left: 16px; right: 16px; transform: translateY(20px); }
    .toast.show { transform: translateY(0); }
  }

  /* Even smaller phones */
  @media (max-width: 380px) {
    .brand h1 { font-size: 19px; }
    .brand .logo-img { width: 40px; height: 40px; }
    .quote-item { padding: 14px 10px; }
  }

  @media print {
    body { background: white; }
    .actions-bar, .icon-btn, footer, .modal-backdrop { display: none !important; }
    .summary { position: static; }
    header { background: white; color: black; border: 1px solid var(--line); }
    header::before { display: none; }
    .brand h1 { color: black; }
    .foot-total, .suggest-row { background: white !important; color: black !important; border: 2px solid black; }
    .foot-total .lbl, .foot-total .val,
    .suggest-row .label, .suggest-row .value { color: black !important; }
  }

  /* Inter font - using system font fallbacks since we can't load fonts in this env */
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
</style>
</head>
<body>
<div class="page">

  <header>
    <div class="brand">
      <div class="logo-img">
        <img src="data:image/png;base64,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" alt="FOSB logo">
      </div>
      <div class="brand-text">
        <span class="eyebrow">Agent Calculator · v3.0</span>
        <h1><span class="wordmark">First One</span> Sales Builder</h1>
      </div>
    </div>
    <div class="meta">
      <span><span class="dot"></span>157 items · 13 add-ons</span>
      <span id="ts"></span>
    </div>
  </header>

  <div class="layout">
    <div>
      <div class="card">
        <div class="card-header">
          <span class="card-title">Quote · <strong id="quote-count">0 Items</strong></span>
          <span class="card-title" id="quote-meta"></span>
        </div>
        <div id="quote-list" class="quote-list">
          <div class="empty-state">
            <div class="big">No items yet</div>
            <div>Click "Add Product" below to build a quote</div>
          </div>
        </div>
        <div class="actions-bar">
          <button class="btn" id="clear-quote-btn"><span class="icon">×</span> Clear Quote</button>
          <button class="btn" id="print-btn"><span class="icon">⎙</span> Print</button>
          <button class="btn" id="export-btn"><span class="icon">↓</span> Export CSV</button>
          <div style="flex:1"></div>
          <button class="btn btn-primary btn-lg" id="add-product-btn" onclick="document.getElementById('modal').classList.add('show'); if(window.openAddProduct) window.openAddProduct();">
            <span class="icon">+</span> Add Product
          </button>
        </div>
      </div>
    </div>

    <aside class="summary">
      <div class="summary-row subtotal">
        <span class="label">Items Subtotal</span>
        <span class="value" id="items-subtotal">RM 0.00</span>
      </div>
      <div class="summary-row subtotal">
        <span class="label">Add-ons Subtotal</span>
        <span class="value" id="addons-subtotal">RM 0.00</span>
      </div>
      <div class="summary-row">
        <span class="label">Cost Total</span>
        <span class="value" id="total">RM 0.00</span>
      </div>

      <div class="markup-control">
        <label>Markup <span id="markup-display">30%</span></label>
        <div class="markup-row">
          <div class="preset-btns">
            <button class="preset-btn" data-val="20">20%</button>
            <button class="preset-btn active" data-val="30">30%</button>
            <button class="preset-btn" data-val="40">40%</button>
            <button class="preset-btn" data-val="50">50%</button>
          </div>
          <input type="number" id="markup-input" value="30" min="0" max="500" step="1" inputmode="decimal">
        </div>
      </div>

      <div class="summary-row profit-row">
        <span class="label">Profit</span>
        <span class="value" id="markup-amt">RM 0.00</span>
      </div>

      <div class="summary-row suggest-row">
        <span class="label">Suggested Price</span>
        <span class="value" id="suggest">RM 0.00</span>
      </div>

      <div class="platform-control">
        <div class="platform-header">
          <span class="ph-title">E-commerce Platform Fee</span>
          <span class="ph-hint">Gross up so you net the suggested price</span>
        </div>
        <div class="platform-pills">
          <button class="platform-pill active" data-platform="none" type="button">
            <span class="pp-icon">○</span> None
          </button>
          <button class="platform-pill" data-platform="shopee" type="button">
            <span class="pp-icon shopee">S</span> Shopee
            <span class="pp-rate">18%</span>
          </button>
          <button class="platform-pill" data-platform="lazada" type="button">
            <span class="pp-icon lazada">L</span> Lazada
            <span class="pp-rate">20%</span>
          </button>
        </div>
        <div class="platform-rate-edit" id="platform-rate-edit" style="display:none">
          <span class="rate-lbl"><span id="platform-rate-lbl">Shopee</span> Fee Rate</span>
          <input type="number" id="platform-rate" value="18" min="0" max="100" step="0.1" inputmode="decimal">
          <span class="rate-pct">%</span>
        </div>
      </div>

      <div class="summary-row platform-row" id="platform-fee-row" style="display:none">
        <span class="label"><span id="platform-fee-label">Platform Fee</span> (<span id="platform-rate-display">18%</span>)</span>
        <span class="value" id="platform-fee-amt">RM 0.00</span>
      </div>

      <div class="summary-row platform-final-row" id="platform-final-row" style="display:none">
        <span class="label">List Price on <span id="platform-final-label">Shopee</span></span>
        <span class="value" id="platform-list-price">RM 0.00</span>
      </div>

      <div class="tier-hint">
        Unit price drops as quantity rises.<br>
        Markup applies to combined total.
      </div>
    </aside>
  </div>

  <footer>Adapted from Agent_Calculator_-_Internal.xlsx · All Prices in RM</footer>
</div>

<!-- MODAL -->
<div class="modal-backdrop" id="modal">
  <div class="modal">
    <div class="modal-header">
      <h2 id="modal-title">Add Product</h2>
      <button class="close" id="modal-close" type="button">×</button>
    </div>
    <div class="modal-body">

      <div class="step">
        <div class="step-label"><span class="step-num">1</span> Choose a Product</div>
        <input type="text" class="search-box" id="search-input" placeholder="Search by Name or Code (e.g. 'snapback', 'CP01')">
        <div class="cat-pills" id="cat-pills"></div>
        <div class="product-grid" id="product-grid"></div>
      </div>

      <div class="step">
        <div class="step-label"><span class="step-num">2</span> Quantity</div>
        <div class="qty-input-wrap">
          <button type="button" id="qty-minus">−</button>
          <input type="number" id="qty-input" value="50" min="1" step="1" inputmode="decimal">
          <button type="button" id="qty-plus">+</button>
        </div>
        <div class="qty-quick">
          <button type="button" data-qty="12">12</button>
          <button type="button" data-qty="30">30</button>
          <button type="button" data-qty="50">50</button>
          <button type="button" data-qty="100">100</button>
          <button type="button" data-qty="200">200</button>
          <button type="button" data-qty="500">500</button>
        </div>
      </div>

      <div class="step">
        <div class="step-label"><span class="step-num">3</span> Add-ons (Optional)</div>
        <div class="addon-list" id="addon-list"></div>
      </div>

    </div>
    <div class="modal-footer">
      <div class="preview">
        <div>Unit price <strong id="modal-unit-price">RM 0.00</strong></div>
        <div>Line total <strong id="modal-line-total">RM 0.00</strong></div>
      </div>
      <div style="display:flex; gap:8px">
        <button class="btn" id="modal-cancel" type="button">Cancel</button>
        <button class="btn btn-primary" id="modal-confirm" type="button">Add to Quote</button>
      </div>
    </div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
(function() {
  'use strict';

  const PRODUCTS = {"AP03":{"item":"Apron","brand":"Oren","tiers":[],"category":"Aprons"},"AP04":{"item":"Apron","brand":"Oren","tiers":[{"qty":29,"price":10.5},{"qty":49,"price":10},{"qty":99,"price":9.5}],"category":"Aprons"},"CI10":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":24.7},{"qty":49,"price":24.2},{"qty":99,"price":23.7}],"category":"Polo Shirts"},"CI11":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":24.7},{"qty":49,"price":24.2},{"qty":99,"price":23.7}],"category":"Polo Shirts"},"CI13":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":24.7},{"qty":49,"price":24.2},{"qty":99,"price":23.7}],"category":"Polo Shirts"},"CJ01":{"item":"Jackets (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":76},{"qty":49,"price":75.5},{"qty":99,"price":75}],"category":"Jackets"},"EJ02":{"item":"Jackets (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":48.6},{"qty":49,"price":48.1},{"qty":99,"price":47.6}],"category":"Jackets"},"EJ03":{"item":"Bomber Jackets (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":58.9},{"qty":49,"price":58.4},{"qty":99,"price":57.9}],"category":"Jackets"},"CT51":{"item":"Round Neck T-Shirts (S/S, 160gsm)","brand":"Oren","tiers":[{"qty":29,"price":8.5},{"qty":49,"price":8},{"qty":99,"price":7.5}],"category":"T-Shirts"},"CT52 (K)":{"item":"Round Neck T-Shirts (S/S, 160gsm)","brand":"Oren","tiers":[{"qty":29,"price":8.7},{"qty":49,"price":8.2},{"qty":99,"price":7.7}],"category":"T-Shirts"},"CT54":{"item":"Round Neck T-Shirts (L/S, 160gsm)","brand":"Oren","tiers":[{"qty":29,"price":12.9},{"qty":49,"price":12.4},{"qty":99,"price":11.9}],"category":"T-Shirts"},"CT55":{"item":"Round Neck Raglan T-Shirts (S/S, 160gsm)","brand":"Oren","tiers":[{"qty":29,"price":11.4},{"qty":49,"price":10.9},{"qty":99,"price":10.4}],"category":"T-Shirts"},"CT56":{"item":"Round Neck Raglan T-Shirts (3/4 Sleeve, 160gsm)","brand":"Oren","tiers":[{"qty":29,"price":15.2},{"qty":49,"price":14.7},{"qty":99,"price":14.2}],"category":"T-Shirts"},"CT60":{"item":"Round Neck T-Shirts (S/S, 180gsm)","brand":"Oren","tiers":[{"qty":29,"price":11.4},{"qty":49,"price":10.9},{"qty":99,"price":10.4}],"category":"T-Shirts"},"CT71":{"item":"Round Neck T-Shirts (S/S, 200gsm)","brand":"Oren","tiers":[{"qty":29,"price":12.4},{"qty":49,"price":11.9},{"qty":99,"price":11.4}],"category":"T-Shirts"},"F116":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F117 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F118":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F119 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[],"category":"Shirts"},"F126":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F127 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F128":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F129 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F130":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F131 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F132":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F133 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F134":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F135 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F140":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F141 (F)":{"item":"Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F142":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F143 (F)":{"item":"Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F144":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F145 (F)":{"item":"Shirts (3/4 Sleeve)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"F146":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":41.9},{"qty":49,"price":41.4},{"qty":99,"price":40.9}],"category":"Shirts"},"F147 (F)":{"item":"Shirts (Muslimah, L/S)","brand":"Oren","tiers":[{"qty":29,"price":45.7},{"qty":49,"price":45.2},{"qty":99,"price":44.7}],"category":"Muslimah Wear"},"F148":{"item":"Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":38.1},{"qty":49,"price":37.6},{"qty":99,"price":37.1}],"category":"Shirts"},"F149 (F)":{"item":"Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":39.1},{"qty":49,"price":38.6},{"qty":99,"price":38.1}],"category":"Shirts"},"HC01":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":15.8},{"qty":49,"price":15.3},{"qty":99,"price":14.8}],"category":"Polo Shirts"},"HC05 (F)":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":15.2},{"qty":49,"price":14.7},{"qty":99,"price":14.2}],"category":"Polo Shirts"},"HC09":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":19},{"qty":49,"price":18.5},{"qty":99,"price":18}],"category":"Polo Shirts"},"HC10":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"HC11":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"HC12":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":15.5},{"qty":49,"price":15},{"qty":99,"price":14.5}],"category":"Polo Shirts"},"HC15":{"item":"Polo Shirts (S/S with Hidden Pocket)","brand":"Oren","tiers":[{"qty":29,"price":19},{"qty":49,"price":18.5},{"qty":99,"price":18}],"category":"Polo Shirts"},"HC16":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.7},{"qty":49,"price":16.2},{"qty":99,"price":15.7}],"category":"Polo Shirts"},"HC20":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"HC21":{"item":"Stand Collar Shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Shirts"},"HC22":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"HC23":{"item":"Stand Collar Shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Shirts"},"HC24":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.7},{"qty":49,"price":17.2},{"qty":99,"price":16.7}],"category":"Polo Shirts"},"HC25":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"HC26":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"HC27":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":20},{"qty":49,"price":19.5},{"qty":99,"price":19}],"category":"Polo Shirts"},"HC28":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":20},{"qty":49,"price":19.5},{"qty":99,"price":19}],"category":"Polo Shirts"},"HC29":{"item":"Polo shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.7},{"qty":49,"price":17.2},{"qty":99,"price":16.7}],"category":"Polo Shirts"},"HZ01":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":21.9},{"qty":49,"price":21.4},{"qty":99,"price":20.9}],"category":"Polo Shirts"},"JK02":{"item":"Jackets (L/S with Reflective Tape)","brand":"Oren","tiers":[{"qty":29,"price":44.8},{"qty":49,"price":44.3},{"qty":99,"price":43.8}],"category":"Jackets"},"LP01":{"item":"Pants","brand":"Oren","tiers":[{"qty":29,"price":44.8},{"qty":49,"price":44.3},{"qty":99,"price":43.8}],"category":"Pants"},"OT01":{"item":"Oversized Tee (S/S)","brand":"Oren","tiers":[{"qty":29,"price":20.9},{"qty":49,"price":20.4},{"qty":99,"price":19.9}],"category":"T-Shirts"},"OV02":{"item":"Coverall (with Reflective Tape)","brand":"Oren","tiers":[{"qty":29,"price":64.7},{"qty":49,"price":64.2},{"qty":99,"price":63.7}],"category":"Jackets"},"QD04":{"item":"Round Neck T-Shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":7.6},{"qty":49,"price":7.1},{"qty":99,"price":6.6}],"category":"T-Shirts"},"QD06":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":12.4},{"qty":49,"price":11.9},{"qty":99,"price":11.4}],"category":"Polo Shirts"},"QD15 (F)":{"item":"Round Neck T-Shirt (S/S)","brand":"Oren","tiers":[{"qty":29,"price":9.1},{"qty":49,"price":8.6},{"qty":99,"price":8.1}],"category":"T-Shirts"},"QD16 (F)":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":12.4},{"qty":49,"price":11.9},{"qty":99,"price":11.4}],"category":"Polo Shirts"},"QD18":{"item":"Zip Neck Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.3},{"qty":49,"price":15.8},{"qty":99,"price":15.3}],"category":"Shirts"},"QD21(F)":{"item":"Zip Neck Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.3},{"qty":49,"price":15.8},{"qty":99,"price":15.3}],"category":"Shirts"},"QD25":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":20.8},{"qty":49,"price":20.3},{"qty":99,"price":19.8}],"category":"Polo Shirts"},"QD30":{"item":"Zip Neck Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Shirts"},"QD31":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":20.8},{"qty":49,"price":20.3},{"qty":99,"price":19.8}],"category":"Polo Shirts"},"QD33":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":20.8},{"qty":49,"price":20.3},{"qty":99,"price":19.8}],"category":"Polo Shirts"},"QD34":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":20.8},{"qty":49,"price":20.3},{"qty":99,"price":19.8}],"category":"Polo Shirts"},"QD36":{"item":"Round Neck Raglan T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":12.4},{"qty":49,"price":11.9},{"qty":99,"price":11.4}],"category":"T-Shirts"},"QD37":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":19.6},{"qty":49,"price":19.1},{"qty":99,"price":18.6}],"category":"Polo Shirts"},"QD43":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":14.8},{"qty":49,"price":14.3},{"qty":99,"price":13.8}],"category":"T-Shirts"},"QD46":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":13.3},{"qty":49,"price":12.8},{"qty":99,"price":12.3}],"category":"T-Shirts"},"QD47":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":16.3},{"qty":49,"price":15.8},{"qty":99,"price":15.3}],"category":"Polo Shirts"},"QD48":{"item":"Round Neck Raglan T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":13.3},{"qty":49,"price":12.8},{"qty":99,"price":12.3}],"category":"T-Shirts"},"QD49":{"item":"Round Neck Raglan T-Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":14.3},{"qty":49,"price":13.8},{"qty":99,"price":13.3}],"category":"T-Shirts"},"QD50":{"item":"Round Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":10.5},{"qty":49,"price":10},{"qty":99,"price":9.5}],"category":"T-Shirts"},"QD51":{"item":"Round Neck T-Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":11.4},{"qty":49,"price":10.9},{"qty":99,"price":10.4}],"category":"T-Shirts"},"QD52":{"item":"V-Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":10.5},{"qty":49,"price":10},{"qty":99,"price":9.5}],"category":"T-Shirts"},"QD53":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"QD54":{"item":"Round Neck T-Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":12.4},{"qty":49,"price":11.9},{"qty":99,"price":11.4}],"category":"T-Shirts"},"QD55":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":11.4},{"qty":49,"price":10.9},{"qty":99,"price":10.4}],"category":"T-Shirts"},"QD56":{"item":"Round Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":9.1},{"qty":49,"price":8.6},{"qty":99,"price":8.1}],"category":"T-Shirts"},"QD57":{"item":"Round Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":12.9},{"qty":49,"price":12.4},{"qty":99,"price":11.9}],"category":"T-Shirts"},"QD58":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":15.8},{"qty":49,"price":15.3},{"qty":99,"price":14.8}],"category":"Polo Shirts"},"QD59":{"item":"Round Neck T-Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":14.8},{"qty":49,"price":14.3},{"qty":99,"price":13.8}],"category":"T-Shirts"},"QD60":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":15.8},{"qty":49,"price":15.3},{"qty":99,"price":14.8}],"category":"Polo Shirts"},"QD61":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":15.8},{"qty":49,"price":15.3},{"qty":99,"price":14.8}],"category":"Polo Shirts"},"QD62":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":13.3},{"qty":49,"price":12.8},{"qty":99,"price":12.3}],"category":"T-Shirts"},"QD63":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":16.3},{"qty":49,"price":15.8},{"qty":99,"price":15.3}],"category":"Polo Shirts"},"QD64":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"QD65":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.3},{"qty":49,"price":15.8},{"qty":99,"price":15.3}],"category":"Polo Shirts"},"QD66":{"item":"Round Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":8.7},{"qty":49,"price":8.2},{"qty":99,"price":7.7}],"category":"T-Shirts"},"QD67":{"item":"Polo Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":16.2},{"qty":49,"price":15.7},{"qty":99,"price":15.2}],"category":"Polo Shirts"},"QD68 (K)":{"item":"Round Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":9.1},{"qty":49,"price":8.6},{"qty":99,"price":8.1}],"category":"T-Shirts"},"QD69":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":19},{"qty":49,"price":18.5},{"qty":99,"price":18}],"category":"Polo Shirts"},"QD70":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":14.8},{"qty":49,"price":14.3},{"qty":99,"price":13.8}],"category":"T-Shirts"},"QD71":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"QD72":{"item":"Round Neck T-Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":8.7},{"qty":49,"price":8.2},{"qty":99,"price":7.7}],"category":"T-Shirts"},"QD73":{"item":"Round Neck Tank Top (Sleeveless)","brand":"Oren","tiers":[{"qty":29,"price":8.7},{"qty":49,"price":8.2},{"qty":99,"price":7.7}],"category":"Vests"},"QD74":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.2},{"qty":49,"price":15.7},{"qty":99,"price":15.2}],"category":"Polo Shirts"},"QD75":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":14.8},{"qty":49,"price":14.3},{"qty":99,"price":13.8}],"category":"T-Shirts"},"QD76":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"QD77":{"item":"Round Neck T-Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":14.8},{"qty":49,"price":14.3},{"qty":99,"price":13.8}],"category":"T-Shirts"},"QD78":{"item":"Polo Shirts (S/S, Sublimation)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"QD79":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":18.6},{"qty":49,"price":18.1},{"qty":99,"price":17.6}],"category":"Polo Shirts"},"QD80":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.3},{"qty":49,"price":15.8},{"qty":99,"price":15.3}],"category":"Polo Shirts"},"SJ01":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":16.2},{"qty":49,"price":15.7},{"qty":99,"price":15.2}],"category":"Polo Shirts"},"SJ03":{"item":"Polo Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":18.1},{"qty":49,"price":17.6},{"qty":99,"price":17.1}],"category":"Polo Shirts"},"SJ04":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"SJ05":{"item":"Polo Shirts (L/S)","brand":"Oren","tiers":[{"qty":29,"price":19},{"qty":49,"price":18.5},{"qty":99,"price":18}],"category":"Polo Shirts"},"SJ08":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"SJ09":{"item":"Polo Shirts (S/S)","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":49,"price":16.6},{"qty":99,"price":16.1}],"category":"Polo Shirts"},"SK01":{"item":"Muslimah (L/S)","brand":"Oren","tiers":[{"qty":29,"price":20.9},{"qty":49,"price":20.4},{"qty":99,"price":19.9}],"category":"Muslimah Wear"},"SK02":{"item":"Muslimah (L/S)","brand":"Oren","tiers":[{"qty":29,"price":20.9},{"qty":49,"price":20.4},{"qty":99,"price":19.9}],"category":"Muslimah Wear"},"SK03":{"item":"Muslimah (L/S)","brand":"Oren","tiers":[{"qty":29,"price":18.1},{"qty":49,"price":17.6},{"qty":99,"price":17.1}],"category":"Muslimah Wear"},"SK04":{"item":"Muslimah (L/S)","brand":"Oren","tiers":[{"qty":29,"price":20.9},{"qty":49,"price":20.4},{"qty":99,"price":19.9}],"category":"Muslimah Wear"},"SK05":{"item":"Muslimah (L/S) , Quick Dry","brand":"Oren","tiers":[{"qty":29,"price":15.2},{"qty":49,"price":14.7},{"qty":99,"price":14.2}],"category":"Muslimah Wear"},"TT01":{"item":"Premium Polo Shirt (S/S)","brand":"Oren","tiers":[],"category":"Polo Shirts"},"TT02":{"item":"Premium Polo Shirt (S/S)","brand":"Oren","tiers":[],"category":"Polo Shirts"},"TW01":{"item":"Towelling (Hand Towel)","brand":"Oren","tiers":[{"qty":29,"price":5.3},{"qty":49,"price":4.8},{"qty":99,"price":4.3}],"category":"Towels"},"TW03":{"item":"Towelling (Bath Towel) [0300/0304/0307/0310/0314/0325]","brand":"Oren","tiers":[{"qty":29,"price":17.1},{"qty":29,"price":20},{"qty":49,"price":16.6},{"qty":49,"price":19.5},{"qty":99,"price":16.1},{"qty":99,"price":19}],"category":"Towels"},"TW06":{"item":"Towelling (Bath Towel) [0600/0604/0607/0610/0614/0625]","brand":"Oren","tiers":[{"qty":29,"price":21},{"qty":29,"price":23.8},{"qty":49,"price":20.5},{"qty":49,"price":23.3},{"qty":99,"price":20},{"qty":99,"price":22.8}],"category":"Towels"},"TW07":{"item":"Towelling (Hand Towel)","brand":"Oren","tiers":[{"qty":29,"price":8.6},{"qty":49,"price":8.1},{"qty":99,"price":7.6}],"category":"Towels"},"TW08":{"item":"Towelling (Bath Towel)","brand":"Oren","tiers":[{"qty":29,"price":11.4},{"qty":49,"price":10.9},{"qty":99,"price":10.4}],"category":"Towels"},"TW09":{"item":"Towelling (Hand Towel)","brand":"Oren","tiers":[{"qty":29,"price":4.2},{"qty":49,"price":3.7},{"qty":99,"price":3.2}],"category":"Towels"},"TW10":{"item":"Towelling (Sport Hand Towel)","brand":"Oren","tiers":[{"qty":29,"price":10.1},{"qty":49,"price":9.6},{"qty":99,"price":9.1}],"category":"Towels"},"VJ06":{"item":"Vest Jacket (Sleeveless)","brand":"Oren","tiers":[{"qty":29,"price":25.8},{"qty":49,"price":25.3},{"qty":99,"price":24.8}],"category":"Vests"},"VT0203":{"item":"Unisex Sleeveless Safety Vest (with reflective tape)","brand":"Oren","tiers":[{"qty":29,"price":9.6},{"qty":49,"price":9.1},{"qty":99,"price":8.6}],"category":"Vests"},"VT03":{"item":"Unisex Sleeveless Utility Vest","brand":"Oren","tiers":[{"qty":29,"price":27.7},{"qty":49,"price":27.2},{"qty":99,"price":26.7}],"category":"Vests"},"WB04":{"item":"Windbreakers (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":40},{"qty":49,"price":39.5},{"qty":99,"price":39}],"category":"Windbreakers"},"WB05":{"item":"Windbreakers (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":40},{"qty":49,"price":39.5},{"qty":99,"price":39}],"category":"Windbreakers"},"WB06":{"item":"Windbreakers (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":33.4},{"qty":49,"price":32.9},{"qty":99,"price":32.4}],"category":"Windbreakers"},"WR01":{"item":"Reversible Windbreakers (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":50},{"qty":49,"price":49.5},{"qty":99,"price":49}],"category":"Windbreakers"},"WR03":{"item":"Reversible Windbreakers (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":52.4},{"qty":49,"price":51.9},{"qty":99,"price":51.4}],"category":"Windbreakers"},"WR04":{"item":"Reversible Windbreakers (L/S Full Zip)","brand":"Oren","tiers":[{"qty":29,"price":52.4},{"qty":49,"price":51.9},{"qty":99,"price":51.4}],"category":"Windbreakers"},"CP01":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":7.6},{"qty":49,"price":7.1},{"qty":99,"price":6.6}],"category":"Caps"},"CP03":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":7.6},{"qty":49,"price":7.1},{"qty":99,"price":6.6}],"category":"Caps"},"CP04":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":7.6},{"qty":49,"price":7.1},{"qty":99,"price":6.6}],"category":"Caps"},"CP05":{"item":"Baseball Caps (Polyster)","brand":"Oren","tiers":[{"qty":29,"price":6.7},{"qty":49,"price":6.2},{"qty":99,"price":5.7}],"category":"Caps"},"CP09":{"item":"Baseball Caps (Polyster)","brand":"Oren","tiers":[{"qty":29,"price":8},{"qty":49,"price":7.5},{"qty":99,"price":7}],"category":"Caps"},"CP12":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":8.6},{"qty":49,"price":8.1},{"qty":99,"price":7.6}],"category":"Caps"},"CP13":{"item":"Baseball Caps (Polyster)","brand":"Oren","tiers":[{"qty":29,"price":8},{"qty":49,"price":7.5},{"qty":99,"price":7}],"category":"Caps"},"CP14":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":8.6},{"qty":49,"price":8.1},{"qty":99,"price":7.6}],"category":"Caps"},"CP17":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":8.6},{"qty":49,"price":8.1},{"qty":99,"price":7.6}],"category":"Caps"},"CP18":{"item":"Baseball Caps (Polyster)","brand":"Oren","tiers":[{"qty":29,"price":8},{"qty":49,"price":7.5},{"qty":99,"price":7}],"category":"Caps"},"CP19":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":8.6},{"qty":49,"price":8.1},{"qty":99,"price":7.6}],"category":"Caps"},"CP21":{"item":"Military Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":8},{"qty":49,"price":7.5},{"qty":99,"price":7}],"category":"Caps"},"CP23":{"item":"Snapback Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":7.6},{"qty":49,"price":7.1},{"qty":99,"price":6.6}],"category":"Caps"},"CP24 (K)":{"item":"Baseball Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":7.2},{"qty":49,"price":6.7},{"qty":99,"price":6.2}],"category":"Caps"},"CP25":{"item":"Snapback Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":7.6},{"qty":49,"price":7.1},{"qty":99,"price":6.6}],"category":"Caps"},"CP26":{"item":"Truckers Caps (Cotton)","brand":"Oren","tiers":[{"qty":29,"price":6.7},{"qty":49,"price":6.2},{"qty":99,"price":5.7}],"category":"Caps"},"FH01":{"item":"Fisherman Hat (Cotton) [0101/0103/0102]","brand":"Oren","tiers":[{"qty":29,"price":8.6},{"qty":29,"price":9.5},{"qty":49,"price":8.1},{"qty":49,"price":9},{"qty":99,"price":7.6},{"qty":99,"price":8.5}],"category":"Hats"}};
  const ADDONS = {"Emb1":{"item":"Embroidery logo (1000 stitches - 8000 stitches)","brand":"Own","tiers":[{"qty":9,"price":6},{"qty":99,"price":3},{"qty":199,"price":2.5}]},"Emb2":{"item":"Embroidery logo (8000 stitches above)","brand":"Own","tiers":[{"qty":9,"price":7},{"qty":99,"price":3.8},{"qty":199,"price":3}]},"Emb3":{"item":"Embroidery wording (1000 stitches - 8000 stitches)","brand":"Own","tiers":[{"qty":9,"price":3.5},{"qty":99,"price":3},{"qty":199,"price":2.6}]},"Emb4":{"item":"Embroidery wording (8000 stitches above)","brand":"Own","tiers":[{"qty":9,"price":5},{"qty":99,"price":4.2},{"qty":199,"price":3.5}]},"Silk1":{"item":"Silkscreen 1 colour","brand":"Own","tiers":[{"qty":29,"price":4.2},{"qty":49,"price":3.5},{"qty":99,"price":2}]},"Silk2":{"item":"Silkscreen 2 colour","brand":"Own","tiers":[{"qty":29,"price":8.5},{"qty":49,"price":5},{"qty":99,"price":3.2}]},"Silk3":{"item":"Silkscreen 3 colour","brand":"Own","tiers":[{"qty":29,"price":12.5},{"qty":49,"price":7.5},{"qty":99,"price":4.5}]},"Silk4":{"item":"Silkscreen 4 colour","brand":"Own","tiers":[{"qty":29,"price":16.5},{"qty":49,"price":10.5},{"qty":99,"price":5.5}]},"HD1":{"item":"HD print (2cm-10cm)","brand":"Own","tiers":[{"qty":5,"price":8},{"qty":10,"price":5},{"qty":20,"price":4.5},{"qty":30,"price":4},{"qty":50,"price":3.5},{"qty":100,"price":2.8},{"qty":200,"price":2},{"qty":300,"price":1.8},{"qty":500,"price":1.4}]},"HD2":{"item":"HD print (11cm-13cm)","brand":"Own","tiers":[{"qty":5,"price":10},{"qty":10,"price":7},{"qty":20,"price":6},{"qty":30,"price":5},{"qty":50,"price":4.2},{"qty":100,"price":3.5},{"qty":200,"price":2.6},{"qty":300,"price":2.4},{"qty":500,"price":2.2},{"qty":1000,"price":1.8}]},"HD3":{"item":"HD print (19cm-23cm)","brand":"Own","tiers":[{"qty":5,"price":12},{"qty":10,"price":10},{"qty":20,"price":8},{"qty":30,"price":6.5},{"qty":50,"price":5.8},{"qty":100,"price":4.8},{"qty":200,"price":4},{"qty":300,"price":3.6},{"qty":500,"price":3.2},{"qty":1000,"price":2.9}]},"HDA4":{"item":"A4 Size","brand":"Own","tiers":[{"qty":1,"price":35},{"qty":2,"price":22},{"qty":3,"price":18},{"qty":4,"price":15},{"qty":5,"price":14},{"qty":6,"price":13},{"qty":7,"price":12},{"qty":8,"price":11},{"qty":9,"price":10}]},"HDA3":{"item":"A3 Size","brand":"Own","tiers":[{"qty":9,"price":16.5},{"qty":19,"price":14},{"qty":29,"price":11.5}]}};

  // ---- State ----
  let quote = []; // [{id, productCode, qty, addonCodes: [..]}]
  let markup = 30;
  // Platform: 'none' | 'shopee' | 'lazada'
  let platform = 'none';
  const PLATFORM_DEFAULTS = { shopee: 18, lazada: 20 };
  let platformRates = { shopee: 18, lazada: 20 };
  let nextId = 1;

  // Modal state
  let editingId = null;
  let modalSelected = { productCode: null, qty: 50, addonCodes: [] };
  let modalCategory = 'All';
  let modalSearch = '';

  const $ = (id) => document.getElementById(id);
  const fmt = (n) => 'RM ' + (n || 0).toLocaleString('en-MY', { minimumFractionDigits: 2, maximumFractionDigits: 2 });

  // ---- Price lookup ----
  function lookupPrice(catalog, code, qty) {
    const p = catalog[code];
    if (!p) return null;
    if (!p.tiers || p.tiers.length === 0) return { nostock: true, product: p };
    if (!qty || qty <= 0) return { product: p, price: 0 };
    const tier = p.tiers.find(t => t.qty >= qty);
    const usedTier = tier || p.tiers[p.tiers.length - 1];
    return { product: p, price: usedTier.price };
  }

  function calcLine(line) {
    const prod = lookupPrice(PRODUCTS, line.productCode, line.qty);
    if (!prod || prod.nostock) return { itemTotal: 0, addonTotal: 0, lineTotal: 0, prod, addonDetails: [] };

    // Apply override if present
    const overrides = line.overrides || {};
    let itemPrice = prod.price;
    let itemOverridden = false;
    if (overrides.item != null) {
      itemPrice = overrides.item;
      itemOverridden = true;
    }
    const itemTotal = itemPrice * line.qty;

    let addonTotal = 0;
    const addonDetails = [];
    const counts = line.addonCounts || {};
    line.addonCodes.forEach(code => {
      const a = lookupPrice(ADDONS, code, line.qty);
      if (a && !a.nostock) {
        const count = Math.max(1, counts[code] || 1);
        let aPrice = a.price;
        let overridden = false;
        if (overrides.addons && overrides.addons[code] != null) {
          aPrice = overrides.addons[code];
          overridden = true;
        }
        const t = aPrice * line.qty * count;
        addonTotal += t;
        addonDetails.push({
          code, product: a.product,
          price: aPrice, originalPrice: a.price,
          count, total: t, overridden
        });
      }
    });
    return {
      itemTotal, addonTotal,
      lineTotal: itemTotal + addonTotal,
      prod: { ...prod, price: itemPrice, originalPrice: prod.price, overridden: itemOverridden },
      addonDetails
    };
  }

  // ---- Component-level upsell logic ----
  // Returns next-tier info for a single component, or null if no upgrade available
  function getComponentUpsell(tiers, qty) {
    if (!tiers || tiers.length === 0 || !qty || qty <= 0) return null;
    const curIdx = tiers.findIndex(t => t.qty >= qty);
    if (curIdx === -1) return null;       // already beyond all tiers
    if (curIdx + 1 >= tiers.length) return null;  // already in last tier
    const curTier = tiers[curIdx];
    const nextTier = tiers[curIdx + 1];
    if (nextTier.price >= curTier.price) return null; // not actually cheaper
    return {
      currentPrice: curTier.price,
      nextPrice: nextTier.price,
      bumpQty: curTier.qty + 1,        // qty needed to reach next tier naturally
      addPcs: (curTier.qty + 1) - qty,
      savings: curTier.price - nextTier.price,  // savings per piece if override applied
    };
  }

  // Returns array of component upsell suggestions for the line
  // Each item: { kind: 'item'|'addon', code, name, currentPrice, nextPrice, bumpQty, addPcs, savings, savingsTotal, bumpDiff }
  function findComponentUpsells(line) {
    if (!line.productCode || line.qty <= 0) return [];
    const overrides = line.overrides || {};
    const results = [];

    // Base product
    const prod = PRODUCTS[line.productCode];
    if (prod && prod.tiers && prod.tiers.length && overrides.item == null) {
      const u = getComponentUpsell(prod.tiers, line.qty);
      if (u) {
        results.push({
          kind: 'item',
          code: line.productCode,
          name: prod.item,
          ...u,
          savingsTotal: u.savings * line.qty,       // if override applied at current qty
          bumpDiff: (u.nextPrice * u.bumpQty) - (u.currentPrice * line.qty), // cost diff of bumping
        });
      }
    }

    // Each add-on
    const addonCounts = line.addonCounts || {};
    line.addonCodes.forEach(code => {
      if (overrides.addons && overrides.addons[code] != null) return;  // already overridden
      const a = ADDONS[code];
      if (!a || !a.tiers || a.tiers.length === 0) return;
      const u = getComponentUpsell(a.tiers, line.qty);
      if (u) {
        const count = Math.max(1, addonCounts[code] || 1);
        results.push({
          kind: 'addon',
          code,
          name: a.item,
          ...u,
          // Multiply by count: a savings of RM 0.50/pc with 2 logos = RM 1.00/pc total savings
          savingsTotal: u.savings * line.qty * count,
          bumpDiff: ((u.nextPrice * u.bumpQty) - (u.currentPrice * line.qty)) * count,
        });
      }
    });

    return results;
  }

  // ---- Render quote list ----
  function renderQuote() {
    const list = $('quote-list');
    if (quote.length === 0) {
      list.innerHTML = '<div class="empty-state"><div class="big">No items yet</div><div>Click "Add Product" below to build a quote</div></div>';
    } else {
      list.innerHTML = '';
      quote.forEach((line, i) => {
        const calc = calcLine(line);
        const prod = PRODUCTS[line.productCode];
        if (!prod) return;
        const unitPrice = line.qty > 0 ? calc.lineTotal / line.qty : 0;

        // Build breakdown rows: base product + each add-on
        let breakdownRows = '';
        const itemOverrideBadge = calc.prod.overridden
          ? ' <span class="override-badge" title="Was RM ' + (calc.prod.originalPrice || 0).toFixed(2) + '">discounted</span>'
          : '';
        const itemResetBtn = calc.prod.overridden
          ? '<button class="bd-reset" data-action="reset-override" data-kind="item" title="Reset to Standard Price">↺</button>'
          : '';
        breakdownRows +=
          '<div class="bd-row' + (calc.prod.overridden ? ' overridden' : '') + '">' +
            '<div class="bd-comp">' +
              '<span class="bd-tag base">item</span>' +
              '<span class="bd-name">' + escapeHtml(prod.item) + itemOverrideBadge + '</span>' +
              '<span class="bd-code">' + escapeHtml(line.productCode) + '</span>' +
            '</div>' +
            '<div class="bd-calc">' +
              '<span class="bd-unit">RM ' + (calc.prod.price || 0).toFixed(2) + '</span>' +
              '<span class="bd-x">×</span>' +
              '<div class="bd-qty-edit" title="Quantity">' +
                '<button type="button" data-action="dec-qty" title="Decrease Quantity">−</button>' +
                '<input type="number" class="bd-qty-input" data-action="set-qty" value="' + line.qty + '" min="1" step="1" inputmode="decimal">' +
                '<button type="button" data-action="inc-qty" title="Increase Quantity">+</button>' +
              '</div>' +
              '<span class="bd-eq">=</span>' +
              '<span class="bd-sub">RM ' + calc.itemTotal.toFixed(2) + '</span>' +
              itemResetBtn +
            '</div>' +
          '</div>';

        calc.addonDetails.forEach(a => {
          const ovBadge = a.overridden
            ? ' <span class="override-badge" title="Was RM ' + a.originalPrice.toFixed(2) + '">discounted</span>'
            : '';
          const resetBtn = a.overridden
            ? '<button class="bd-reset" data-action="reset-override" data-kind="addon" data-code="' + escapeHtml(a.code) + '" title="Reset to Standard Price">↺</button>'
            : '';
          // Count editor (typeable, same style as item qty)
          const countDisplay = a.count > 1
            ? '<span class="bd-count-badge">×' + a.count + '</span>'
            : '';
          const countStepper =
            '<div class="bd-qty-edit" title="How Many of This Add-on per Piece">' +
              '<button data-action="dec-addon-count" data-code="' + escapeHtml(a.code) + '" type="button">−</button>' +
              '<input type="number" class="bd-qty-input" data-action="set-addon-count" data-code="' + escapeHtml(a.code) + '" value="' + a.count + '" min="1" step="1" inputmode="decimal">' +
              '<button data-action="inc-addon-count" data-code="' + escapeHtml(a.code) + '" type="button">+</button>' +
            '</div>';
          breakdownRows +=
            '<div class="bd-row addon-row' + (a.overridden ? ' overridden' : '') + '">' +
              '<div class="bd-comp">' +
                '<span class="bd-tag addon">add-on</span>' +
                '<span class="bd-name">' + escapeHtml(a.product.item) + countDisplay + ovBadge + '</span>' +
                '<span class="bd-code">' + escapeHtml(a.code) + '</span>' +
              '</div>' +
              '<div class="bd-calc">' +
                '<button class="bd-unit bd-unit-editable" data-action="edit-addon-price" data-code="' + escapeHtml(a.code) + '" data-price="' + a.price + '" type="button" title="Click to Adjust Price (Give a Discount)">RM ' + a.price.toFixed(2) + '</button>' +
                '<span class="bd-x">×</span>' +
                '<span class="bd-qty">' + line.qty + '</span>' +
                '<span class="bd-x">×</span>' +
                countStepper +
                '<span class="bd-eq">=</span>' +
                '<span class="bd-sub">RM ' + a.total.toFixed(2) + '</span>' +
                resetBtn +
                '<button class="bd-remove" data-action="remove-addon" data-addon="' + escapeHtml(a.code) + '" title="Remove This Add-on">×</button>' +
              '</div>' +
            '</div>';
        });

        // "+ Add add-on" button at end of breakdown, only if some add-ons aren't already used
        const availableAddons = Object.keys(ADDONS).filter(c => !line.addonCodes.includes(c));
        if (availableAddons.length > 0) {
          breakdownRows +=
            '<div class="bd-row bd-addmore-row">' +
              '<button class="bd-addmore" data-action="add-addon" type="button">+ Add Another Add-on</button>' +
            '</div>';
        }

        // Upsell suggestions — one per component that has a next-tier available
        const upsells = findComponentUpsells(line);
        const dismissedSet = new Set(line.dismissedUpsells || []);
        let upsellHtml = '';
        upsells.forEach(u => {
          const key = u.kind + ':' + u.code;
          if (dismissedSet.has(key)) return;

          const isSavings = u.bumpDiff <= 0;
          const klass = isSavings ? 'upsell-banner savings' : 'upsell-banner tip';
          const icon = isSavings ? '✨' : '💡';

          const componentLabel = u.kind === 'item'
            ? 'this item'
            : escapeHtml(u.name);

          // Always show two action buttons:
          //  1) Bump qty (only when reasonable: <=15 extra or actual savings)
          //  2) Use next-tier price at current qty (always available)
          const showBump = isSavings || (u.addPcs <= 15 && u.bumpDiff > 0);

          let msg;
          if (isSavings) {
            msg = '<strong>' + componentLabel + '</strong> reaches a cheaper tier at <strong>' + u.bumpQty + ' pcs</strong> (RM ' + u.nextPrice.toFixed(2) + '/pc, +' + u.addPcs + ' pcs)';
          } else {
            msg = '<strong>' + componentLabel + '</strong> next tier kicks in at <strong>' + u.bumpQty + ' pcs</strong> (RM ' + u.nextPrice.toFixed(2) + '/pc, +' + u.addPcs + ' pcs)';
          }

          upsellHtml +=
            '<div class="' + klass + '">' +
              '<span class="upsell-icon">' + icon + '</span>' +
              '<span class="upsell-msg">' + msg + '</span>' +
              '<div class="upsell-actions">' +
                (showBump ? '<button class="upsell-btn" data-action="apply-upsell" data-newqty="' + u.bumpQty + '" type="button">Bump To ' + u.bumpQty + '</button>' : '') +
                '<button class="upsell-btn alt" data-action="override-price" data-kind="' + u.kind + '" data-code="' + escapeHtml(u.code) + '" data-newprice="' + u.nextPrice + '" type="button">Use RM ' + u.nextPrice.toFixed(2) + ' at ' + line.qty + ' Pcs</button>' +
              '</div>' +
              '<button class="upsell-dismiss" data-action="dismiss-upsell" data-key="' + escapeHtml(key) + '" type="button" title="Dismiss">×</button>' +
            '</div>';
        });

        const row = document.createElement('div');
        row.className = 'quote-item';
        row.dataset.id = line.id;
        row.innerHTML =
          '<div class="quote-item-head">' +
            '<div class="num">' + String(i+1).padStart(2,'0') + '</div>' +
            '<div class="head-title">' +
              '<div class="name">' + escapeHtml(prod.item) + (calc.addonDetails.length ? ' <span class="plus">+ ' + calc.addonDetails.length + ' add-on' + (calc.addonDetails.length === 1 ? '' : 's') + '</span>' : '') + '</div>' +
              '<div class="qty-line">' +
                '<span class="qty-label">Quantity:</span>' +
                '<div class="qty-edit">' +
                  '<button type="button" data-action="dec-qty" title="Decrease Quantity">−</button>' +
                  '<input type="number" class="qty-edit-input" data-action="set-qty" value="' + line.qty + '" min="1" step="1" inputmode="decimal">' +
                  '<button type="button" data-action="inc-qty" title="Increase Quantity">+</button>' +
                '</div>' +
                '<span class="qty-unit">pcs</span>' +
              '</div>' +
            '</div>' +
            '<div class="actions">' +
              '<button class="icon-btn" data-action="edit" title="Edit Product or Add-ons">✎</button>' +
              '<button class="icon-btn danger" data-action="remove" title="Remove">×</button>' +
            '</div>' +
          '</div>' +
          '<div class="breakdown">' + breakdownRows + '</div>' +
          upsellHtml +
          '<div class="quote-item-foot">' +
            '<div class="foot-unit"><span class="lbl">Unit Price (all-in)</span><span class="val">' + fmt(unitPrice) + '</span></div>' +
            '<div class="foot-total"><span class="lbl">Line Total</span><span class="val">' + fmt(calc.lineTotal) + '</span></div>' +
          '</div>';
        list.appendChild(row);
      });

      // Inline "Add another product" CTA after the items
      const addMore = document.createElement('div');
      addMore.className = 'quote-addmore';
      addMore.innerHTML = '<button type="button" id="quote-addmore-btn" class="quote-addmore-btn"><span class="icon">+</span> Add Another Product</button>';
      list.appendChild(addMore);
    }

    $('quote-count').textContent = quote.length + ' item' + (quote.length === 1 ? '' : 's');
    updateTotals();
  }

  function updateTotals() {
    let itemsSub = 0, addonsSub = 0;
    quote.forEach(line => {
      const c = calcLine(line);
      itemsSub += c.itemTotal;
      addonsSub += c.addonTotal;
    });
    const total = itemsSub + addonsSub;
    const markupAmt = total * (markup / 100);
    const suggested = total + markupAmt;
    $('items-subtotal').textContent = fmt(itemsSub);
    $('addons-subtotal').textContent = fmt(addonsSub);
    $('total').textContent = fmt(total);
    $('markup-amt').textContent = fmt(markupAmt);
    $('suggest').textContent = fmt(suggested);
    $('markup-display').textContent = markup + '%';

    // Platform fee: gross-up so seller nets the suggested price after fee
    // Formula: list = suggested / (1 - rate)   |   fee = list - suggested
    const feeRow = $('platform-fee-row');
    const finalRow = $('platform-final-row');
    if (platform !== 'none') {
      const rate = platformRates[platform];
      if (rate > 0 && rate < 100) {
        const rateDec = rate / 100;
        const listPrice = suggested / (1 - rateDec);
        const fee = listPrice - suggested;
        const niceName = platform.charAt(0).toUpperCase() + platform.slice(1);

        // Apply platform-specific classes
        feeRow.className = 'summary-row platform-row ' + platform;
        finalRow.className = 'summary-row platform-final-row ' + platform;
        feeRow.style.display = '';
        finalRow.style.display = '';

        $('platform-fee-label').textContent = niceName + ' Fee';
        $('platform-rate-display').textContent = rate + '%';
        $('platform-fee-amt').textContent = fmt(fee);
        $('platform-final-label').textContent = niceName;
        $('platform-list-price').textContent = fmt(listPrice);
      } else {
        feeRow.style.display = 'none';
        finalRow.style.display = 'none';
      }
    } else {
      feeRow.style.display = 'none';
      finalRow.style.display = 'none';
    }
  }

  function escapeHtml(s) {
    return String(s).replace(/[&<>"']/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
  }

  // ---- Quote item handlers ----
  $('quote-list').addEventListener('click', (e) => {
    // Handle inline "add another product" CTA at end of list
    if (e.target.closest('#quote-addmore-btn')) {
      openModal(null);
      return;
    }
    const btn = e.target.closest('[data-action]');
    if (!btn) return;
    const row = btn.closest('.quote-item');
    if (!row) return;
    const id = Number(row.dataset.id);
    const action = btn.dataset.action;
    if (action === 'remove') {
      quote = quote.filter(q => q.id !== id);
      renderQuote();
      showToast('Item Removed');
    } else if (action === 'edit') {
      openModal(id);
    } else if (action === 'edit-addon-price') {
      const addonCode = btn.dataset.code;
      const currentPrice = Number(btn.dataset.price);
      // Replace button with an inline input
      const input = document.createElement('input');
      input.type = 'number';
      input.inputMode = 'decimal';
      input.className = 'bd-unit bd-unit-input';
      input.value = currentPrice.toFixed(2);
      input.step = '0.01';
      input.min = '0';
      input.dataset.lineId = id;
      input.dataset.code = addonCode;
      input.dataset.original = currentPrice;
      btn.parentNode.replaceChild(input, btn);
      input.focus();
      input.select();

      const commit = (cancel) => {
        const line = quote.find(q => q.id === id);
        if (!line) { renderQuote(); return; }
        if (cancel) { renderQuote(); return; }
        const newPrice = Number(input.value);
        if (isNaN(newPrice) || newPrice < 0) { renderQuote(); return; }
        // If unchanged from original (standard tier price), do nothing special
        if (newPrice === currentPrice) { renderQuote(); return; }
        if (!line.overrides) line.overrides = { addons: {} };
        if (!line.overrides.addons) line.overrides.addons = {};
        line.overrides.addons[addonCode] = newPrice;
        renderQuote();
        showToast('Price Set to RM ' + newPrice.toFixed(2));
      };

      input.addEventListener('blur', () => commit(false));
      input.addEventListener('keydown', (ev) => {
        if (ev.key === 'Enter') { ev.preventDefault(); input.blur(); }
        else if (ev.key === 'Escape') { ev.preventDefault(); commit(true); }
      });
      return;
    } else if (action === 'remove-addon') {
      const addonCode = btn.dataset.addon;
      const line = quote.find(q => q.id === id);
      if (line) {
        line.addonCodes = line.addonCodes.filter(c => c !== addonCode);
        if (line.addonCounts) delete line.addonCounts[addonCode];
        if (line.overrides && line.overrides.addons) delete line.overrides.addons[addonCode];
        renderQuote();
        showToast('Add-on Removed');
      }
    } else if (action === 'inc-qty') {
      const line = quote.find(q => q.id === id);
      if (line) {
        line.qty = (line.qty || 0) + 1;
        line.dismissedUpsells = [];
        renderQuote();
      }
    } else if (action === 'dec-qty') {
      const line = quote.find(q => q.id === id);
      if (line && line.qty > 1) {
        line.qty -= 1;
        line.dismissedUpsells = [];
        renderQuote();
      }
    } else if (action === 'inc-addon-count') {
      const addonCode = btn.dataset.code;
      const line = quote.find(q => q.id === id);
      if (line) {
        if (!line.addonCounts) line.addonCounts = {};
        const cur = line.addonCounts[addonCode] || 1;
        line.addonCounts[addonCode] = cur + 1;
        renderQuote();
      }
    } else if (action === 'dec-addon-count') {
      const addonCode = btn.dataset.code;
      const line = quote.find(q => q.id === id);
      if (line) {
        if (!line.addonCounts) line.addonCounts = {};
        const cur = line.addonCounts[addonCode] || 1;
        if (cur > 1) {
          line.addonCounts[addonCode] = cur - 1;
          if (line.addonCounts[addonCode] === 1) delete line.addonCounts[addonCode]; // keep map clean
          renderQuote();
        }
      }
    } else if (action === 'add-addon') {
      openModal(id);
    } else if (action === 'apply-upsell') {
      const newQty = Number(btn.dataset.newqty);
      const line = quote.find(q => q.id === id);
      if (line && newQty > 0) {
        line.qty = newQty;
        line.dismissedUpsells = [];
        renderQuote();
        showToast('Quantity Bumped to ' + newQty);
      }
    } else if (action === 'override-price') {
      const kind = btn.dataset.kind;
      const code = btn.dataset.code;
      const newPrice = Number(btn.dataset.newprice);
      const line = quote.find(q => q.id === id);
      if (line && !isNaN(newPrice)) {
        if (!line.overrides) line.overrides = { addons: {} };
        if (!line.overrides.addons) line.overrides.addons = {};
        if (kind === 'item') {
          line.overrides.item = newPrice;
        } else {
          line.overrides.addons[code] = newPrice;
        }
        renderQuote();
        showToast('Price Overridden to RM ' + newPrice.toFixed(2));
      }
    } else if (action === 'dismiss-upsell') {
      const key = btn.dataset.key;
      const line = quote.find(q => q.id === id);
      if (line && key) {
        if (!line.dismissedUpsells) line.dismissedUpsells = [];
        if (!line.dismissedUpsells.includes(key)) line.dismissedUpsells.push(key);
        renderQuote();
      }
    } else if (action === 'reset-override') {
      const kind = btn.dataset.kind;
      const code = btn.dataset.code;
      const line = quote.find(q => q.id === id);
      if (line && line.overrides) {
        if (kind === 'item') {
          delete line.overrides.item;
        } else if (line.overrides.addons) {
          delete line.overrides.addons[code];
        }
        renderQuote();
        showToast('Price Reset');
      }
    }
  });

  // Inline qty typing — re-render and restore focus to the qty input
  $('quote-list').addEventListener('input', (e) => {
    const action = e.target.dataset.action;
    if (action !== 'set-qty' && action !== 'set-addon-count') return;
    const row = e.target.closest('.quote-item');
    if (!row) return;
    const id = Number(row.dataset.id);
    const line = quote.find(q => q.id === id);
    if (!line) return;
    const newVal = Number(e.target.value);
    if (isNaN(newVal) || newVal < 1) return;  // ignore empty / 0 — let them keep typing

    if (action === 'set-qty') {
      if (newVal === line.qty) return;
      line.qty = newVal;
      line.dismissedUpsells = [];
    } else { // set-addon-count
      const code = e.target.dataset.code;
      if (!code) return;
      if (!line.addonCounts) line.addonCounts = {};
      const cur = line.addonCounts[code] || 1;
      if (newVal === cur) return;
      if (newVal === 1) delete line.addonCounts[code];
      else line.addonCounts[code] = newVal;
    }

    // Save selection state so we can restore focus + cursor after re-render
    const cursorPos = e.target.selectionStart;
    const inHead = !!e.target.closest('.quote-item-head');
    const inBreakdown = !!e.target.closest('.breakdown');
    const targetCode = e.target.dataset.code || null;
    renderQuote();
    // Restore focus to the matching input on the same line
    const newRow = $('quote-list').querySelector('.quote-item[data-id="' + id + '"]');
    if (newRow) {
      let scope = newRow;
      if (inHead) scope = newRow.querySelector('.quote-item-head') || newRow;
      else if (inBreakdown) scope = newRow.querySelector('.breakdown') || newRow;
      let selector = '[data-action="' + action + '"]';
      if (targetCode) selector += '[data-code="' + targetCode + '"]';
      const input = scope.querySelector(selector);
      if (input) {
        input.focus();
        try { input.setSelectionRange(cursorPos, cursorPos); } catch (_) {}
      }
    }
  });

  // Allow empty value while typing without snapping back, but snap to 1 on blur
  $('quote-list').addEventListener('blur', (e) => {
    const action = e.target.dataset.action;
    if (action !== 'set-qty' && action !== 'set-addon-count') return;
    const v = Number(e.target.value);
    if (isNaN(v) || v < 1) {
      const row = e.target.closest('.quote-item');
      if (!row) return;
      const id = Number(row.dataset.id);
      const line = quote.find(q => q.id === id);
      if (!line) return;
      if (action === 'set-qty') {
        line.qty = 1;
      } else {
        const code = e.target.dataset.code;
        if (code && line.addonCounts) delete line.addonCounts[code];
      }
      renderQuote();
    }
  }, true);  // capture phase since blur doesn't bubble

  // ---- Modal ----
  function openModal(editId) {
    editingId = editId || null;
    if (editId) {
      const line = quote.find(q => q.id === editId);
      modalSelected = {
        productCode: line.productCode,
        qty: line.qty,
        addonCodes: [...line.addonCodes],
      };
      $('modal-title').textContent = 'Edit Item';
      $('modal-confirm').textContent = 'Save Changes';
    } else {
      modalSelected = { productCode: null, qty: 50, addonCodes: [] };
      $('modal-title').textContent = 'Add Product';
      $('modal-confirm').textContent = 'Add to Quote';
    }
    modalCategory = modalSelected.productCode ? PRODUCTS[modalSelected.productCode].category : 'All';
    modalSearch = '';
    $('search-input').value = '';
    $('qty-input').value = modalSelected.qty;
    renderCategories();
    renderProductList();
    renderAddons();
    updateModalPreview();
    $('modal').classList.add('show');
    // Reset scroll position so the list always starts from the top
    requestAnimationFrame(() => {
      const grid = $('product-grid');
      if (grid) grid.scrollTop = 0;
      const body = document.querySelector('.modal-body');
      if (body) body.scrollTop = 0;
    });
    // Auto-focus search on desktop only — iOS Safari has quirks with focus()
    // that can prevent the modal from displaying correctly
    if (window.innerWidth > 640) {
      setTimeout(() => $('search-input').focus(), 50);
    }
  }

  function closeModal() {
    $('modal').classList.remove('show');
    editingId = null;
    // Defensive reset: ensure next openModal starts fully clean
    modalSelected = { productCode: null, qty: 50, addonCodes: [] };
    modalCategory = 'All';
    modalSearch = '';
  }

  // Expose openModal globally so the inline onclick fallback works
  // (defensive: in case addEventListener fails or DOM timing is off on iOS Safari)
  window.openAddProduct = () => openModal(null);

  $('add-product-btn').addEventListener('click', () => openModal(null));
  $('modal-close').addEventListener('click', closeModal);
  $('modal-cancel').addEventListener('click', closeModal);
  $('modal').addEventListener('click', (e) => {
    if (e.target.id === 'modal') closeModal();
  });
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && $('modal').classList.contains('show')) closeModal();
  });

  // ---- Categories ----
  function getCategories() {
    const counts = { All: Object.keys(PRODUCTS).length };
    Object.values(PRODUCTS).forEach(p => {
      counts[p.category] = (counts[p.category] || 0) + 1;
    });
    // Sort categories by count desc, but keep 'All' first
    const cats = Object.keys(counts).filter(c => c !== 'All').sort((a, b) => counts[b] - counts[a]);
    return ['All'].concat(cats).map(c => ({ name: c, count: counts[c] }));
  }

  function renderCategories() {
    const wrap = $('cat-pills');
    wrap.innerHTML = '';
    getCategories().forEach(cat => {
      const btn = document.createElement('button');
      btn.className = 'cat-pill' + (cat.name === modalCategory ? ' active' : '');
      btn.type = 'button';
      btn.innerHTML = escapeHtml(cat.name) + ' <span class="count">' + cat.count + '</span>';
      btn.addEventListener('click', () => {
        modalCategory = cat.name;
        renderCategories();
        renderProductList();
      });
      wrap.appendChild(btn);
    });
  }

  function renderProductList() {
    const grid = $('product-grid');
    grid.innerHTML = '';
    const search = modalSearch.trim().toLowerCase();
    const filtered = Object.entries(PRODUCTS).filter(([code, p]) => {
      if (modalCategory !== 'All' && p.category !== modalCategory) return false;
      if (search) {
        const hay = (code + ' ' + p.item + ' ' + p.brand).toLowerCase();
        if (!hay.includes(search)) return false;
      }
      return true;
    });

    if (filtered.length === 0) {
      grid.innerHTML = '<div class="empty-msg">No products match. Try a different search or category.</div>';
      return;
    }

    filtered.forEach(([code, p]) => {
      const fromPrice = p.tiers && p.tiers.length ? p.tiers[p.tiers.length - 1].price : null;
      const opt = document.createElement('div');
      opt.className = 'product-option' + (modalSelected.productCode === code ? ' selected' : '');
      opt.dataset.code = code;
      opt.innerHTML =
        '<div class="left">' +
          '<div class="code">' + escapeHtml(code) + '</div>' +
          '<div>' +
            '<div class="name">' + escapeHtml(p.item) + '</div>' +
            '<div class="brand-tag">' + escapeHtml(p.brand) + '</div>' +
          '</div>' +
        '</div>' +
        '<div class="from">' + (fromPrice ? 'from <strong>RM ' + fromPrice.toFixed(2) + '</strong>' : '<em>no stock</em>') + '</div>';
      opt.addEventListener('click', () => {
        modalSelected.productCode = code;
        renderProductList();
        updateModalPreview();
      });
      grid.appendChild(opt);
    });
  }

  $('search-input').addEventListener('input', (e) => {
    modalSearch = e.target.value;
    renderProductList();
  });

  // ---- Quantity ----
  function setQty(val) {
    val = Math.max(1, Number(val) || 1);
    modalSelected.qty = val;
    $('qty-input').value = val;
    updateModalPreview();
  }
  $('qty-input').addEventListener('input', (e) => setQty(e.target.value));
  $('qty-minus').addEventListener('click', () => setQty(modalSelected.qty - 1));
  $('qty-plus').addEventListener('click', () => setQty(modalSelected.qty + 1));
  document.querySelectorAll('.qty-quick button').forEach(b => {
    b.addEventListener('click', () => setQty(b.dataset.qty));
  });

  // ---- Add-ons ----
  function renderAddons() {
    const wrap = $('addon-list');
    wrap.innerHTML = '';
    Object.entries(ADDONS).forEach(([code, a]) => {
      const fromPrice = a.tiers && a.tiers.length ? a.tiers[a.tiers.length - 1].price : null;
      const checked = modalSelected.addonCodes.includes(code);
      const card = document.createElement('label');
      card.className = 'addon-card-opt' + (checked ? ' selected' : '');
      card.innerHTML =
        '<input type="checkbox" data-code="' + escapeHtml(code) + '"' + (checked ? ' checked' : '') + '>' +
        '<div class="label-text">' +
          '<div class="name">' + escapeHtml(a.item) + '</div>' +
          '<span class="from">' + escapeHtml(code) + (fromPrice ? ' · from RM ' + fromPrice.toFixed(2) : '') + '</span>' +
        '</div>';
      const cb = card.querySelector('input');
      cb.addEventListener('change', () => {
        if (cb.checked) {
          if (!modalSelected.addonCodes.includes(code)) modalSelected.addonCodes.push(code);
        } else {
          modalSelected.addonCodes = modalSelected.addonCodes.filter(c => c !== code);
        }
        card.classList.toggle('selected', cb.checked);
        updateModalPreview();
      });
      wrap.appendChild(card);
    });
  }

  function updateModalPreview() {
    if (!modalSelected.productCode) {
      $('modal-unit-price').textContent = 'RM 0.00';
      $('modal-line-total').textContent = 'RM 0.00';
      $('modal-confirm').disabled = true;
      $('modal-confirm').style.opacity = 0.5;
      $('modal-confirm').style.cursor = 'not-allowed';
      return;
    }
    $('modal-confirm').disabled = false;
    $('modal-confirm').style.opacity = 1;
    $('modal-confirm').style.cursor = 'pointer';
    const calc = calcLine({ productCode: modalSelected.productCode, qty: modalSelected.qty, addonCodes: modalSelected.addonCodes });
    const unitPrice = modalSelected.qty > 0 ? calc.lineTotal / modalSelected.qty : 0;
    $('modal-unit-price').textContent = fmt(unitPrice);
    $('modal-line-total').textContent = fmt(calc.lineTotal);
  }

  $('modal-confirm').addEventListener('click', () => {
    if (!modalSelected.productCode) return;
    if (editingId) {
      const line = quote.find(q => q.id === editingId);
      line.productCode = modalSelected.productCode;
      line.qty = modalSelected.qty;
      line.addonCodes = [...modalSelected.addonCodes];
      // Drop counts for add-ons that are no longer selected
      if (line.addonCounts) {
        Object.keys(line.addonCounts).forEach(code => {
          if (!line.addonCodes.includes(code)) delete line.addonCounts[code];
        });
      }
      showToast('Item Updated');
    } else {
      quote.push({
        id: nextId++,
        productCode: modalSelected.productCode,
        qty: modalSelected.qty,
        addonCodes: [...modalSelected.addonCodes],
      });
      showToast('Added to Quote');
    }
    closeModal();
    renderQuote();
  });

  // ---- Clear, markup, export, print ----
  $('clear-quote-btn').addEventListener('click', () => {
    if (quote.length === 0) return;
    if (!confirm('Clear the Entire Quote?')) return;
    quote = [];
    renderQuote();
  });

  function setMarkup(val) {
    markup = Math.max(0, Number(val) || 0);
    $('markup-input').value = markup;
    document.querySelectorAll('.preset-btn').forEach(b => {
      b.classList.toggle('active', Number(b.dataset.val) === markup);
    });
    updateTotals();
  }
  document.querySelectorAll('.preset-btn').forEach(btn => {
    btn.addEventListener('click', () => setMarkup(btn.dataset.val));
  });
  $('markup-input').addEventListener('input', (e) => setMarkup(e.target.value));

  // Platform fee selector
  function setPlatform(p) {
    platform = p;
    document.querySelectorAll('.platform-pill').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.platform === p);
    });
    const editWrap = $('platform-rate-edit');
    if (p === 'none') {
      editWrap.style.display = 'none';
    } else {
      editWrap.style.display = '';
      $('platform-rate-lbl').textContent = p.charAt(0).toUpperCase() + p.slice(1);
      $('platform-rate').value = platformRates[p];
    }
    updateTotals();
  }
  document.querySelectorAll('.platform-pill').forEach(btn => {
    btn.addEventListener('click', () => setPlatform(btn.dataset.platform));
  });
  $('platform-rate').addEventListener('input', (e) => {
    if (platform === 'none') return;
    const v = Number(e.target.value);
    if (!isNaN(v) && v >= 0 && v < 100) {
      platformRates[platform] = v;
      updateTotals();
    }
  });

  $('print-btn').addEventListener('click', () => window.print());

  $('export-btn').addEventListener('click', () => {
    if (quote.length === 0) {
      showToast('Quote Is Empty');
      return;
    }
    const lines = [['#', 'Code', 'Item', 'Brand', 'Qty', 'Unit (RM)', 'Item Subtotal (RM)', 'Add-ons (RM)', 'Combined Unit (RM)', 'Line Total (RM)']];
    let itemsSub = 0, addonsSub = 0;
    quote.forEach((line, i) => {
      const calc = calcLine(line);
      const prod = PRODUCTS[line.productCode];
      const combinedUnit = line.qty > 0 ? calc.lineTotal / line.qty : 0;
      lines.push([
        i+1, line.productCode, prod.item, prod.brand, line.qty,
        (calc.prod.price || 0).toFixed(2),
        calc.itemTotal.toFixed(2),
        calc.addonTotal.toFixed(2),
        combinedUnit.toFixed(2),
        calc.lineTotal.toFixed(2),
      ]);
      calc.addonDetails.forEach(a => {
        const itemName = a.count > 1 ? a.product.item + ' (×' + a.count + ')' : a.product.item;
        const qtyDisplay = a.count > 1 ? line.qty + ' × ' + a.count : line.qty;
        lines.push(['', '  ↳ ' + a.code, itemName, a.product.brand, qtyDisplay, a.price.toFixed(2), '', a.total.toFixed(2), '', '']);
      });
      itemsSub += calc.itemTotal;
      addonsSub += calc.addonTotal;
    });
    const total = itemsSub + addonsSub;
    const suggested = total * (1 + markup/100);
    lines.push([]);
    lines.push(['','','','','','','Items Subtotal', itemsSub.toFixed(2), '', '']);
    lines.push(['','','','','','','Add-ons Subtotal', addonsSub.toFixed(2), '', '']);
    lines.push(['','','','','','','Cost Total', total.toFixed(2), '', '']);
    lines.push(['','','','','','', 'Profit (markup ' + markup + '%)', (total * markup/100).toFixed(2), '', '']);
    lines.push(['','','','','','','Suggested Price', suggested.toFixed(2), '', '']);
    if (platform !== 'none') {
      const rate = platformRates[platform];
      if (rate > 0 && rate < 100) {
        const listPrice = suggested / (1 - rate / 100);
        const fee = listPrice - suggested;
        const niceName = platform.charAt(0).toUpperCase() + platform.slice(1);
        lines.push(['','','','','','', niceName + ' Fee (' + rate + '%)', fee.toFixed(2), '', '']);
        lines.push(['','','','','','','List Price on ' + niceName, listPrice.toFixed(2), '', '']);
      }
    }

    const csv = lines.map(r => r.map(c => {
      const s = String(c == null ? '' : c);
      return /[",\n]/.test(s) ? '"' + s.replace(/"/g, '""') + '"' : s;
    }).join(',')).join('\n');

    const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'agent-quote-' + new Date().toISOString().slice(0,10) + '.csv';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    showToast('Quote Exported');
  });

  // ---- Toast ----
  let toastTimer;
  function showToast(msg) {
    const t = $('toast');
    t.textContent = msg;
    t.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => t.classList.remove('show'), 1800);
  }

  // ---- Init ----
  $('ts').textContent = new Date().toLocaleDateString('en-MY', { day: '2-digit', month: 'short', year: 'numeric' }) + ' · BUILD-7';
  renderQuote();
})();
</script>
</body>
</html>
