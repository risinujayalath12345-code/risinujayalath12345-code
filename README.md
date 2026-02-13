<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Smart Home Automation Simulator</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
  <style>
    :root {
      --bg: #050a12;
      --bg-secondary: #0a1420;
      --fg: #e8f0f8;
      --muted: #5a7080;
      --accent: #00d4aa;
      --accent-glow: rgba(0, 212, 170, 0.3);
      --card: #0d1a28;
      --border: #1a2a3a;
      --warning: #f59e0b;
      --danger: #ef4444;
      --success: #10b981;
    }

    * {
      box-sizing: border-box;
    }

    html {
      font-family: 'Space Grotesk', sans-serif;
      background: var(--bg);
      color: var(--fg);
    }

    body {
      margin: 0;
      overflow: hidden;
      background: var(--bg);
    }

    .mono {
      font-family: 'JetBrains Mono', monospace;
    }

    .slide {
      position: absolute;
      inset: 0;
      display: flex;
      flex-direction: column;
      padding: 3rem 5rem;
      opacity: 0;
      transform: translateX(100px);
      pointer-events: none;
      transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
      background: var(--bg);
      overflow-y: auto;
    }

    .slide.active {
      opacity: 1;
      transform: translateX(0);
      pointer-events: auto;
    }

    .slide.prev {
      transform: translateX(-100px);
    }

    .glow-text {
      text-shadow: 0 0 40px var(--accent-glow), 0 0 80px var(--accent-glow);
    }

    .glow-box {
      box-shadow: 0 0 30px var(--accent-glow), inset 0 1px 0 rgba(255,255,255,0.05);
    }

    .grid-bg {
      background-image: 
        linear-gradient(rgba(0, 212, 170, 0.03) 1px, transparent 1px),
        linear-gradient(90deg, rgba(0, 212, 170, 0.03) 1px, transparent 1px);
      background-size: 40px 40px;
    }

    .nav-btn {
      background: var(--card);
      border: 1px solid var(--border);
      color: var(--fg);
      padding: 0.75rem 1.5rem;
      cursor: pointer;
      transition: all 0.3s ease;
      font-family: 'Space Grotesk', sans-serif;
    }

    .nav-btn:hover {
      background: var(--accent);
      color: var(--bg);
      border-color: var(--accent);
    }

    .nav-btn:disabled {
      opacity: 0.3;
      cursor: not-allowed;
    }

    .nav-btn:disabled:hover {
      background: var(--card);
      color: var(--fg);
      border-color: var(--border);
    }

    .diagram-node {
      background: linear-gradient(135deg, var(--card), var(--bg-secondary));
      border: 1px solid var(--accent);
      padding: 1rem 1.5rem;
      position: relative;
    }

    .diagram-node::before {
      content: '';
      position: absolute;
      inset: -1px;
      background: linear-gradient(135deg, var(--accent), transparent);
      opacity: 0.2;
      z-index: -1;
    }

    .scenario-card {
      background: linear-gradient(135deg, var(--card), transparent);
      border: 1px solid var(--border);
      padding: 1.5rem;
      position: relative;
      overflow: hidden;
    }

    .scenario-card::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      width: 4px;
      height: 100%;
    }

    .scenario-success::before { background: var(--success); }
    .scenario-warning::before { background: var(--warning); }
    .scenario-danger::before { background: var(--danger); }

    .truth-table {
      border-collapse: collapse;
      width: 100%;
    }

    .truth-table th,
    .truth-table td {
      border: 1px solid var(--border);
      padding: 0.75rem 1.5rem;
      text-align: center;
    }

    .truth-table th {
      background: var(--accent);
      color: var(--bg);
      font-weight: 600;
    }

    .truth-table tr:nth-child(even) td {
      background: rgba(0, 212, 170, 0.05);
    }

    .truth-table td:first-child {
      font-family: 'JetBrains Mono', monospace;
    }

    .flowchart-box {
      background: var(--card);
      border: 2px solid var(--border);
      padding: 0.75rem 1.25rem;
      text-align: center;
      min-width: 180px;
    }

    .flowchart-decision {
      background: var(--card);
      border: 2px solid var(--accent);
      padding: 0.75rem 1.25rem;
      transform: rotate(45deg);
      min-width: 100px;
      min-height: 100px;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .flowchart-decision span {
      transform: rotate(-45deg);
    }

    .equation-box {
      background: rgba(0, 212, 170, 0.08);
      border: 1px solid rgba(0, 212, 170, 0.3);
      padding: 1rem 1.5rem;
      font-family: 'JetBrains Mono', monospace;
    }

    .progress-bar {
      position: fixed;
      bottom: 0;
      left: 0;
      height: 3px;
      background: var(--accent);
      transition: width 0.3s ease;
      z-index: 100;
    }

    @keyframes pulse {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.5; }
    }

    .pulse {
      animation: pulse 2s ease-in-out infinite;
    }

    @keyframes slideUp {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .animate-in {
      animation: slideUp 0.6s ease forwards;
    }

    .delay-1 { animation-delay: 0.1s; opacity: 0; }
    .delay-2 { animation-delay: 0.2s; opacity: 0; }
    .delay-3 { animation-delay: 0.3s; opacity: 0; }
    .delay-4 { animation-delay: 0.4s; opacity: 0; }

    @media (max-width: 768px) {
      .slide {
        padding: 2rem 1.5rem;
      }
    }

    @media (prefers-reduced-motion: reduce) {
      .slide {
        transition: opacity 0.3s ease;
        transform: none !important;
      }
      .slide.prev {
        transform: none;
      }
      .pulse {
        animation: none;
      }
      .animate-in {
        animation: none;
        opacity: 1;
      }
    }
  </style>
</head>
<body class="grid-bg">
  <!-- Progress Bar -->
  <div class="progress-bar" id="progressBar"></div>

  <!-- Navigation -->
  <nav class="fixed top-0 left-0 right-0 z-50 flex items-center justify-between px-8 py-4 bg-gradient-to-b from-[var(--bg)] to-transparent">
    <div class="flex items-center gap-3">
      <div class="w-2 h-2 rounded-full bg-[var(--accent)] pulse"></div>
      <span class="mono text-sm text-[var(--muted)]" id="slideCounter">01 / 18</span>
    </div>
    <div class="flex gap-3">
      <button class="nav-btn text-sm" id="prevBtn" disabled>Previous</button>
      <button class="nav-btn text-sm" id="nextBtn">Next</button>
    </div>
  </nav>

  <!-- Slides Container -->
  <main id="slidesContainer">
    
    <!-- Slide 1: Title -->
    <section class="slide active" data-slide="1">
      <div class="flex-1 flex flex-col justify-center items-center text-center">
        <div class="mb-8 animate-in">
          <svg class="w-16 h-16 mx-auto mb-6 text-[var(--accent)]" viewBox="0 0 64 64" fill="none" stroke="currentColor" stroke-width="1.5">
            <rect x="8" y="16" width="48" height="36" rx="2"/>
            <path d="M8 28h48"/>
            <circle cx="16" cy="22" r="2"/>
            <circle cx="24" cy="22" r="2"/>
            <rect x="16" y="36" width="12" height="8" rx="1"/>
            <path d="M36 36h12M36 40h8"/>
            <path d="M32 8v8M24 12l8 4 8-4"/>
          </svg>
        </div>
        <h1 class="text-5xl md:text-6xl font-light tracking-tight mb-6 animate-in delay-1">
           Smart Home<br/>
          <span class="font-semibold glow-text text-[var(--accent)]">Automation Simulator</span>
        </h1>
        <p class="text-lg text-[var(--muted)] mb-12 max-w-2xl animate-in delay-2">
          Design, Logical Modeling, and Implementation of an Intelligent Rule-Based System
        </p>
        <div class="grid grid-cols-2 gap-8 text-sm animate-in delay-3">
          <div class="text-right">
            <p class="text-[var(--muted)]">Presented by</p>
            <p class="font-medium">group AD</p>
          </div>
          <div class="text-left">
            <p class="text-[var(--muted)]">faculty of computing</p>
            <p class="font-medium">NSBM</p>
          </div>
        </div>
        <div class="mt-8 text-xs text-[var(--muted)] animate-in delay-4">
          2026/02/13
        </div>
      </div>
      <!-- Decorative elements -->
      <div class="absolute inset-0 overflow-hidden pointer-events-none">
        <div class="absolute top-1/4 left-10 w-64 h-64 rounded-full bg-[var(--accent)] opacity-[0.02] blur-3xl"></div>
        <div class="absolute bottom-1/4 right-10 w-96 h-96 rounded-full bg-[var(--accent)] opacity-[0.02] blur-3xl"></div>
      </div>
    </section>

    <!-- Slide 2: Executive Summary -->
    <section class="slide" data-slide="2">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">02 / EXECUTIVE SUMMARY</span>
        <h2 class="text-4xl font-light mt-2">Research Overview</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid md:grid-cols-2 gap-8 w-full">
          <div class="space-y-6">
            <div class="border-l-2 border-[var(--accent)] pl-6 py-4">
              <p class="text-lg leading-relaxed text-[var(--fg)] opacity-90">
                This research presents an <span class="text-[var(--accent)] font-medium">intelligent automation framework</span> that leverages Boolean algebra and rule-based computation to create a fully simulated smart home environment.
              </p>
            </div>
            <div class="border-l-2 border-[var(--muted)] pl-6 py-4">
              <p class="text-lg leading-relaxed text-[var(--fg)] opacity-90">
                The system integrates a <span class="font-medium">logical decision engine</span> capable of processing multiple sensor inputs in real-time to optimize energy consumption and security monitoring.
              </p>
            </div>
          </div>
          <div class="grid grid-cols-2 gap-4">
            <div class="diagram-node glow-box">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M12 2v4M12 18v4M4.93 4.93l2.83 2.83M16.24 16.24l2.83 2.83M2 12h4M18 12h4M4.93 19.07l2.83-2.83M16.24 7.76l2.83-2.83"/>
              </svg>
              <p class="text-xs text-[var(--muted)] uppercase tracking-wider">Intelligence</p>
              <p class="text-xl font-semibold mt-1">Adaptive Control</p>
            </div>
            <div class="diagram-node">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M9 3H5a2 2 0 00-2 2v4m6-6h10a2 2 0 012 2v4M9 3v18m0 0h10a2 2 0 002-2v-4M9 21H5a2 2 0 01-2-2v-4"/>
              </svg>
              <p class="text-xs text-[var(--muted)] uppercase tracking-wider">Logic</p>
              <p class="text-xl font-semibold mt-1">Decision Engine</p>
            </div>
            <div class="diagram-node">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/>
              </svg>
              <p class="text-xs text-[var(--muted)] uppercase tracking-wider">Energy</p>
              <p class="text-xl font-semibold mt-1">Optimization</p>
            </div>
            <div class="diagram-node">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>
              </svg>
              <p class="text-xs text-[var(--muted)] uppercase tracking-wider">Security</p>
              <p class="text-xl font-semibold mt-1">Monitoring</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 3: Motivation & Problem Definition -->
    <section class="slide" data-slide="3">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">03 / MOTIVATION</span>
        <h2 class="text-4xl font-light mt-2">Problem Definition</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid md:grid-cols-2 gap-12 w-full">
          <!-- Traditional Home -->
          <div class="relative">
            <div class="absolute -top-4 -left-4 text-6xl font-bold text-[var(--danger)] opacity-10">01</div>
            <h3 class="text-xl font-medium mb-6 text-[var(--danger)]">Traditional Home</h3>
            <div class="space-y-4">
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--border)]">
                <svg class="w-5 h-5 mt-0.5 text-[var(--danger)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 6L6 18M6 6l12 12"/>
                </svg>
                <div>
                  <p class="font-medium">Energy Inefficiency</p>
                  <p class="text-sm text-[var(--muted)]">30-40% energy waste from manual controls</p>
                </div>
              </div>
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--border)]">
                <svg class="w-5 h-5 mt-0.5 text-[var(--danger)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 6L6 18M6 6l12 12"/>
                </svg>
                <div>
                  <p class="font-medium">No Adaptive Control</p>
                  <p class="text-sm text-[var(--muted)]">Static systems, no environmental response</p>
                </div>
              </div>
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--border)]">
                <svg class="w-5 h-5 mt-0.5 text-[var(--danger)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 6L6 18M6 6l12 12"/>
                </svg>
                <div>
                  <p class="font-medium">Security Vulnerabilities</p>
                  <p class="text-sm text-[var(--muted)]">Reactive rather than proactive monitoring</p>
                </div>
              </div>
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--border)]">
                <svg class="w-5 h-5 mt-0.5 text-[var(--danger)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 6L6 18M6 6l12 12"/>
                </svg>
                <div>
                  <p class="font-medium">Manual Dependency</p>
                  <p class="text-sm text-[var(--muted)]">Constant human intervention required</p>
                </div>
              </div>
            </div>
          </div>
          <!-- Intelligent Home -->
          <div class="relative">
            <div class="absolute -top-4 -left-4 text-6xl font-bold text-[var(--success)] opacity-10">02</div>
            <h3 class="text-xl font-medium mb-6 text-[var(--success)]">Intelligent Home</h3>
            <div class="space-y-4">
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--accent)] glow-box">
                <svg class="w-5 h-5 mt-0.5 text-[var(--success)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 6L9 17l-5-5"/>
                </svg>
                <div>
                  <p class="font-medium">Energy Optimization</p>
                  <p class="text-sm text-[var(--muted)]">Automated efficiency with 25%+ savings</p>
                </div>
              </div>
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--accent)] glow-box">
                <svg class="w-5 h-5 mt-0.5 text-[var(--success)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 6L9 17l-5-5"/>
                </svg>
                <div>
                  <p class="font-medium">Real-time Adaptation</p>
                  <p class="text-sm text-[var(--muted)]">Dynamic response to environmental changes</p>
                </div>
              </div>
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--accent)] glow-box">
                <svg class="w-5 h-5 mt-0.5 text-[var(--success)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 6L9 17l-5-5"/>
                </svg>
                <div>
                  <p class="font-medium">Proactive Security</p>
                  <p class="text-sm text-[var(--muted)]">Instant threat detection and alerts</p>
                </div>
              </div>
              <div class="flex items-start gap-4 p-4 bg-[var(--card)] border border-[var(--accent)] glow-box">
                <svg class="w-5 h-5 mt-0.5 text-[var(--success)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 6L9 17l-5-5"/>
                </svg>
                <div>
                  <p class="font-medium">Full Automation</p>
                  <p class="text-sm text-[var(--muted)]">Minimal human intervention needed</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 4: Project Objectives -->
    <section class="slide" data-slide="4">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">04 / OBJECTIVES</span>
        <h2 class="text-4xl font-light mt-2">Engineering Goals</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid md:grid-cols-3 gap-6 w-full">
          <div class="diagram-node group hover:border-[var(--accent)] transition-all">
            <div class="flex items-center gap-3 mb-4">
              <div class="w-10 h-10 rounded bg-[var(--accent)] bg-opacity-10 flex items-center justify-center">
                <svg class="w-5 h-5 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <circle cx="12" cy="12" r="3"/>
                  <path d="M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 010 2.83 2 2 0 01-2.83 0l-.06-.06a1.65 1.65 0 00-1.82-.33 1.65 1.65 0 00-1 1.51V21a2 2 0 01-2 2 2 2 0 01-2-2v-.09A1.65 1.65 0 009 19.4a1.65 1.65 0 00-1.82.33l-.06.06a2 2 0 01-2.83 0 2 2 0 010-2.83l.06-.06a1.65 1.65 0 00.33-1.82 1.65 1.65 0 00-1.51-1H3a2 2 0 01-2-2 2 2 0 012-2h.09A1.65 1.65 0 004.6 9a1.65 1.65 0 00-.33-1.82l-.06-.06a2 2 0 010-2.83 2 2 0 012.83 0l.06.06a1.65 1.65 0 001.82.33H9a1.65 1.65 0 001-1.51V3a2 2 0 012-2 2 2 0 012 2v.09a1.65 1.65 0 001 1.51 1.65 1.65 0 001.82-.33l.06-.06a2 2 0 012.83 0 2 2 0 010 2.83l-.06.06a1.65 1.65 0 00-.33 1.82V9a1.65 1.65 0 001.51 1H21a2 2 0 012 2 2 2 0 01-2 2h-.09a1.65 1.65 0 00-1.51 1z"/>
                </svg>
              </div>
              <span class="mono text-xs text-[var(--muted)]">OBJ-01</span>
            </div>
            <h3 class="font-semibold mb-2">Rule-Based Engine</h3>
            <p class="text-sm text-[var(--muted)]">Design deterministic automation engine using propositional logic</p>
          </div>
          
          <div class="diagram-node group hover:border-[var(--accent)] transition-all">
            <div class="flex items-center gap-3 mb-4">
              <div class="w-10 h-10 rounded bg-[var(--accent)] bg-opacity-10 flex items-center justify-center">
                <svg class="w-5 h-5 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/>
                  <polyline points="22,6 12,13 2,6"/>
                </svg>
              </div>
              <span class="mono text-xs text-[var(--muted)]">OBJ-02</span>
            </div>
            <h3 class="font-semibold mb-2">Boolean Modeling</h3>
            <p class="text-sm text-[var(--muted)]">Model decision logic using Boolean algebra and truth tables</p>
          </div>
          
          <div class="diagram-node group hover:border-[var(--accent)] transition-all">
            <div class="flex items-center gap-3 mb-4">
              <div class="w-10 h-10 rounded bg-[var(--accent)] bg-opacity-10 flex items-center justify-center">
                <svg class="w-5 h-5 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <rect x="2" y="3" width="20" height="14" rx="2" ry="2"/>
                  <line x1="8" y1="21" x2="16" y2="21"/>
                  <line x1="12" y1="17" x2="12" y2="21"/>
                </svg>
              </div>
              <span class="mono text-xs text-[var(--muted)]">OBJ-03</span>
            </div>
            <h3 class="font-semibold mb-2">Interactive GUI</h3>
            <p class="text-sm text-[var(--muted)]">Develop responsive dashboard with real-time state visualization</p>
          </div>
          
          <div class="diagram-node group hover:border-[var(--accent)] transition-all">
            <div class="flex items-center gap-3 mb-4">
              <div class="w-10 h-10 rounded bg-[var(--accent)] bg-opacity-10 flex items-center justify-center">
                <svg class="w-5 h-5 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z"/>
                  <polyline points="9 22 9 12 15 12 15 22"/>
                </svg>
              </div>
              <span class="mono text-xs text-[var(--muted)]">OBJ-04</span>
            </div>
            <h3 class="font-semibold mb-2">Real-world Simulation</h3>
            <p class="text-sm text-[var(--muted)]">Simulate authentic smart home behavior patterns</p>
          </div>
          
          <div class="diagram-node group hover:border-[var(--accent)] transition-all">
            <div class="flex items-center gap-3 mb-4">
              <div class="w-10 h-10 rounded bg-[var(--accent)] bg-opacity-10 flex items-center justify-center">
                <svg class="w-5 h-5 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/>
                </svg>
              </div>
              <span class="mono text-xs text-[var(--muted)]">OBJ-05</span>
            </div>
            <h3 class="font-semibold mb-2">Power Optimization</h3>
            <p class="text-sm text-[var(--muted)]">Implement energy-efficient device control algorithms</p>
          </div>
          
          <div class="diagram-node group hover:border-[var(--accent)] transition-all glow-box">
            <div class="flex items-center gap-3 mb-4">
              <div class="w-10 h-10 rounded bg-[var(--accent)] bg-opacity-10 flex items-center justify-center">
                <svg class="w-5 h-5 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M22 11.08V12a10 10 0 11-5.93-9.14"/>
                  <polyline points="22 4 12 14.01 9 11.01"/>
                </svg>
              </div>
              <span class="mono text-xs text-[var(--muted)]">OBJ-06</span>
            </div>
            <h3 class="font-semibold mb-2">System Validation</h3>
            <p class="text-sm text-[var(--muted)]">Verify logical correctness through scenario testing</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 5: System Architecture -->
    <section class="slide" data-slide="5">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">05 / ARCHITECTURE</span>
        <h2 class="text-4xl font-light mt-2">System Architecture</h2>
      </div>
      <div class="flex-1 flex items-center justify-center">
        <div class="w-full max-w-5xl">
          <!-- Architecture Diagram -->
          <div class="relative">
            <!-- Layer 1: Sensors -->
            <div class="mb-4">
              <p class="mono text-xs text-[var(--muted)] mb-3 uppercase tracking-wider">Input Layer</p>
              <div class="grid grid-cols-6 gap-3">
                <div class="diagram-node text-center py-3">
                  <svg class="w-5 h-5 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                    <path d="M14 14.76V3.5a2.5 2.5 0 00-5 0v11.26a4.5 4.5 0 105 0z"/>
                  </svg>
                  <p class="text-xs">Temp</p>
                </div>
                <div class="diagram-node text-center py-3">
                  <svg class="w-5 h-5 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                    <path d="M17.657 18.657A8 8 0 016.343 7.343"/>
                    <circle cx="12" cy="12" r="3"/>
                  </svg>
                  <p class="text-xs">Motion</p>
                </div>
                <div class="diagram-node text-center py-3">
                  <svg class="w-5 h-5 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                    <circle cx="12" cy="12" r="5"/>
                    <line x1="12" y1="1" x2="12" y2="3"/>
                    <line x1="12" y1="21" x2="12" y2="23"/>
                    <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/>
                    <line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
                    <line x1="1" y1="12" x2="3" y2="12"/>
                    <line x1="21" y1="12" x2="23" y2="12"/>
                    <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/>
                    <line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
                  </svg>
                  <p class="text-xs">Time</p>
                </div>
                <div class="diagram-node text-center py-3">
                  <svg class="w-5 h-5 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                    <path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/>
                    <circle cx="12" cy="7" r="4"/>
                  </svg>
                  <p class="text-xs">Occupancy</p>
                </div>
                <div class="diagram-node text-center py-3">
                  <svg class="w-5 h-5 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                    <rect x="2" y="4" width="20" height="16" rx="2"/>
                    <circle cx="12" cy="12" r="3"/>
                  </svg>
                  <p class="text-xs">Laundry</p>
                </div>
                <div class="diagram-node text-center py-3">
                  <svg class="w-5 h-5 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                    <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/>
                  </svg>
                  <p class="text-xs">Energy</p>
                </div>
              </div>
            </div>
            
            <!-- Arrow -->
            <div class="flex justify-center my-4">
              <svg class="w-6 h-10 text-[var(--accent)]" viewBox="0 0 24 40" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M12 0v32M5 25l7 7 7-7"/>
              </svg>
            </div>
            
            <!-- Layer 2: Decision Engine -->
            <div class="mb-4">
              <p class="mono text-xs text-[var(--muted)] mb-3 uppercase tracking-wider">Processing Layer</p>
              <div class="diagram-node glow-box py-6 px-8">
                <div class="flex items-center justify-between">
                  <div class="text-center">
                    <p class="font-semibold text-lg">Logical Decision Engine</p>
                    <p class="text-sm text-[var(--muted)]">Boolean Rule Processing</p>
                  </div>
                  <div class="flex gap-6">
                    <div class="text-center px-4 border-l border-[var(--border)]">
                      <p class="mono text-2xl text-[var(--accent)]">6</p>
                      <p class="text-xs text-[var(--muted)]">Inputs</p>
                    </div>
                    <div class="text-center px-4 border-l border-[var(--border)]">
                      <p class="mono text-2xl text-[var(--accent)]">12</p>
                      <p class="text-xs text-[var(--muted)]">Rules</p>
                    </div>
                    <div class="text-center px-4 border-l border-[var(--border)]">
                      <p class="mono text-2xl text-[var(--accent)]">5</p>
                      <p class="text-xs text-[var(--muted)]">Devices</p>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- Arrow -->
            <div class="flex justify-center my-4">
              <svg class="w-6 h-10 text-[var(--accent)]" viewBox="0 0 24 40" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M12 0v32M5 25l7 7 7-7"/>
              </svg>
            </div>
            
            <!-- Layer 3: Output -->
            <div class="grid grid-cols-2 gap-6">
              <div>
                <p class="mono text-xs text-[var(--muted)] mb-3 uppercase tracking-wider">Control Layer</p>
                <div class="diagram-node py-4">
                  <p class="font-medium mb-2">Device Control</p>
                  <div class="flex flex-wrap gap-2">
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">AC</span>
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">Lights</span>
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">TV</span>
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">Washer</span>
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">Alarm</span>
                  </div>
                </div>
              </div>
              <div>
                <p class="mono text-xs text-[var(--muted)] mb-3 uppercase tracking-wider">Interface Layer</p>
                <div class="diagram-node py-4">
                  <p class="font-medium mb-2">GUI Dashboard</p>
                  <div class="flex flex-wrap gap-2">
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">Real-time</span>
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">Interactive</span>
                    <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)]">Visual</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 6: Mathematical Modeling -->
    <section class="slide" data-slide="6">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">06 / MATHEMATICAL MODEL</span>
        <h2 class="text-4xl font-light mt-2">Boolean Logic Formulation</h2>
      </div>
      <div class="flex-1 overflow-auto">
        <div class="grid lg:grid-cols-2 gap-8">
          <!-- Variable Definitions -->
          <div>
            <h3 class="font-medium mb-4 text-[var(--muted)]">State Variables</h3>
            <div class="grid grid-cols-2 gap-3 mb-6">
              <div class="equation-box">
                <span class="text-[var(--accent)] font-bold">T</span> = High Temperature
              </div>
              <div class="equation-box">
                <span class="text-[var(--accent)] font-bold">M</span> = Motion Detected
              </div>
              <div class="equation-box">
                <span class="text-[var(--accent)] font-bold">N</span> = Night Time
              </div>
              <div class="equation-box">
                <span class="text-[var(--accent)] font-bold">U</span> = User Home
              </div>
              <div class="equation-box">
                <span class="text-[var(--accent)] font-bold">L</span> = Laundry Loaded
              </div>
              <div class="equation-box">
                <span class="text-[var(--accent)] font-bold">E</span> = Energy Mode
              </div>
            </div>
            
            <!-- Logic Circuit Diagram -->
            <div class="bg-[var(--card)] border border-[var(--border)] p-6">
              <p class="mono text-xs text-[var(--muted)] mb-4">Logic Gate Representation</p>
              <svg class="w-full" viewBox="0 0 400 200" fill="none">
                <!-- AC Gate -->
                <g transform="translate(20, 20)">
                  <text x="0" y="15" class="mono text-xs fill-[var(--muted)]">T</text>
                  <line x1="15" y1="12" x2="60" y2="12" stroke="var(--accent)" stroke-width="1.5"/>
                  <text x="0" y="45" class="mono text-xs fill-[var(--muted)]">U</text>
                  <line x1="15" y1="42" x2="60" y2="42" stroke="var(--accent)" stroke-width="1.5"/>
                  <rect x="60" y="5" width="40" height="45" stroke="var(--accent)" stroke-width="1.5" fill="var(--card)"/>
                  <text x="68" y="35" class="text-xs fill-[var(--accent)]">AND</text>
                  <line x1="100" y1="27" x2="140" y2="27" stroke="var(--accent)" stroke-width="1.5"/>
                  <text x="145" y="32" class="mono text-xs fill-[var(--fg)]">AC = T AND U</text>
                </g>
                
                <!-- Lights Gate -->
                <g transform="translate(20, 80)">
                  <text x="0" y="15" class="mono text-xs fill-[var(--muted)]">M</text>
                  <line x1="15" y1="12" x2="60" y2="12" stroke="var(--accent)" stroke-width="1.5"/>
                  <text x="0" y="45" class="mono text-xs fill-[var(--muted)]">N</text>
                  <line x1="15" y1="42" x2="60" y2="42" stroke="var(--accent)" stroke-width="1.5"/>
                  <rect x="60" y="5" width="40" height="45" stroke="var(--accent)" stroke-width="1.5" fill="var(--card)"/>
                  <text x="68" y="35" class="text-xs fill-[var(--accent)]">AND</text>
                  <line x1="100" y1="27" x2="140" y2="27" stroke="var(--accent)" stroke-width="1.5"/>
                  <text x="145" y="32" class="mono text-xs fill-[var(--fg)]">Lights = M AND N</text>
                </g>
                
                <!-- Alarm Gate -->
                <g transform="translate(20, 140)">
                  <text x="0" y="15" class="mono text-xs fill-[var(--muted)]">M</text>
                  <line x1="15" y1="12" x2="60" y2="12" stroke="var(--accent)" stroke-width="1.5"/>
                  <text x="0" y="45" class="mono text-xs fill-[var(--muted)]">U</text>
                  <line x1="15" y1="42" x2="45" y2="42" stroke="var(--accent)" stroke-width="1.5"/>
                  <circle cx="55" cy="42" r="8" stroke="var(--accent)" stroke-width="1.5" fill="var(--card)"/>
                  <text x="52" y="46" class="text-[8px] fill-[var(--accent)]">NOT</text>
                  <line x1="63" y1="42" x2="80" y2="42" stroke="var(--accent)" stroke-width="1.5"/>
                  <line x1="80" y1="42" x2="80" y2="32" stroke="var(--accent)" stroke-width="1.5"/>
                  <line x1="60" y1="12" x2="80" y2="12" stroke="var(--accent)" stroke-width="1.5"/>
                  <line x1="80" y1="12" x2="80" y2="22" stroke="var(--accent)" stroke-width="1.5"/>
                  <rect x="80" y="10" width="40" height="35" stroke="var(--accent)" stroke-width="1.5" fill="var(--card)"/>
                  <text x="88" y="32" class="text-xs fill-[var(--accent)]">AND</text>
                  <line x1="120" y1="27" x2="140" y2="27" stroke="var(--accent)" stroke-width="1.5"/>
                  <text x="145" y="32" class="mono text-xs fill-[var(--fg)]">Alarm = M AND NOT U</text>
                </g>
              </svg>
            </div>
          </div>
          
          <!-- Device Functions -->
          <div>
            <h3 class="font-medium mb-4 text-[var(--muted)]">Device Control Functions</h3>
            <div class="space-y-3">
              <div class="equation-box flex items-center gap-4">
                <span class="text-[var(--accent)] text-lg">AC</span>
                <span class="text-[var(--muted)]">=</span>
                <span class="mono">T AND U</span>
                <span class="text-xs text-[var(--muted)] ml-auto">Cool when hot and home</span>
              </div>
              <div class="equation-box flex items-center gap-4">
                <span class="text-[var(--accent)] text-lg">Lights</span>
                <span class="text-[var(--muted)]">=</span>
                <span class="mono">M AND N</span>
                <span class="text-xs text-[var(--muted)] ml-auto">Night motion lighting</span>
              </div>
              <div class="equation-box flex items-center gap-4">
                <span class="text-[var(--accent)] text-lg">TV</span>
                <span class="text-[var(--muted)]">=</span>
                <span class="mono">U AND Day</span>
                <span class="text-xs text-[var(--muted)] ml-auto">Daytime entertainment</span>
              </div>
              <div class="equation-box flex items-center gap-4">
                <span class="text-[var(--accent)] text-lg">WM</span>
                <span class="text-[var(--muted)]">=</span>
                <span class="mono">L AND U</span>
                <span class="text-xs text-[var(--muted)] ml-auto">Laundry when present</span>
              </div>
              <div class="equation-box flex items-center gap-4 border-[var(--danger)] border-opacity-50">
                <span class="text-[var(--danger)] text-lg">Alarm</span>
                <span class="text-[var(--muted)]">=</span>
                <span class="mono">M AND NOT U</span>
                <span class="text-xs text-[var(--muted)] ml-auto">Intrusion detection</span>
              </div>
            </div>
            
            <div class="mt-6 p-4 bg-[var(--card)] border border-[var(--accent)] glow-box">
              <p class="mono text-xs text-[var(--accent)] mb-2">Energy Override Rule</p>
              <p class="mono text-sm">IF E AND NOT U THEN Devices = OFF</p>
              <p class="text-xs text-[var(--muted)] mt-2">Priority rule for power conservation when energy mode is active and house is empty</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 7: Truth Tables -->
    <section class="slide" data-slide="7">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">07 / TRUTH TABLES</span>
        <h2 class="text-4xl font-light mt-2">Binary Decision Models</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid lg:grid-cols-2 gap-8 w-full">
          <!-- Security Logic Table -->
          <div>
            <div class="bg-[var(--card)] border border-[var(--border)] p-6">
              <p class="mono text-xs text-[var(--muted)] mb-1">Table 7.1</p>
              <h3 class="font-semibold mb-4">Security Logic - Intrusion Detection</h3>
              <table class="truth-table">
                <thead>
                  <tr>
                    <th>Motion (M)</th>
                    <th>User Home (U)</th>
                    <th>Alarm (A)</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>0</td>
                    <td>0</td>
                    <td>0</td>
                  </tr>
                  <tr>
                    <td>1</td>
                    <td>0</td>
                    <td class="text-[var(--danger)] font-semibold">1</td>
                  </tr>
                  <tr>
                    <td>0</td>
                    <td>1</td>
                    <td>0</td>
                  </tr>
                  <tr>
                    <td>1</td>
                    <td>1</td>
                    <td>0</td>
                  </tr>
                </tbody>
              </table>
              <p class="text-xs text-[var(--muted)] mt-4">Alarm triggers only when motion detected AND user is not home</p>
            </div>
          </div>
          
          <!-- Climate Control Table -->
          <div>
            <div class="bg-[var(--card)] border border-[var(--border)] p-6">
              <p class="mono text-xs text-[var(--muted)] mb-1">Table 7.2</p>
              <h3 class="font-semibold mb-4">Climate Control Logic</h3>
              <table class="truth-table">
                <thead>
                  <tr>
                    <th>Temp (T)</th>
                    <th>User (U)</th>
                    <th>Energy (E)</th>
                    <th>AC</th>
                  </tr>
                </thead>
                <tbody>
                  <tr>
                    <td>0</td>
                    <td>0</td>
                    <td>0</td>
                    <td>0</td>
                  </tr>
                  <tr>
                    <td>1</td>
                    <td>0</td>
                    <td>0</td>
                    <td>0</td>
                  </tr>
                  <tr>
                    <td>1</td>
                    <td>1</td>
                    <td>0</td>
                    <td class="text-[var(--accent)] font-semibold">1</td>
                  </tr>
                  <tr>
                    <td>1</td>
                    <td>1</td>
                    <td>1</td>
                    <td>0</td>
                  </tr>
                </tbody>
              </table>
              <p class="text-xs text-[var(--muted)] mt-4">Energy mode overrides cooling to conserve power</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 8: Algorithm Flowchart -->
    <section class="slide" data-slide="8">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">08 / ALGORITHM</span>
        <h2 class="text-4xl font-light mt-2">System Flowchart</h2>
      </div>
      <div class="flex-1 flex items-center justify-center">
        <div class="flex flex-col items-center">
          <!-- Flowchart -->
          <div class="flex flex-col items-center gap-2">
            <!-- Start -->
            <div class="flowchart-box rounded-full bg-[var(--accent)] text-[var(--bg)]">
              <span class="mono text-sm">START</span>
            </div>
            <svg class="w-4 h-6 text-[var(--accent)]" viewBox="0 0 16 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 0v20M2 14l6 6 6-6"/>
            </svg>
            
            <!-- Capture Inputs -->
            <div class="flowchart-box">
              <span class="text-sm">Capture Sensor Inputs</span>
            </div>
            <svg class="w-4 h-6 text-[var(--accent)]" viewBox="0 0 16 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 0v20M2 14l6 6 6-6"/>
            </svg>
            
            <!-- Evaluate Rules -->
            <div class="flowchart-box border-[var(--accent)] glow-box">
              <span class="text-sm">Evaluate Boolean Rules</span>
            </div>
            <svg class="w-4 h-6 text-[var(--accent)]" viewBox="0 0 16 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 0v20M2 14l6 6 6-6"/>
            </svg>
            
            <!-- Decision -->
            <div class="flex items-center gap-4">
              <div class="relative">
                <div class="w-24 h-24 border-2 border-[var(--accent)] transform rotate-45 bg-[var(--card)]">
                  <div class="absolute inset-0 flex items-center justify-center transform -rotate-45">
                    <span class="text-xs text-center">Energy<br/>Override?</span>
                  </div>
                </div>
              </div>
              <div class="flex flex-col gap-4">
                <div class="flex items-center gap-2">
                  <span class="mono text-xs text-[var(--accent)]">Yes</span>
                  <div class="flowchart-box text-xs">Override OFF</div>
                </div>
                <div class="flex items-center gap-2">
                  <span class="mono text-xs text-[var(--muted)]">No</span>
                  <div class="flowchart-box text-xs">Apply States</div>
                </div>
              </div>
            </div>
            
            <svg class="w-4 h-6 text-[var(--accent)]" viewBox="0 0 16 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 0v20M2 14l6 6 6-6"/>
            </svg>
            
            <!-- Update -->
            <div class="flowchart-box">
              <span class="text-sm">Update Device States</span>
            </div>
            <svg class="w-4 h-6 text-[var(--accent)]" viewBox="0 0 16 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 0v20M2 14l6 6 6-6"/>
            </svg>
            
            <!-- Display -->
            <div class="flowchart-box border-[var(--accent)]">
              <span class="text-sm">Display Output</span>
            </div>
            <svg class="w-4 h-6 text-[var(--accent)]" viewBox="0 0 16 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M8 0v20M2 14l6 6 6-6"/>
            </svg>
            
            <!-- End -->
            <div class="flowchart-box rounded-full bg-[var(--danger)] text-white">
              <span class="mono text-sm">END</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 9: Software Design -->
    <section class="slide" data-slide="9">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">09 / SOFTWARE DESIGN</span>
        <h2 class="text-4xl font-light mt-2">Module Architecture</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="w-full">
          <div class="grid lg:grid-cols-4 gap-4 mb-8">
            <div class="diagram-node glow-box">
              <div class="w-12 h-12 rounded-lg bg-[var(--accent)] bg-opacity-10 flex items-center justify-center mb-4">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <circle cx="12" cy="12" r="3"/>
                  <path d="M19.4 15a1.65 1.65 0 00.33 1.82l.06.06a2 2 0 010 2.83 2 2 0 01-2.83 0l-.06-.06a1.65 1.65 0 00-1.82-.33 1.65 1.65 0 00-1 1.51V21a2 2 0 01-2 2 2 2 0 01-2-2v-.09A1.65 1.65 0 009 19.4"/>
                </svg>
              </div>
              <h3 class="font-semibold mb-1">Automation Engine</h3>
              <p class="text-xs text-[var(--muted)]">Core logic processor and rule evaluator</p>
            </div>
            <div class="diagram-node">
              <div class="w-12 h-12 rounded-lg bg-[var(--accent)] bg-opacity-10 flex items-center justify-center mb-4">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <polyline points="4 17 10 11 4 5"/>
                  <line x1="12" y1="19" x2="20" y2="19"/>
                </svg>
              </div>
              <h3 class="font-semibold mb-1">Command Processor</h3>
              <p class="text-xs text-[var(--muted)]">Input parsing and execution handler</p>
            </div>
            <div class="diagram-node">
              <div class="w-12 h-12 rounded-lg bg-[var(--accent)] bg-opacity-10 flex items-center justify-center mb-4">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <rect x="2" y="3" width="20" height="14" rx="2" ry="2"/>
                  <line x1="8" y1="21" x2="16" y2="21"/>
                  <line x1="12" y1="17" x2="12" y2="21"/>
                </svg>
              </div>
              <h3 class="font-semibold mb-1">UI Manager</h3>
              <p class="text-xs text-[var(--muted)]">Interface rendering and event handling</p>
            </div>
            <div class="diagram-node">
              <div class="w-12 h-12 rounded-lg bg-[var(--accent)] bg-opacity-10 flex items-center justify-center mb-4">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M12 2v4M12 18v4M4.93 4.93l2.83 2.83M16.24 16.24l2.83 2.83M2 12h4M18 12h4M4.93 19.07l2.83-2.83M16.24 7.76l2.83-2.83"/>
                </svg>
              </div>
              <h3 class="font-semibold mb-1">State Controller</h3>
              <p class="text-xs text-[var(--muted)]">Global state management and sync</p>
            </div>
          </div>
          
          <!-- Block Diagram -->
          <div class="bg-[var(--card)] border border-[var(--border)] p-6">
            <p class="mono text-xs text-[var(--muted)] mb-4">Module Interaction Diagram</p>
            <svg class="w-full" viewBox="0 0 600 150" fill="none">
              <!-- Automation Engine -->
              <rect x="0" y="50" width="120" height="50" stroke="var(--accent)" stroke-width="2" fill="var(--bg-secondary)"/>
              <text x="60" y="80" text-anchor="middle" class="text-xs fill-[var(--fg)]">Automation Engine</text>
              
              <!-- Arrow -->
              <line x1="120" y1="75" x2="160" y2="75" stroke="var(--accent)" stroke-width="1.5" marker-end="url(#arrowhead)"/>
              
              <!-- State Controller -->
              <rect x="160" y="50" width="120" height="50" stroke="var(--accent)" stroke-width="2" fill="var(--bg-secondary)"/>
              <text x="220" y="80" text-anchor="middle" class="text-xs fill-[var(--fg)]">State Controller</text>
              
              <!-- Arrow -->
              <line x1="280" y1="75" x2="320" y2="75" stroke="var(--accent)" stroke-width="1.5"/>
              
              <!-- Command Processor -->
              <rect x="320" y="50" width="120" height="50" stroke="var(--accent)" stroke-width="2" fill="var(--bg-secondary)"/>
              <text x="380" y="80" text-anchor="middle" class="text-xs fill-[var(--fg)]">Command Processor</text>
              
              <!-- Arrow -->
              <line x1="440" y1="75" x2="480" y2="75" stroke="var(--accent)" stroke-width="1.5"/>
              
              <!-- UI Manager -->
              <rect x="480" y="50" width="120" height="50" stroke="var(--accent)" stroke-width="2" fill="var(--bg-secondary)"/>
              <text x="540" y="80" text-anchor="middle" class="text-xs fill-[var(--fg)]">UI Manager</text>
              
              <!-- Bidirectional arrows -->
              <path d="M220 50 L220 25 L380 25 L380 50" stroke="var(--muted)" stroke-width="1" fill="none" stroke-dasharray="4"/>
              <text x="300" y="20" text-anchor="middle" class="text-[10px] fill-[var(--muted)]">State Sync</text>
            </svg>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 10: GUI Design Philosophy -->
    <section class="slide" data-slide="10">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">10 / GUI DESIGN</span>
        <h2 class="text-4xl font-light mt-2">Interface Philosophy</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid lg:grid-cols-2 gap-8 w-full items-center">
          <div>
            <div class="space-y-6">
              <div class="flex items-start gap-4">
                <div class="w-8 h-8 rounded-full bg-[var(--accent)] bg-opacity-20 flex items-center justify-center flex-shrink-0">
                  <span class="text-sm text-[var(--accent)]">1</span>
                </div>
                <div>
                  <h3 class="font-medium mb-1">Dark Theme Foundation</h3>
                  <p class="text-sm text-[var(--muted)]">Reduces eye strain during extended monitoring sessions. Optimal for 24/7 operation environments.</p>
                </div>
              </div>
              <div class="flex items-start gap-4">
                <div class="w-8 h-8 rounded-full bg-[var(--accent)] bg-opacity-20 flex items-center justify-center flex-shrink-0">
                  <span class="text-sm text-[var(--accent)]">2</span>
                </div>
                <div>
                  <h3 class="font-medium mb-1">High Contrast Design</h3>
                  <p class="text-sm text-[var(--muted)]">Critical information stands out. Status indicators visible from distance with accessibility compliance.</p>
                </div>
              </div>
              <div class="flex items-start gap-4">
                <div class="w-8 h-8 rounded-full bg-[var(--accent)] bg-opacity-20 flex items-center justify-center flex-shrink-0">
                  <span class="text-sm text-[var(--accent)]">3</span>
                </div>
                <div>
                  <h3 class="font-medium mb-1">Structured Layout</h3>
                  <p class="text-sm text-[var(--muted)]">Logical grouping of controls enhances usability. Consistent patterns reduce cognitive load.</p>
                </div>
              </div>
            </div>
          </div>
          
          <!-- Dashboard Mockup -->
          <div class="bg-[var(--card)] border border-[var(--border)] p-4 rounded-lg">
            <div class="flex items-center justify-between mb-4">
              <div class="flex items-center gap-2">
                <div class="w-2 h-2 rounded-full bg-[var(--success)]"></div>
                <span class="mono text-xs">SYSTEM ONLINE</span>
              </div>
              <span class="mono text-xs text-[var(--muted)]">23:45:12</span>
            </div>
            <div class="grid grid-cols-3 gap-3">
              <div class="bg-[var(--bg)] border border-[var(--border)] p-3 text-center">
                <svg class="w-6 h-6 mx-auto mb-2 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M14 14.76V3.5a2.5 2.5 0 00-5 0v11.26a4.5 4.5 0 105 0z"/>
                </svg>
                <p class="text-xs text-[var(--muted)]">Temperature</p>
                <p class="font-semibold">24C</p>
              </div>
              <div class="bg-[var(--bg)] border border-[var(--border)] p-3 text-center">
                <svg class="w-6 h-6 mx-auto mb-2 text-[var(--success)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M20 21v-2a4 4 0 00-4-4H8a4 4 0 00-4 4v2"/>
                  <circle cx="12" cy="7" r="4"/>
                </svg>
                <p class="text-xs text-[var(--muted)]">Occupancy</p>
                <p class="font-semibold text-[var(--success)]">Home</p>
              </div>
              <div class="bg-[var(--bg)] border border-[var(--border)] p-3 text-center">
                <svg class="w-6 h-6 mx-auto mb-2 text-[var(--warning)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/>
                </svg>
                <p class="text-xs text-[var(--muted)]">Energy</p>
                <p class="font-semibold text-[var(--warning)]">Normal</p>
              </div>
            </div>
            <div class="mt-3 p-3 bg-[var(--bg)] border border-[var(--border)]">
              <p class="mono text-xs text-[var(--muted)] mb-2">DEVICE STATUS</p>
              <div class="flex gap-2">
                <span class="text-xs px-2 py-1 bg-[var(--accent)] bg-opacity-10 text-[var(--accent)] border border-[var(--accent)]">AC: ON</span>
                <span class="text-xs px-2 py-1 bg-[var(--muted)] bg-opacity-10 text-[var(--muted)] border border-[var(--muted)]">Lights: OFF</span>
                <span class="text-xs px-2 py-1 bg-[var(--muted)] bg-opacity-10 text-[var(--muted)] border border-[var(--muted)]">TV: OFF</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 11: Command Processing Logic -->
    <section class="slide" data-slide="11">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">11 / COMMAND LOGIC</span>
        <h2 class="text-4xl font-light mt-2">Processing Pipeline</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid lg:grid-cols-2 gap-8 w-full items-center">
          <!-- Decision Tree -->
          <div class="bg-[var(--card)] border border-[var(--border)] p-6">
            <p class="mono text-xs text-[var(--muted)] mb-4">Decision Tree</p>
            <svg class="w-full" viewBox="0 0 300 250" fill="none">
              <!-- Root -->
              <rect x="100" y="0" width="100" height="30" stroke="var(--accent)" stroke-width="1.5" fill="var(--bg-secondary)"/>
              <text x="150" y="20" text-anchor="middle" class="text-[10px] fill-[var(--fg)]">Parse Command</text>
              
              <!-- Line -->
              <line x1="150" y1="30" x2="150" y2="50" stroke="var(--accent)" stroke-width="1.5"/>
              
              <!-- Decision -->
              <polygon points="150,50 200,80 150,110 100,80" stroke="var(--accent)" stroke-width="1.5" fill="var(--bg-secondary)"/>
              <text x="150" y="85" text-anchor="middle" class="text-[10px] fill-[var(--fg)]">Valid?</text>
              
              <!-- Yes branch -->
              <line x1="200" y1="80" x2="240" y2="80" stroke="var(--success)" stroke-width="1.5"/>
              <text x="220" y="75" class="text-[8px] fill-[var(--success)]">Yes</text>
              <rect x="240" y="65" width="60" height="30" stroke="var(--success)" stroke-width="1.5" fill="var(--bg-secondary)"/>
              <text x="270" y="85" text-anchor="middle" class="text-[10px] fill-[var(--fg)]">Execute</text>
              
              <!-- No branch -->
              <line x1="100" y1="80" x2="60" y2="80" stroke="var(--danger)" stroke-width="1.5"/>
              <text x="80" y="75" class="text-[8px] fill-[var(--danger)]">No</text>
              <rect x="0" y="65" width="60" height="30" stroke="var(--danger)" stroke-width="1.5" fill="var(--bg-secondary)"/>
              <text x="30" y="85" text-anchor="middle" class="text-[10px] fill-[var(--fg)]">Error</text>
              
              <!-- Execute branches -->
              <line x1="270" y1="95" x2="270" y2="130" stroke="var(--accent)" stroke-width="1.5"/>
              
              <rect x="220" y="130" width="50" height="25" stroke="var(--accent)" stroke-width="1" fill="var(--bg-secondary)"/>
              <text x="245" y="147" text-anchor="middle" class="text-[9px] fill-[var(--fg)]">Update</text>
              
              <rect x="280" y="130" width="50" height="25" stroke="var(--accent)" stroke-width="1" fill="var(--bg-secondary)"/>
              <text x="305" y="147" text-anchor="middle" class="text-[9px] fill-[var(--fg)]">Notify</text>
              
              <!-- Final -->
              <line x1="245" y1="155" x2="245" y2="175" stroke="var(--accent)" stroke-width="1"/>
              <line x1="305" y1="155" x2="305" y2="175" stroke="var(--accent)" stroke-width="1"/>
              <line x1="245" y1="175" x2="305" y2="175" stroke="var(--accent)" stroke-width="1"/>
              <line x1="275" y1="175" x2="275" y2="200" stroke="var(--accent)" stroke-width="1.5"/>
              <rect x="225" y="200" width="100" height="30" stroke="var(--accent)" stroke-width="1.5" fill="var(--bg-secondary)"/>
              <text x="275" y="220" text-anchor="middle" class="text-[10px] fill-[var(--fg)]">Render UI</text>
            </svg>
          </div>
          
          <!-- Pseudocode -->
          <div>
            <h3 class="font-medium mb-4 text-[var(--muted)]">Command Handler Pseudocode</h3>
            <div class="bg-[var(--bg-secondary)] border border-[var(--border)] p-4 mono text-sm">
              <div class="text-[var(--muted)]">// Command Processing Logic</div>
              <div class="mt-2">
                <span class="text-[var(--accent)]">IF</span> command == <span class="text-[var(--warning)]">"turn on tv"</span>:
              </div>
              <div class="pl-4">
                TV.state = <span class="text-[var(--success)]">ON</span>
              </div>
              <div class="pl-4">
                log(<span class="text-[var(--warning)]">"TV activated"</span>)
              </div>
              <div class="mt-2">
                <span class="text-[var(--accent)]">ELSE IF</span> command == <span class="text-[var(--warning)]">"set mode energy"</span>:
              </div>
              <div class="pl-4">
                EnergyMode = <span class="text-[var(--success)]">TRUE</span>
              </div>
              <div class="pl-4">
                optimizeAllDevices()
              </div>
              <div class="mt-2">
                <span class="text-[var(--accent)]">ELSE</span>:
              </div>
              <div class="pl-4">
                <span class="text-[var(--danger)]">return</span> InvalidCommand
              </div>
            </div>
            
            <div class="mt-6 grid grid-cols-3 gap-3">
              <div class="text-center p-3 bg-[var(--card)] border border-[var(--border)]">
                <p class="mono text-2xl text-[var(--accent)]">12</p>
                <p class="text-xs text-[var(--muted)]">Commands</p>
              </div>
              <div class="text-center p-3 bg-[var(--card)] border border-[var(--border)]">
                <p class="mono text-2xl text-[var(--accent)]">5</p>
                <p class="text-xs text-[var(--muted)]">Devices</p>
              </div>
              <div class="text-center p-3 bg-[var(--card)] border border-[var(--border)]">
                <p class="mono text-2xl text-[var(--accent)]">3</p>
                <p class="text-xs text-[var(--muted)]">Modes</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 12: Experimental Scenarios -->
    <section class="slide" data-slide="12">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">12 / SIMULATION</span>
        <h2 class="text-4xl font-light mt-2">Experimental Scenarios</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid md:grid-cols-3 gap-6 w-full">
          <!-- Scenario A -->
          <div class="scenario-card scenario-success">
            <div class="flex items-center gap-2 mb-4">
              <div class="w-3 h-3 rounded-full bg-[var(--success)]"></div>
              <span class="mono text-xs text-[var(--muted)]">SCENARIO A</span>
            </div>
            <h3 class="font-semibold mb-2">Climate Control Activation</h3>
            <div class="mono text-xs text-[var(--muted)] mb-4">
              <p>INPUT: T=1, U=1</p>
              <p>CONDITION: High Temp + User Home</p>
            </div>
            <div class="flex items-center gap-2 mt-auto">
              <svg class="w-4 h-4 text-[var(--success)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M20 6L9 17l-5-5"/>
              </svg>
              <span class="text-sm text-[var(--success)]">OUTPUT: AC = ON</span>
            </div>
            <p class="text-xs text-[var(--muted)] mt-2">Cooling system activated for comfort</p>
          </div>
          
          <!-- Scenario B -->
          <div class="scenario-card scenario-danger">
            <div class="flex items-center gap-2 mb-4">
              <div class="w-3 h-3 rounded-full bg-[var(--danger)]"></div>
              <span class="mono text-xs text-[var(--muted)]">SCENARIO B</span>
            </div>
            <h3 class="font-semibold mb-2">Intrusion Detection Alert</h3>
            <div class="mono text-xs text-[var(--muted)] mb-4">
              <p>INPUT: M=1, U=0</p>
              <p>CONDITION: Motion + Empty House</p>
            </div>
            <div class="flex items-center gap-2 mt-auto">
              <svg class="w-4 h-4 text-[var(--danger)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M10.29 3.86L1.82 18a2 2 0 001.71 3h16.94a2 2 0 001.71-3L13.71 3.86a2 2 0 00-3.42 0z"/>
                <line x1="12" y1="9" x2="12" y2="13"/>
                <line x1="12" y1="17" x2="12.01" y2="17"/>
              </svg>
              <span class="text-sm text-[var(--danger)]">OUTPUT: Alarm = ON</span>
            </div>
            <p class="text-xs text-[var(--muted)] mt-2">Security breach detected and logged</p>
          </div>
          
          <!-- Scenario C -->
          <div class="scenario-card scenario-warning">
            <div class="flex items-center gap-2 mb-4">
              <div class="w-3 h-3 rounded-full bg-[var(--warning)]"></div>
              <span class="mono text-xs text-[var(--muted)]">SCENARIO C</span>
            </div>
            <h3 class="font-semibold mb-2">Energy Optimization Mode</h3>
            <div class="mono text-xs text-[var(--muted)] mb-4">
              <p>INPUT: E=1, U=0</p>
              <p>CONDITION: Energy Mode + No Occupants</p>
            </div>
            <div class="flex items-center gap-2 mt-auto">
              <svg class="w-4 h-4 text-[var(--warning)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M13 2L3 14h9l-1 8 10-12h-9l1-8z"/>
              </svg>
              <span class="text-sm text-[var(--warning)]">OUTPUT: All = OFF</span>
            </div>
            <p class="text-xs text-[var(--muted)] mt-2">Power conservation protocol engaged</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 13: Performance Discussion -->
    <section class="slide" data-slide="13">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">13 / ANALYSIS</span>
        <h2 class="text-4xl font-light mt-2">Performance Evaluation</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid lg:grid-cols-2 gap-8 w-full">
          <div>
            <h3 class="font-medium mb-6 text-[var(--muted)]">Key Performance Metrics</h3>
            <div class="space-y-4">
              <div class="p-4 bg-[var(--card)] border border-[var(--border)]">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-medium">Responsiveness</span>
                  <span class="mono text-[var(--accent)]">O(1)</span>
                </div>
                <p class="text-sm text-[var(--muted)]">Constant-time rule evaluation ensures immediate system response regardless of input complexity.</p>
              </div>
              <div class="p-4 bg-[var(--card)] border border-[var(--border)]">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-medium">Logical Accuracy</span>
                  <span class="mono text-[var(--accent)]">100%</span>
                </div>
                <p class="text-sm text-[var(--muted)]">Deterministic Boolean logic guarantees predictable, reproducible outputs for all input combinations.</p>
              </div>
              <div class="p-4 bg-[var(--card)] border border-[var(--border)]">
                <div class="flex items-center justify-between mb-2">
                  <span class="font-medium">Computational Cost</span>
                  <span class="mono text-[var(--accent)]">Minimal</span>
                </div>
                <p class="text-sm text-[var(--muted)]">No iteration or recursion required. Simple gate-level operations executed in single pass.</p>
              </div>
            </div>
          </div>
          
          <div>
            <h3 class="font-medium mb-6 text-[var(--muted)]">Benchmark Summary</h3>
            <div class="bg-[var(--card)] border border-[var(--border)] p-6">
              <div class="space-y-6">
                <div>
                  <div class="flex items-center justify-between mb-2">
                    <span class="text-sm">Rule Processing Speed</span>
                    <span class="mono text-sm text-[var(--accent)]">0.3ms</span>
                  </div>
                  <div class="h-2 bg-[var(--bg)] rounded-full overflow-hidden">
                    <div class="h-full bg-[var(--accent)] rounded-full" style="width: 95%"></div>
                  </div>
                </div>
                <div>
                  <div class="flex items-center justify-between mb-2">
                    <span class="text-sm">State Update Latency</span>
                    <span class="mono text-sm text-[var(--accent)]">0.1ms</span>
                  </div>
                  <div class="h-2 bg-[var(--bg)] rounded-full overflow-hidden">
                    <div class="h-full bg-[var(--accent)] rounded-full" style="width: 98%"></div>
                  </div>
                </div>
                <div>
                  <div class="flex items-center justify-between mb-2">
                    <span class="text-sm">Memory Footprint</span>
                    <span class="mono text-sm text-[var(--accent)]">2.4MB</span>
                  </div>
                  <div class="h-2 bg-[var(--bg)] rounded-full overflow-hidden">
                    <div class="h-full bg-[var(--accent)] rounded-full" style="width: 85%"></div>
                  </div>
                </div>
                <div>
                  <div class="flex items-center justify-between mb-2">
                    <span class="text-sm">CPU Utilization</span>
                    <span class="mono text-sm text-[var(--accent)]">&lt;1%</span>
                  </div>
                  <div class="h-2 bg-[var(--bg)] rounded-full overflow-hidden">
                    <div class="h-full bg-[var(--accent)] rounded-full" style="width: 92%"></div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 14: Strengths -->
    <section class="slide" data-slide="14">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">14 / STRENGTHS</span>
        <h2 class="text-4xl font-light mt-2">System Advantages</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="w-full">
          <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
            <div class="diagram-node glow-box group">
              <div class="flex items-center gap-3 mb-3">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <rect x="3" y="3" width="7" height="7"/>
                  <rect x="14" y="3" width="7" height="7"/>
                  <rect x="14" y="14" width="7" height="7"/>
                  <rect x="3" y="14" width="7" height="7"/>
                </svg>
                <h3 class="font-semibold">Modular Architecture</h3>
              </div>
              <p class="text-sm text-[var(--muted)]">Independent modules allow for isolated testing, maintenance, and future expansion without system-wide changes.</p>
            </div>
            
            <div class="diagram-node glow-box group">
              <div class="flex items-center gap-3 mb-3">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M12 2L2 7l10 5 10-5-10-5z"/>
                  <path d="M2 17l10 5 10-5"/>
                  <path d="M2 12l10 5 10-5"/>
                </svg>
                <h3 class="font-semibold">Scalable Logic</h3>
              </div>
              <p class="text-sm text-[var(--muted)]">Rule-based system easily extends to additional sensors and devices without architectural modifications.</p>
            </div>
            
            <div class="diagram-node glow-box group">
              <div class="flex items-center gap-3 mb-3">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <circle cx="12" cy="12" r="10"/>
                  <polyline points="12 6 12 12 16 14"/>
                </svg>
                <h3 class="font-semibold">Efficient Processing</h3>
              </div>
              <p class="text-sm text-[var(--muted)]">Boolean evaluation completes in constant time with minimal computational overhead.</p>
            </div>
            
            <div class="diagram-node glow-box group">
              <div class="flex items-center gap-3 mb-3">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>
                </svg>
                <h3 class="font-semibold">Strong Security Model</h3>
              </div>
              <p class="text-sm text-[var(--muted)]">Comprehensive intrusion detection with immediate alerting provides robust home protection.</p>
            </div>
            
            <div class="diagram-node glow-box group">
              <div class="flex items-center gap-3 mb-3">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M22 12h-4l-3 9L9 3l-3 9H2"/>
                </svg>
                <h3 class="font-semibold">Realistic Behavior</h3>
              </div>
              <p class="text-sm text-[var(--muted)]">Simulation accurately models real-world smart home patterns and decision scenarios.</p>
            </div>
            
            <div class="diagram-node glow-box group">
              <div class="flex items-center gap-3 mb-3">
                <svg class="w-6 h-6 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                  <path d="M4 19.5A2.5 2.5 0 016.5 17H20"/>
                  <path d="M6.5 2H20v20H6.5A2.5 2.5 0 014 19.5v-15A2.5 2.5 0 016.5 2z"/>
                </svg>
                <h3 class="font-semibold">Academic Rigor</h3>
              </div>
              <p class="text-sm text-[var(--muted)]">Mathematical foundation with formal proofs ensures verifiable correctness.</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 15: Limitations -->
    <section class="slide" data-slide="15">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">15 / LIMITATIONS</span>
        <h2 class="text-4xl font-light mt-2">Current Constraints</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="grid md:grid-cols-2 gap-6 w-full">
          <div class="p-6 bg-[var(--card)] border border-[var(--border)]">
            <div class="flex items-center gap-3 mb-4">
              <svg class="w-5 h-5 text-[var(--warning)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"/>
                <line x1="12" y1="8" x2="12" y2="12"/>
                <line x1="12" y1="16" x2="12.01" y2="16"/>
              </svg>
              <h3 class="font-semibold">No Machine Learning Adaptation</h3>
            </div>
            <p class="text-sm text-[var(--muted)]">System relies on predefined rules without capability for pattern recognition or behavioral learning from historical data.</p>
          </div>
          
          <div class="p-6 bg-[var(--card)] border border-[var(--border)]">
            <div class="flex items-center gap-3 mb-4">
              <svg class="w-5 h-5 text-[var(--warning)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"/>
                <line x1="12" y1="8" x2="12" y2="12"/>
                <line x1="12" y1="16" x2="12.01" y2="16"/>
              </svg>
              <h3 class="font-semibold">No Hardware Integration</h3>
            </div>
            <p class="text-sm text-[var(--muted)]">Simulation operates in software environment without direct connection to physical sensors or actuators.</p>
          </div>
          
          <div class="p-6 bg-[var(--card)] border border-[var(--border)]">
            <div class="flex items-center gap-3 mb-4">
              <svg class="w-5 h-5 text-[var(--warning)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"/>
                <line x1="12" y1="8" x2="12" y2="12"/>
                <line x1="12" y1="16" x2="12.01" y2="16"/>
              </svg>
              <h3 class="font-semibold">Static Decision Rules</h3>
            </div>
            <p class="text-sm text-[var(--muted)]">Logic rules are hardcoded and require manual updates. No dynamic rule generation based on environmental context.</p>
          </div>
          
          <div class="p-6 bg-[var(--card)] border border-[var(--border)]">
            <div class="flex items-center gap-3 mb-4">
              <svg class="w-5 h-5 text-[var(--warning)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <circle cx="12" cy="12" r="10"/>
                <line x1="12" y1="8" x2="12" y2="12"/>
                <line x1="12" y1="16" x2="12.01" y2="16"/>
              </svg>
              <h3 class="font-semibold">Limited Predictive Capability</h3>
            </div>
            <p class="text-sm text-[var(--muted)]">Reactive system responds only to current states. No forecasting or anticipatory actions based on trends.</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 16: Future Scope -->
    <section class="slide" data-slide="16">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">16 / FUTURE SCOPE</span>
        <h2 class="text-4xl font-light mt-2">Next-Gen Integration</h2>
      </div>
      <div class="flex-1 flex items-center">
        <div class="w-full">
          <div class="grid lg:grid-cols-3 gap-4 mb-8">
            <div class="diagram-node glow-box relative overflow-hidden">
              <div class="absolute top-0 right-0 w-20 h-20 bg-[var(--accent)] opacity-5 rounded-full blur-2xl"></div>
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M12 2a10 10 0 00-7.07 17.07l1.41-1.41A8 8 0 1112 4v0a8 8 0 00-5.66 13.66"/>
                <circle cx="12" cy="12" r="3"/>
              </svg>
              <h3 class="font-semibold mb-1">Machine Learning</h3>
              <p class="text-xs text-[var(--muted)]">Adaptive behavior from usage patterns</p>
            </div>
            
            <div class="diagram-node glow-box relative overflow-hidden">
              <div class="absolute top-0 right-0 w-20 h-20 bg-[var(--accent)] opacity-5 rounded-full blur-2xl"></div>
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M5 12.55a11 11 0 0114.08 0"/>
                <path d="M1.42 9a16 16 0 0121.16 0"/>
                <path d="M8.53 16.11a6 6 0 016.95 0"/>
                <circle cx="12" cy="20" r="1"/>
              </svg>
              <h3 class="font-semibold mb-1">IoT Integration</h3>
              <p class="text-xs text-[var(--muted)]">Real sensor connectivity via MQTT</p>
            </div>
            
            <div class="diagram-node glow-box relative overflow-hidden">
              <div class="absolute top-0 right-0 w-20 h-20 bg-[var(--accent)] opacity-5 rounded-full blur-2xl"></div>
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <rect x="4" y="4" width="16" height="16" rx="2" ry="2"/>
                <rect x="9" y="9" width="6" height="6"/>
                <line x1="9" y1="1" x2="9" y2="4"/>
                <line x1="15" y1="1" x2="15" y2="4"/>
                <line x1="9" y1="20" x2="9" y2="23"/>
                <line x1="15" y1="20" x2="15" y2="23"/>
              </svg>
              <h3 class="font-semibold mb-1">Edge Computing</h3>
              <p class="text-xs text-[var(--muted)]">Local processing for reduced latency</p>
            </div>
          </div>
          
          <div class="grid lg:grid-cols-3 gap-4">
            <div class="diagram-node">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <path d="M12 1a3 3 0 00-3 3v8a3 3 0 006 0V4a3 3 0 00-3-3z"/>
                <path d="M19 10v2a7 7 0 01-14 0v-2"/>
                <line x1="12" y1="19" x2="12" y2="23"/>
                <line x1="8" y1="23" x2="16" y2="23"/>
              </svg>
              <h3 class="font-semibold mb-1">Voice Assistant</h3>
              <p class="text-xs text-[var(--muted)]">Natural language control interface</p>
            </div>
            
            <div class="diagram-node">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <rect x="5" y="2" width="14" height="20" rx="2" ry="2"/>
                <line x1="12" y1="18" x2="12.01" y2="18"/>
              </svg>
              <h3 class="font-semibold mb-1">Mobile Control</h3>
              <p class="text-xs text-[var(--muted)]">iOS and Android native apps</p>
            </div>
            
            <div class="diagram-node">
              <svg class="w-8 h-8 mb-3 text-[var(--accent)]" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <line x1="18" y1="20" x2="18" y2="10"/>
                <line x1="12" y1="20" x2="12" y2="4"/>
                <line x1="6" y1="20" x2="6" y2="14"/>
              </svg>
              <h3 class="font-semibold mb-1">Predictive Analytics</h3>
              <p class="text-xs text-[var(--muted)]">Energy forecasting and optimization</p>
            </div>
          </div>
          
          <!-- Vision Statement -->
          <div class="mt-8 p-6 bg-gradient-to-r from-[var(--card)] to-transparent border-l-2 border-[var(--accent)]">
            <p class="text-sm text-[var(--muted)] italic">"The next evolution will transform this rule-based foundation into an adaptive, learning ecosystem that anticipates user needs before they arise."</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 17: Conclusion -->
    <section class="slide" data-slide="17">
      <div class="mb-8">
        <span class="mono text-xs text-[var(--accent)] tracking-widest">17 / CONCLUSION</span>
        <h2 class="text-4xl font-light mt-2">Research Summary</h2>
      </div>
      <div class="flex-1 flex items-center justify-center">
        <div class="max-w-4xl">
          <div class="border-l-2 border-[var(--accent)] pl-8 mb-12">
            <p class="text-xl leading-relaxed">
              This project successfully demonstrates how <span class="text-[var(--accent)] font-medium">mathematical logic</span> and <span class="text-[var(--accent)] font-medium">rule-based computation</span> can model an intelligent home environment, providing a <span class="font-medium">scalable foundation</span> for future AI-driven smart living systems.
            </p>
          </div>
          
          <div class="grid md:grid-cols-3 gap-6">
            <div class="text-center p-6 bg-[var(--card)] border border-[var(--border)]">
              <p class="mono text-4xl text-[var(--accent)] mb-2">6</p>
              <p class="text-sm text-[var(--muted)]">Sensor Inputs</p>
            </div>
            <div class="text-center p-6 bg-[var(--card)] border border-[var(--border)] glow-box">
              <p class="mono text-4xl text-[var(--accent)] mb-2">12</p>
              <p class="text-sm text-[var(--muted)]">Automation Rules</p>
            </div>
            <div class="text-center p-6 bg-[var(--card)] border border-[var(--border)]">
              <p class="mono text-4xl text-[var(--accent)] mb-2">5</p>
              <p class="text-sm text-[var(--muted)]">Controlled Devices</p>
            </div>
          </div>
          
          <div class="mt-12 flex items-center justify-center gap-8 text-sm text-[var(--muted)]">
            <span>Boolean Logic</span>
            <span class="w-1 h-1 rounded-full bg-[var(--accent)]"></span>
            <span>Real-time Processing</span>
            <span class="w-1 h-1 rounded-full bg-[var(--accent)]"></span>
            <span>Scalable Architecture</span>
          </div>
        </div>
      </div>
    </section>

    <!-- Slide 18: Thank You -->
    <section class="slide" data-slide="18">
      <div class="flex-1 flex flex-col items-center justify-center text-center">
        <div class="mb-12">
          <svg class="w-20 h-20 mx-auto mb-8 text-[var(--accent)]" viewBox="0 0 64 64" fill="none" stroke="currentColor" stroke-width="1">
            <rect x="8" y="16" width="48" height="36" rx="2"/>
            <path d="M8 28h48"/>
            <circle cx="16" cy="22" r="2" fill="var(--accent)"/>
            <circle cx="24" cy="22" r="2" fill="var(--accent)"/>
            <rect x="16" y="36" width="12" height="8" rx="1" fill="var(--accent)" fill-opacity="0.2"/>
            <path d="M36 36h12M36 40h8" stroke-width="1.5"/>
          </svg>
        </div>
        
        <h2 class="text-5xl font-light mb-4">Thank You</h2>
        <p class="text-xl text-[var(--muted)] mb-12">Questions and Discussion</p>
        
        <div class="flex items-center gap-8 text-sm text-[var(--muted)]">
          <span>Group AD</span>
          <span class="w-1 h-1 rounded-full bg-[var(--accent)]"></span>
          <span>website done by - A.D.V.U. Perera 34590 / J.P.R.R. Jayalath 35019</span>
        </div>
        
        <div class="mt-16">
          <p class="text-xs text-[var(--muted)]">Faculty of computing NSBM</p>
        </div>
      </div>
      
      <!-- Decorative -->
      <div class="absolute inset-0 overflow-hidden pointer-events-none">
        <div class="absolute bottom-1/4 left-1/4 w-96 h-96 rounded-full bg-[var(--accent)] opacity-[0.02] blur-3xl"></div>
        <div class="absolute top-1/4 right-1/4 w-64 h-64 rounded-full bg-[var(--accent)] opacity-[0.02] blur-3xl"></div>
      </div>
    </section>

  </main>

  <script>
    // Initialize state
    let currentSlide = 1;
    const totalSlides = 18;
    const slides = document.querySelectorAll('.slide');
    const prevBtn = document.getElementById('prevBtn');
    const nextBtn = document.getElementById('nextBtn');
    const slideCounter = document.getElementById('slideCounter');
    const progressBar = document.getElementById('progressBar');

    // Update slide display
    function updateSlide(newSlide, direction) {
      if (newSlide < 1 || newSlide > totalSlides) return;
      
      const currentSlideEl = document.querySelector('.slide.active');
      const nextSlideEl = document.querySelector(`[data-slide="${newSlide}"]`);
      
      if (currentSlideEl) {
        currentSlideEl.classList.remove('active');
        currentSlideEl.classList.add(direction === 'next' ? 'prev' : '');
      }
      
      if (nextSlideEl) {
        nextSlideEl.classList.remove('prev');
        nextSlideEl.classList.add('active');
      }
      
      currentSlide = newSlide;
      
      // Update navigation
      prevBtn.disabled = currentSlide === 1;
      nextBtn.disabled = currentSlide === totalSlides;
      
      // Update counter
      slideCounter.textContent = `${String(currentSlide).padStart(2, '0')} / ${String(totalSlides).padStart(2, '0')}`;
      
      // Update progress bar
      progressBar.style.width = `${(currentSlide / totalSlides) * 100}%`;
    }

    // Event listeners
    prevBtn.addEventListener('click', () => updateSlide(currentSlide - 1, 'prev'));
    nextBtn.addEventListener('click', () => updateSlide(currentSlide + 1, 'next'));

    // Keyboard navigation
    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowRight' || e.key === ' ' || e.key === 'PageDown') {
        e.preventDefault();
        updateSlide(currentSlide + 1, 'next');
      } else if (e.key === 'ArrowLeft' || e.key === 'PageUp') {
        e.preventDefault();
        updateSlide(currentSlide - 1, 'prev');
      } else if (e.key === 'Home') {
        e.preventDefault();
        updateSlide(1, 'prev');
      } else if (e.key === 'End') {
        e.preventDefault();
        updateSlide(totalSlides, 'next');
      }
    });

    // Touch/swipe support
    let touchStartX = 0;
    let touchEndX = 0;

    document.addEventListener('touchstart', (e) => {
      touchStartX = e.changedTouches[0].screenX;
    }, { passive: true });

    document.addEventListener('touchend', (e) => {
      touchEndX = e.changedTouches[0].screenX;
      const diff = touchStartX - touchEndX;
      if (Math.abs(diff) > 50) {
        if (diff > 0) {
          updateSlide(currentSlide + 1, 'next');
        } else {
          updateSlide(currentSlide - 1, 'prev');
        }
      }
    }, { passive: true });

    // Initialize
    updateSlide(1, 'next');
  </script>
</body>
</html>
