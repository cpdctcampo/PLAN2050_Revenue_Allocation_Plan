<b>Process Outline: Prioritized Project Funding Allocation</b>

<br><br>

<b>1. Data Preparation</b>

<ul>
    <li><b>Load input files</b>
        <ul>
            <li>clrp_rev_cvta.csv → annual revenue budgets by source (including CVTA)</li>
            <li>clrp_project_chest.csv → project list with costs, B/C Rank, county, and primary funding source</li>
        </ul>
    </li>

    <li><b>Clean revenue data (df1)</b>
        <ul>
            <li>Remove special dash characters, convert values to numeric format, and set Years as the index.</li>
            <li>Define Base Year = 2033 and Escalation Rate = 3%.</li>
            <li>Apply a 90% budget cap so only 90% of each year's revenue is available for programming.</li>
        </ul>
    </li>

    <li><b>Prepare project data (df2)</b>
        <ul>
            <li>Validate numeric columns and remove projects with missing primary funding sources or zero cost.</li>
            <li>Initialize allocation columns for each year (primary source and CVTA).</li>
            <li>Sort projects by B/C Rank (ascending; lower rank = higher priority).</li>
            <li>Track remaining cost (Base Year $M), fully funded status, and allocation comments.</li>
        </ul>
    </li>
</ul>

<b>2. Step 1 – CVTA Allocation (Chesterfield Only, Front-Loaded)</b>

<ul>
    <li><b>Target:</b> Chesterfield County projects only.</li>

    <li><b>Method:</b> Front-loaded allocation using the earliest available years.
        <ul>
            <li>Allocate projects in B/C Rank priority order.</li>
            <li>Use annual CVTA revenues constrained to the 90% cap.</li>
            <li>Allocate as much funding as possible each year up to the remaining project cost or annual funding cap.</li>
            <li>Convert between Base Year and Year-of-Expenditure (YOE) dollars using the escalation formula.</li>
        </ul>
    </li>

    <li><b>Output:</b>
        <ul>
            <li><code>cvta_yyyy</code> columns store annual YOE allocations.</li>
            <li><code>CVTA_Allocated_Base</code> and <code>CVTA_Allocated_YOE</code> track project-level totals.</li>
            <li>Remaining costs are reduced and fully funded projects are flagged and removed from subsequent allocation steps.</li>
        </ul>
    </li>
</ul>

<b>3. Step 2 – Allocate Other Revenue Sources (Non-CVTA)</b>

<ul>
    <li><b>Available Sources:</b> All revenue sources in df1 except CVTA (e.g., federal, state, and local).</li>

    <li><b>Eligible Projects:</b> All projects not fully funded during the CVTA allocation step.</li>

    <li><b>Method:</b> Front-loaded allocation by Primary Funding Source.
        <ul>
            <li>Process remaining projects in B/C Rank order.</li>
            <li>Use the project's designated primary funding source.</li>
            <li>Allocate funding in the earliest possible years while respecting annual caps and remaining project costs.</li>
        </ul>
    </li>

    <li><b>Output:</b>
        <ul>
            <li>Primary-source allocations are stored in annual year columns.</li>
            <li>Remaining project costs are updated.</li>
            <li>Projects reaching full funding are flagged accordingly.</li>
        </ul>
    </li>
</ul>

<b>4. Budget Sweep (Residual Allocation)</b>

<ul>
    <li><b>Purpose:</b> Use any remaining funding capacity to maximize project funding.</li>

    <li><b>Process:</b>
        <ul>
            <li>Review all revenue sources with remaining annual balances.</li>
            <li>Allocate available funds to eligible unfunded projects.</li>
            <li>CVTA funds remain restricted to Chesterfield projects.</li>
            <li>Other revenue sources can only fund projects matching their designated primary source.</li>
            <li>Continue applying front-loaded allocations until budgets are exhausted or projects are fully funded.</li>
        </ul>
    </li>

    <li><b>Result:</b> Achieves near-100% utilization of the programmed 90% revenue cap.</li>
</ul>

<b>5. Compute Final Metrics</b>

<ul>
    <li><b>Allocated Totals</b>
        <ul>
            <li><b>YOE Dollars:</b> Sum of primary funding allocations and CVTA allocations.</li>
            <li><b>Base-Year Dollars:</b> Discounted value of all allocations using the escalation rate.</li>
            <li><b>Remaining Balance:</b> Original project cost minus total allocated Base-Year funding.</li>
        </ul>
    </li>

    <li><b>Anticipated Opening Year</b>
        <ul>
            <li>Defined as the latest year receiving any funding (primary or CVTA).</li>
            <li>Projects receiving no funding remain blank.</li>
        </ul>
    </li>
</ul>

<b>6. Build Summary Tables</b>

<ul>
    <li><b>df3 – Annual Programmed vs. Available Revenue</b>
        <ul>
            <li>Original annual revenue (100%).</li>
            <li>Available revenue after applying the 90% cap.</li>
            <li>Total programmed funding (YOE dollars).</li>
            <li>Utilization percentage = Programmed Funding ÷ Available Revenue.</li>
            <li>Includes a TOTAL row summarizing all years.</li>
        </ul>
    </li>

    <li><b>df2 (Augmented Project Table)</b>
        <ul>
            <li>Annual allocation details.</li>
            <li>CVTA funding allocations.</li>
            <li>Total funded amounts.</li>
            <li>Remaining balances.</li>
            <li>Anticipated Opening Year.</li>
            <li>Comments and funding status indicators.</li>
        </ul>
    </li>
</ul>

<b>7. Output</b>

<ul>
    <li><b>Export to CSV</b>
        <ul>
            <li><b>df2_allocated_cvta_priority.csv</b> – Project allocations, funding status, and Anticipated Opening Year.</li>
            <li><b>df3_budget_comparison_cvta.csv</b> – Annual revenue availability, programming, and utilization.</li>
        </ul>
    </li>

    <li><b>Console Summary</b>
        <ul>
            <li>Total utilization by revenue source.</li>
            <li>Counts of funded and fully funded projects.</li>
            <li>Count of Chesterfield projects funded with CVTA revenue.</li>
            <li>Top 10 projects by B/C Rank with funding status and Anticipated Opening Year.</li>
        </ul>
    </li>
</ul>

<hr>

<b>Key Design Choices</b>

<ul>
    <li><b>Front-Loaded Allocation:</b> Projects consume the earliest available revenues, minimizing gaps between funding initiation and project completion.</li>

    <li><b>CVTA Reserved for Chesterfield:</b> CVTA funding is restricted to Chesterfield County projects and allocated before any other funding source.</li>

    <li><b>90% Revenue Cap:</b> Only 90% of annual revenues are available for project programming to maintain fiscal constraint.</li>

    <li><b>Budget Sweep:</b> Remaining revenues are redistributed to maximize funding utilization.</li>

    <li><b>Anticipated Opening Year:</b> Defined as the last year in which a project receives funding, providing an estimated completion timeline.</li>
</ul>
