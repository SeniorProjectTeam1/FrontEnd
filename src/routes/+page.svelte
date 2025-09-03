<script>
  let topic = $state("Campus productivity for engineering students");
  let category = $state("EdTech");
  let innovation = $state(50);

  let minTech = $state(6);
  let minResources = $state(6);
  let minTimelineSpeed = $state(6);
  let maxPrice = $state(6);
  let minMarket = $state(7);

  let loading = $state(false);
  let results = $state(null);
  let selected = $state(null);
  let activeTab = $state("ideas");
  let error = $state(null);

  const categories = ["EdTech","IoT/Embedded","Health","FinTech","Sustainability","Campus Life"];

  const clamp = (n,min,max)=>Math.max(min,Math.min(max,n));
  const randInt = (a,b)=>Math.floor(Math.random()*(b-a+1))+a;

  function makeIdea(title, radical=false){
    const tech       = clamp(randInt(5,10) - (radical?1:0),1,10);
    const resources  = randInt(5,10);
    const timeline   = randInt(5,10);
    const price      = randInt(3,9) + (radical?1:0);
    const mBase      = 6 + (radical?1:0) + (Math.random()>.5?1:0);
    const market_rating = clamp(Math.round(mBase + (Math.random()>.7?1:0)), 1, 10);

    return {
      title,
      elevator_pitch: radical
        ? "Bold concept that challenges current tools; higher risk, higher potential."
        : "Practical idea aimed at quick value with low risk.",
      innovation_level: radical ? "radical" : "moderate",
      feasibility: { tech, resources, timeline, price_rating: price, notes: "6–10 weeks, 2–3 devs." },
      market: {
        pain_point: "Students lack structured focus & task visibility across courses.",
        target_users: "College students; secondary: advisors/TAs",
        competitors: ["Notion","Todoist","Forest","Google Tasks"].slice(0, randInt(1,3)),
        moat: radical? "Unique data/insights; step-change UX" : "Tight integrations & habit loops",
        risks: ["Crowded space","Retention post-semester","Data access"].slice(0, randInt(1,3))
      },
      market_rating,
      mvp_scope: ["Auth + profiles","Idea wizard","Scorecards","Export one-pager"]
    };
  }

  function generateMockIdeas() {
    const rad = innovation >= 66;
    const titles = [
      "Course Compass: syllabus-to-planner",
      "Focus Pods: Pomodoro + peer matcher",
      "Smart Office-Hours balancer",
      "IoT occupancy heatmap for labs",
      "Grade-to-Task checklist bridge",
      "Study Streaks: habit loops & rewards",
      "TA Queue: live triage & FAQs",
      "Auto-Flashcards from lecture PDFs",
      "Skill Map: tasks → competencies",
      "Group-Work Radar: teammate reliability"
    ];
    return { ideas: titles.map((t,i)=>makeIdea(t, rad && i%3===0)) };
  }

  const marketScore = (i) => i.market_rating;
  function passesFineGrained(i){
    const f=i.feasibility, m=marketScore(i);
    const okTech = f.tech >= minTech;
    const okRes  = f.resources >= minResources;
    const okTime = f.timeline >= minTimelineSpeed;
    const okPrice= f.price_rating <= maxPrice;
    const okMkt  = m >= minMarket;
    return { okTech, okRes, okTime, okPrice, okMkt, all: okTech && okRes && okTime && okPrice && okMkt, m };
  }
  function filteredIdeas(){
    if(!results) return [];
    return results.ideas.filter(i => passesFineGrained(i).all);
  }

  async function onGenerate(){
    loading = true; error = null; results = null; selected=null; activeTab="ideas";
    await new Promise(r=>setTimeout(r, 1200));
    results = generateMockIdeas();
    loading = false;
  }
</script>

