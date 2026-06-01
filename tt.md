<b>Process Outline: Prioritized Project Funding Allocation</b>

<br><br>

<b>1. Data Preparation</b>

<ul>
  <li><b>Load input files</b>
    <ul>
      <li>clrp_rev_cvta.csv → annual revenue budgets by source (incl. CVTA)</li>
      <li>clrp_project_chest.csv → project list with costs, B/C Rank, county, primary funding source</li>
    </ul>
  </li>

  <li><b>Clean revenue data (df1)</b>
    <ul>
      <li>Remove special dash characters, convert to numeric, set Years as index</li>
      <li>Define base year = 2033, escalation rate = 3%</li>
      <li>Budget cap = 90% of each year’s revenue (only 90% is made available for programming)</li>
    </ul>
  </li>

  <li><b>Prepare project data (df2)</b>
    <ul>
      <li>Validate numeric columns, drop rows with missing primary funding source or zero cost</li>
      <li>Initialize allocation columns for each year (primary source + CVTA)</li>
      <li>Sort projects by B/C Rank (ascending, lower rank = higher priority)</li>
      <li>Track remaining cost (base $M), fully funded flag, comment</li>
    </ul>
  </li>
</ul>

<b>2. Step 1 – CVTA Allocation (Chesterfield only, front loaded)</b>

<ul>
  <li><b>Target:</b> Chesterfield County projects only</li>

  <li><b>Method:</b> Front loaded – allocate funds in earliest possible years
    <ul>
      <li>For each project in priority order (B/C Rank)</li>
      <li>Use 90% CVTA annual caps (converted to YOE dollars)</li>
      <li>Each year: allocate as much as possible, up to remaining cost or annual cap</li>
      <li>Convert between base and YOE using escalation formula</li>
    </ul>
  </li>

  <li><b>Output:</b>
    <ul>
      <li>cvta_yyyy columns store YOE allocations</li>
      <li>CVTA_Allocated_Base, CVTA_Allocated_YOE totals per project</li>
      <li>Remaining cost reduced; fully funded projects marked and removed from later steps</li>
    </ul>
  </li>
</ul>

<b>3. Step 2 – Allocate Other Revenue Sources (non CVTA)</b>

<ul>
  <li><b>Available sources:</b> all columns in df1 except CVTA (e.g., state, federal, local)</li>

  <li><b>Eligible projects:</b> all projects not fully funded by CVTA (any county)</li>

  <li><b>Method:</b> front loaded, one-to-one by Primary Funding Source
    <ul>
      <li>For each remaining project (sorted by B/C Rank)</li>
      <li>Use its designated primary funding source’s 90% annual cap</li>
      <li>Allocate earliest years first, respecting annual cap and remaining cost</li>
    </ul>
  </li>

  <li><b>Output:</b>
    <ul>
      <li>Primary source allocation stored in year columns (no prefix)</li>
      <li>Remaining cost updated; fully funded projects marked</li>
    </ul>
  </li>
</ul>

<b>4. Budget Sweep (Residual Allocation)</b>

<ul>
  <li><b>Purpose:</b> Use any leftover 90% cap funds to fully fund as many remaining projects as possible</li>

  <li><b>Process:</b>
    <ul>
      <li>Iterate over all sources (CVTA + other) that still have positive budget in any year</li>
      <li>For each source, allocate to eligible unfunded projects</li>
      <li>CVTA → only Chesterfield; other → match primary source</li>
      <li>Apply front loaded logic within each source’s remaining annual caps</li>
      <li>Update allocations, remaining cost, and fully funded status</li>
    </ul>
  </li>

  <li><b>Result:</b> Near 100% utilization of the 90% cap for all revenue sources</li>
</ul>

<b>5. Compute Final Metrics</b>

<ul>
  <li><b>Allocated totals</b>
    <ul>
      <li>YOE $ = sum of primary year columns + CVTA year columns</li>
      <li>Base $ = discounted sum of all allocations using escalation rate</li>
      <li>Remaining balance = original total cost − allocated base</li>
    </ul>
  </li>

  <li><b>Anticipated Opening Year</b>
    <ul>
      <li>For each project: the latest year that receives any funding (primary or CVTA)</li>
      <li>None if no funding allocated</li>
    </ul>
  </li>
</ul>

<b>6. Build Summary Tables</b>

<ul>
  <li><b>df3 – Annual Programmed vs. Available</b>
    <ul>
      <li>Original revenue (100%)</li>
      <li>90% cap available for programming</li>
      <li>Total programmed (YOE) from allocations</li>
      <li>Utilization % = programmed / 90% cap</li>
      <li>Includes a TOTAL row summing over all years</li>
    </ul>
  </li>

  <li><b>df2 (augmented) – Project level allocation details</b>
    <ul>
      <li>Adds all allocation columns, totals, opening year, comments, fully funded flag</li>
    </ul>
  </li>
</ul>

<b>7. Output</b>

<ul>
  <li><b>Export to CSV</b>
    <ul>
      <li>df2_allocated_cvta_priority.csv – project allocations with opening year</li>
      <li>df3_budget_comparison_cvta.csv – annual budget utilization</li>
    </ul>
  </li>

  <li><b>Console summary</b>
    <ul>
      <li>Utilization totals by source</li>
      <li>Count of funded / fully funded projects, Chesterfield CVTA funded count</li>
      <li>Top 10 projects by B/C Rank with funding status and opening year</li>
    </ul>
  </li>
</ul>

<hr>

<b>Key design choices</b>

<ul>
  <li><b>Front loaded allocation</b> – each project consumes earliest available funds, minimizing gaps between start and opening year.</li>
  <li><b>CVTA reserved exclusively for Chesterfield</b> – allocated first before other sources.</li>
  <li><b>90% cap</b> – only 90% of each year’s revenue is made available for programming.</li>
  <li><b>Budget sweep</b> – ensures maximum feasible funding within caps.</li>
  <li><b>Anticipated Opening Year</b> – last year with any funding (useful for completion timing).</li>
</ul>
