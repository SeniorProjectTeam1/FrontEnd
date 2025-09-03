<script>
  // ---------- STATE ----------
  let topic = $state("Campus productivity for engineering students");
  let category = $state("EdTech");

  let innovation = $state(50); // 0..100

  // Fine-grained filters (1..10 scales)
  let minTech = $state(6);
  let minResources = $state(6);
  let minTimelineSpeed = $state(6); // higher = faster
  let maxPrice = $state(6);         // lower = cheaper
  let minMarket = $state(7);

  let loading = $state(false);
  let results = $state(null);
  let selected = $state(null);
  let activeTab = $state("ideas");
  let error = $state(null);

  const categories = ["EdTech","IoT/Embedded","Health","FinTech","Sustainability","Campus Life"];

  // ---------- MOCK IDEA FACTORY ----------
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
      feasibility: { tech, resources, timeline, price_rating: price },
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

  // ---------- SCORING & FILTERS ----------
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

  // ---------- ACTIONS ----------
  async function onGenerate(){
    loading = true; error = null; results = null; selected=null; activeTab="ideas";
    await new Promise(r=>setTimeout(r, 1000));
    results = generateMockIdeas();
    loading = false;
  }
</script>

<style>
  :root {
    --panel:#0b1222;
    --card:#0f1b34;
    --muted:#9fb3d1;
    --text:#9fb3d1; /* unified text color */
    --background:#0b1222;
  }
  body {
    margin:0;
    font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,Arial;
    background:var(--background);
    color:var(--text);
  }
  .wrap {
    display:grid;
    grid-template-columns:360px 1fr;
    gap:16px;
    min-height:100vh;
    padding:16px;
    background:var(--background);
  }
  .panel {
    background:var(--panel);
    border:1px solid rgba(255,255,255,.06);
    border-radius:16px;
    padding:16px;
  }
  .right {
    display:flex;
    flex-direction:column;
    gap:12px;
  }
  h1 { font-size:20px; margin:0 0 12px; color:var(--text); }
  label { font-size:12px; color:var(--text); display:block; margin-bottom:6px; }
  input, select, textarea {
    width:100%;
    background:#0f1b34;
    color:var(--text);
    border:1px solid rgba(255,255,255,.08);
    border-radius:10px;
    padding:10px 12px;
    box-sizing:border-box;
  }
  textarea { min-height:78px; resize:vertical; }
  .row { display:flex; gap:10px; }
  .btn {
    background:linear-gradient(180deg,#2563eb,#1d4ed8);
    border:none;
    color:#fff;
    padding:10px 14px;
    border-radius:10px;
    cursor:pointer;
    font-weight:600;
  }
  .tabs { display:flex; gap:8px; }
  .tab {
    padding:8px 12px;
    border-radius:999px;
    cursor:pointer;
    font-weight:600;
    color:var(--text);
    border:1px solid transparent;
  }
  .tab.active { background:#1e293b; border-color:#2b3a57; }
  .grid { display:grid; grid-template-columns:repeat(auto-fill, minmax(280px,1fr)); gap:12px; }
  .card {
    background:var(--card);
    border:1px solid rgba(255,255,255,.06);
    border-radius:14px;
    padding:14px;
  }
  .title { font-weight:700; margin-bottom:6px; color:var(--text); }
  .pitch { font-size:13px; color:var(--muted); }
  .badge {
    font-size:11px; padding:4px 8px; border-radius:999px;
    border:1px solid rgba(255,255,255,.12); color:var(--text);
  }
  .slider-group {
    padding:10px 12px;
    border:1px solid rgba(255,255,255,.10);
    border-radius:10px;
    margin-bottom:10px;
    background:rgba(255,255,255,.02);
    width:100%;
    box-sizing:border-box;
  }
  .two-col { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
  @media (max-width:1024px){
    .wrap{ grid-template-columns:1fr; }
    .two-col{ grid-template-columns:1fr; }
  }
</style>

<div class="wrap">
  <!-- LEFT: Controls -->
  <aside class="panel">
    <h1>AI Ideation (Front-end mock)</h1>

    <div class="slider-group">
      <label>Topic / Context</label>
      <textarea bind:value={topic}></textarea>
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
        <input type="range" min="1" max="10" bind:value={minTech} />
        <div class="mini">{minTech}/10</div>
      </div>
      <div class="slider-group">
        <label>Resources (min)</label>
        <input type="range" min="1" max="10" bind:value={minResources} />
        <div class="mini">{minResources}/10</div>
      </div>
    </div>

    <div class="two-col">
      <div class="slider-group">
        <label>Timeline Speed (min)</label>
        <input type="range" min="1" max="10" bind:value={minTimelineSpeed} />
        <div class="mini">{minTimelineSpeed}/10 (higher = faster)</div>
      </div>
      <div class="slider-group">
        <label>Price (max)</label>
        <input type="range" min="1" max="10" bind:value={maxPrice} />
        <div class="mini">{maxPrice}/10 (lower = cheaper)</div>
      </div>
    </div>

    <div class="slider-group">
      <label>Market Strength (min)</label>
      <input type="range" min="1" max="10" bind:value={minMarket} />
      <div class="mini">{minMarket}/10</div>
    </div>

    <div class="row" style="margin-top:12px">
      <button class="btn" on:click={onGenerate} disabled={loading}>
        {loading ? "Loading…" : "Generate 10 ideas"}
      </button>
    </div>
  </aside>

  <!-- RIGHT -->
  <section class="right">
    <div class="panel" style="min-height:320px">
      {#if error}
        <div class="card">Error: {error}</div>
      {:else if loading}
        <div class="card">Loading ideas… Please wait.</div>
      {:else if !results}
        <div class="card">Click “Generate 10 ideas”.</div>
      {:else}
        <div class="grid">
          {#each filteredIdeas() as it}
            <div class="card">
              <div class="title">{it.title}</div>
              <div class="pitch">{it.elevator_pitch}</div>
              <div class="mini">Tech {it.feasibility.tech}/10 · Res {it.feasibility.resources}/10 · Speed {it.feasibility.timeline}/10 · Price {it.feasibility.price_rating}/10 · Market {marketScore(it)}/10</div>
            </div>
          {/each}
        </div>
      {/if}
    </div>
  </section>
</div>