<style>
  :root{ --panel:#0b1222; --card:#0f1b34; --muted:#9fb3d1; }
  body{ margin:0; font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,Arial; background:#0b1222; color:#e6edf6; }
  .wrap{ display:grid; grid-template-columns:360px minmax(0,1fr); gap:16px; min-height:100vh; padding:16px; }
  .panel{ background:var(--panel); border:1px solid rgba(255,255,255,.06); border-radius:16px; padding:16px; }
  .right{ display:flex; flex-direction:column; gap:12px; }
  h1{ font-size:20px; margin:0 0 12px; color:#ffffff; }
  label{ font-size:12px; color:var(--muted); display:block; margin-bottom:6px; }

  input, select, textarea{
    width:100%;
    max-width:100%;
    background:#0f1b34;
    color:#e6edf6;
    border:1px solid rgba(255,255,255,.08);
    border-radius:10px;
    padding:10px 12px;
    box-sizing:border-box;
  }

  /* Ensure range tracks reach the true edges (no inner padding/margins) */
  input[type="range"]{
    width:100%;
    max-width:100%;
    padding:0;
    margin:0;
    background:#0f1b34;
    border:1px solid rgba(255,255,255,.08);
    border-radius:10px;
    height:34px;
    box-sizing:border-box;
  }

  textarea{ min-height:78px; resize:vertical; }
  .row{ display:flex; gap:10px; }
  .btn{ background:linear-gradient(180deg,#2563eb,#1d4ed8); border:none; color:#fff; padding:10px 14px; border-radius:10px; cursor:pointer; font-weight:600; }
  .btn.secondary{ background:#0f1b34; border:1px solid rgba(255,255,255,.12); }
  .tabs{ display:flex; gap:8px; }
  .tab{ padding:8px 12px; border-radius:999px; cursor:pointer; font-weight:600; color:#c8d6ee; border:1px solid transparent; }
  .tab.active{ background:#1e293b; border-color:#2b3a57; }
  .grid{ display:grid; grid-template-columns:repeat(auto-fill, minmax(280px,1fr)); gap:12px; }
  .card{ background:var(--card); border:1px solid rgba(255,255,255,.06); border-radius:14px; padding:14px; }
  .title{ font-weight:700; margin-bottom:6px; }
  .pitch{ font-size:13px; color:#b7c7e6; }
  .badges{ display:flex; gap:8px; margin-top:8px; flex-wrap:wrap; }
  .badge{ font-size:11px; padding:4px 8px; border-radius:999px; border:1px solid rgba(255,255,255,.12); color:#c7d4ee; }
  .ok{ background:#083b2d; color:#7ee7c6; border-color:#175a47; }
  .no{ background:#3b0a0a; color:#ffb4b4; border-color:#7a2c2c; }
  .mini{ font-size:12px; color:#a9b8d7; }
  .hr{ height:1px; background:rgba(255,255,255,.06); margin:8px 0; }

  .slider-group{
    padding:10px 12px;
    border:1px solid rgba(255,255,255,.10);
    border-radius:10px;
    margin-bottom:10px;
    background:rgba(255,255,255,.02);
    overflow:hidden;
  }

  .two-col{ display:grid; grid-template-columns:minmax(0,1fr) minmax(0,1fr); gap:10px; }

  .content-panel{ position:relative; min-height:320px; }
  .overlay{
    position:absolute; inset:0; display:flex; flex-direction:column;
    align-items:center; justify-content:center;
    background:rgba(2,6,23,.72); backdrop-filter:saturate(110%) blur(2px);
    z-index:10; text-align:center; padding:24px;
  }
  .spinner{
    width:48px; height:48px; border-radius:50%;
    border:3px solid rgba(255,255,255,.2); border-top-color:#fff;
    animation:spin 0.9s linear infinite; margin-bottom:12px;
  }
  @keyframes spin{ to{ transform: rotate(360deg) } }

  @media (max-width:1024px){
    .wrap{ grid-template-columns:1fr; }
    .two-col{ grid-template-columns:1fr; }
  }
</style>

<div class="wrap">
  <aside class="panel">
    <h1>AI Ideation (Front-end Mock up)</h1>

    <div class="slider-group">
      <label>Topic / Context</label>
      <textarea bind:value={topic} placeholder="What domain, constraints, audience?"></textarea>
    </div>

    <div class="slider-group">
      <label>Category</label>
      <select bind:value={category}>
        {#each categories as c}<option>{c}</option>{/each}
      </select>
    </div>

    <div class="slider-group">
      <label>Innovation</label>
      <input type="range" min="0" max="100" bind:value={innovation} />
      <div class="mini">{innovation<=33?"Practical":innovation<=66?"Balanced":"Radical"}</div>
    </div>

    <div class="two-col">
      <div class="slider-group">
        <label>Tech (min)</label>
        <input type="range" min="1" max="10" step="1" bind:value={minTech} />
        <div class="mini">{minTech}/10</div>
      </div>
      <div class="slider-group">
        <label>Resources (min)</label>
        <input type="range" min="1" max="10" step="1" bind:value={minResources} />
        <div class="mini">{minResources}/10</div>
      </div>
    </div>

    <div class="two-col">
      <div class="slider-group">
        <label>Timeline Speed (min)</label>
        <input type="range" min="1" max="10" step="1" bind:value={minTimelineSpeed} />
        <div class="mini">{minTimelineSpeed}/10 (higher = faster)</div>
      </div>
      <div class="slider-group">
        <label>Price (max)</label>
        <input type="range" min="1" max="10" step="1" bind:value={maxPrice} />
        <div class="mini">{maxPrice}/10 (lower = cheaper)</div>
      </div>
    </div>

    <div class="slider-group">
      <label>Market Strength (min)</label>
      <input type="range" min="1" max="10" step="1" bind:value={minMarket} />
      <div class="mini">{minMarket}/10</div>
    </div>

    <div class="row" style="margin-top:12px">
      <button class="btn" on:click={onGenerate} disabled={loading}>
        {loading ? "Generating…" : "Generate 10 ideas"}
      </button>
    </div>

    <div class="hr"></div>
    <div class="mini">Filters apply live to the 10 generated ideas.</div>
  </aside>

  <section class="right">
    <div class="panel" style="padding:10px">
      <div class="tabs">
        <div class="tab {activeTab==='ideas'?'active':''}" on:click={()=>activeTab='ideas'}>Ideas</div>
        <div class="tab {activeTab==='compare'?'active':''}" on:click={()=>activeTab='compare'}>Scores</div>
        <div class="tab {activeTab==='details'?'active':''}" on:click={()=>activeTab='details'}>One-Pager</div>
      </div>
    </div>

    <div class="panel content-panel" aria-busy={loading}>
      {#if loading}
        <div class="overlay" role="status" aria-live="polite">
          <div class="spinner"></div>
          <div class="mini">Generating 10 ideas…</div>
        </div>
      {/if}

      {#if error}
        <div class="card">Error: {error}</div>
      {:else if !results}
        <div class="card">Click “Generate 10 ideas”.</div>
      {:else}
        {#if activeTab === 'ideas'}
          <div class="mini" style="margin-bottom:8px">
            Showing {filteredIdeas().length} / 10 ideas that match your filters.
          </div>
          <div class="grid">
            {#each filteredIdeas() as it}
              {#key it.title}
              <div class="card">
                <div class="title">{it.title}</div>
                <div class="pitch">{it.elevator_pitch}</div>
                <div class="badges" style="margin-top:8px">
                  <span class="badge">Innovation: {it.innovation_level}</span>
                  <span class="badge">Tech {it.feasibility.tech}/10</span>
                  <span class="badge">Res {it.feasibility.resources}/10</span>
                  <span class="badge">Speed {it.feasibility.timeline}/10</span>
                  <span class="badge">Price {it.feasibility.price_rating}/10</span>
                  <span class="badge">Market {marketScore(it)}/10</span>
                </div>
                <div class="hr"></div>
                <div class="badges">
                  {#if passesFineGrained(it).okTech}<span class="badge ok">Tech ✓</span>{:else}<span class="badge no">Tech ✕</span>{/if}
                  {#if passesFineGrained(it).okRes}<span class="badge ok">Res ✓</span>{:else}<span class="badge no">Res ✕</span>{/if}
                  {#if passesFineGrained(it).okTime}<span class="badge ok">Speed ✓</span>{:else}<span class="badge no">Speed ✕</span>{/if}
                  {#if passesFineGrained(it).okPrice}<span class="badge ok">Price ✓</span>{:else}<span class="badge no">Price ✕</span>{/if}
                  {#if passesFineGrained(it).okMkt}<span class="badge ok">Market ✓</span>{:else}<span class="badge no">Market ✕</span>{/if}
                </div>
                <div class="mini" style="margin-top:6px"><b>MVP:</b> {it.mvp_scope.join(", ")}</div>
                <div class="row" style="margin-top:8px">
                  <button class="btn secondary" on:click={()=>{selected=it; activeTab='details';}}>Open details</button>
                </div>
              </div>
              {/key}
            {/each}
          </div>
        {:else if activeTab === 'compare'}
          <div class="grid">
            {#each (results?.ideas ?? []) as it}
              <div class="card">
                <div class="title">{it.title}</div>
                <div class="badges">
                  <span class="badge">Tech {it.feasibility.tech}/10</span>
                  <span class="badge">Resources {it.feasibility.resources}/10</span>
                  <span class="badge">Speed {it.feasibility.timeline}/10</span>
                  <span class="badge">Price {it.feasibility.price_rating}/10</span>
                  <span class="badge">Market {marketScore(it)}/10</span>
                </div>
              </div>
            {/each}
          </div>
        {:else}
          {#if selected}
            <div class="card">
              <div class="title">{selected.title}</div>
              <div class="pitch">{selected.elevator_pitch}</div>
              <div class="hr"></div>
              <div class="mini"><b>Problem/Users:</b> {selected.market.pain_point} — {selected.market.target_users}</div>
              <div class="mini"><b>Competitors:</b> {selected.market.competitors.join(", ")}</div>
              <div class="mini"><b>Moat:</b> {selected.market.moat}</div>
              <div class="mini"><b>Risks:</b> {selected.market.risks.join(", ")}</div>
              <div class="mini"><b>MVP:</b> {selected.mvp_scope.join(", ")}</div>
              <div class="badges" style="margin-top:10px">
                <span class="badge">Tech {selected.feasibility.tech}/10</span>
                <span class="badge">Res {selected.feasibility.resources}/10</span>
                <span class="badge">Speed {selected.feasibility.timeline}/10</span>
                <span class="badge">Price {selected.feasibility.price_rating}/10</span>
                <span class="badge">Market {marketScore(selected)}/10</span>
              </div>
            </div>
          {:else}
            <div class="card">Select a card → “Open details”.</div>
          {/if}
        {/if}
      {/if}
    </div>
  </section>
</div>
