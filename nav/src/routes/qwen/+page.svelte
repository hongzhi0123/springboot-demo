<script>
  // ----- Configuration -----
  const sources = ['EBA', 'BaFin', 'ACPR', 'DNB'];
  const types = [
    { key: 'all', label: 'All TPPs' },
    { key: 'credit', label: 'Credit Institutes' },
    { key: 'payment', label: 'Payment Institutes' }
  ];

  // ----- State (runes) -----
  let activeSource = $state('EBA');
  let activeType = $state('all');
  let tableData = $state([]);
  let loading = $state(false);
  let error = $state(null);
  let openDropdown = $state(null);   // name of source with open dropdown
  let hoverTimeout = $state(null);

  // ----- Derived values -----
  const typeLabel = $derived(types.find(t => t.key === activeType)?.label ?? activeType);
  const breadcrumb = $derived(`${activeSource} › ${typeLabel}`);

  // ----- Helper functions -----
  function escapeHtml(str) {
    if (!str) return '';
    return str.replace(/[&<>]/g, (m) => {
      if (m === '&') return '&amp;';
      if (m === '<') return '&lt;';
      if (m === '>') return '&gt;';
      return m;
    });
  }

  // ----- Mock data fetch -----
  async function fetchTPPs(source, type) {
    await new Promise(resolve => setTimeout(resolve, 300));
    const mockData = {
      EBA: {
        all: [
          { name: 'EBA Bank One', type: 'Credit', country: 'DE' },
          { name: 'EBA Pay Ltd', type: 'Payment', country: 'FR' },
          { name: 'EBA Credit Corp', type: 'Credit', country: 'IT' }
        ],
        credit: [
          { name: 'EBA Bank One', type: 'Credit', country: 'DE' },
          { name: 'EBA Credit Corp', type: 'Credit', country: 'IT' }
        ],
        payment: [{ name: 'EBA Pay Ltd', type: 'Payment', country: 'FR' }]
      },
      BaFin: {
        all: [
          { name: 'BaFin Bank', type: 'Credit', country: 'DE' },
          { name: 'BaFin Payment', type: 'Payment', country: 'DE' }
        ],
        credit: [{ name: 'BaFin Bank', type: 'Credit', country: 'DE' }],
        payment: [{ name: 'BaFin Payment', type: 'Payment', country: 'DE' }]
      },
      ACPR: {
        all: [
          { name: 'ACPR Bank', type: 'Credit', country: 'FR' },
          { name: 'ACPR Pay', type: 'Payment', country: 'FR' }
        ],
        credit: [{ name: 'ACPR Bank', type: 'Credit', country: 'FR' }],
        payment: [{ name: 'ACPR Pay', type: 'Payment', country: 'FR' }]
      },
      DNB: {
        all: [
          { name: 'DNB Credit', type: 'Credit', country: 'NL' },
          { name: 'DNB Payment', type: 'Payment', country: 'NL' }
        ],
        credit: [{ name: 'DNB Credit', type: 'Credit', country: 'NL' }],
        payment: [{ name: 'DNB Payment', type: 'Payment', country: 'NL' }]
      }
    };
    return mockData[source]?.[type] || [];
  }

  // ----- Effect: load data when source/type changes -----
  $effect(() => {
    (async () => {
      loading = true;
      error = null;
      try {
        tableData = await fetchTPPs(activeSource, activeType);
      } catch (err) {
        error = err.message;
        tableData = [];
      } finally {
        loading = false;
      }
    })();
  });

  // ----- Event handlers -----
  function selectSourceType(source, typeKey) {
    activeSource = source;
    activeType = typeKey;
    openDropdown = null;  // close dropdown after selection
  }

  // Hover logic
  function onTabEnter(source) {
    if (hoverTimeout) clearTimeout(hoverTimeout);
    openDropdown = source;
  }

  function onTabLeave() {
    hoverTimeout = setTimeout(() => {
      const dropdownElem = document.querySelector(`.dropdown-${openDropdown}`);
      if (dropdownElem && dropdownElem.matches(':hover')) {
        return;
      }
      openDropdown = null;
    }, 150);
  }

  function onDropdownEnter() {
    if (hoverTimeout) clearTimeout(hoverTimeout);
  }

  function onDropdownLeave() {
    openDropdown = null;
  }

  function handleAdmin() {
    alert('Admin page – would navigate to admin panel');
  }

  function handleSettings() {
    alert('Settings page – would open settings');
  }
</script>

<main>
  <h1>PSD2 Third‑Party Providers</h1>

  <!-- Top navigation bar -->
  <div class="top-nav">
    <div class="source-tabs">
      {#each sources as source}
        <div
          class="source-tab {activeSource === source ? 'active' : ''}"
          on:mouseenter={() => onTabEnter(source)}
          on:mouseleave={onTabLeave}
        >
          {source}
          {#if openDropdown === source}
            <div
              class="type-dropdown dropdown-{source}"
              on:mouseenter={onDropdownEnter}
              on:mouseleave={onDropdownLeave}
            >
              {#each types as type}
                <a
                  href="#"
                  on:click|preventDefault={() => selectSourceType(source, type.key)}
                >
                  {type.label}
                </a>
              {/each}
            </div>
          {/if}
        </div>
      {/each}
    </div>
    <div class="right-nav">
      <a href="#" on:click|preventDefault={handleAdmin}>Admin</a>
      <a href="#" on:click|preventDefault={handleSettings}>Settings</a>
    </div>
  </div>

  <!-- Breadcrumb -->
  <div class="breadcrumb">
    {breadcrumb}
  </div>

  <!-- Table or status -->
  {#if loading}
    <div class="loading">Loading data...</div>
  {:else if error}
    <div class="error">Error: {error}</div>
  {:else if tableData.length === 0}
    <div class="loading">No TPPs found for this selection.</div>
  {:else}
     <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Type</th>
          <th>Country</th>
        </tr>
      </thead>
      <tbody>
        {#each tableData as tpp}
          <tr>
            <td>{escapeHtml(tpp.name)}</td>
            <td>{escapeHtml(tpp.type)}</td>
            <td>{escapeHtml(tpp.country)}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
</main>

<style>
  /* Same CSS as before – omitted for brevity */
  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  main {
    max-width: 1200px;
    margin: 2rem auto;
    background: white;
    border-radius: 8px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    padding: 2rem;
    font-family: sans-serif;
  }

  .top-nav {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    border-bottom: 2px solid #ddd;
    margin-bottom: 1rem;
  }

  .source-tabs {
    display: flex;
    gap: 1rem;
    position: relative;
  }

  .source-tab {
    padding: 0.5rem 1rem;
    cursor: pointer;
    background: #f5f5f5;
    border-radius: 8px 8px 0 0;
    transition: background 0.2s;
    position: relative;
    margin-bottom: -2px;
  }

  .source-tab.active {
    background: #007bff;
    color: white;
    border-bottom: 2px solid #007bff;
  }

  .source-tab:hover {
    background: #e0e0e0;
  }

  .source-tab.active:hover {
    background: #0069d9;
  }

  .type-dropdown {
    position: absolute;
    top: 100%;
    left: 0;
    background: white;
    border: 1px solid #ccc;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    z-index: 100;
    min-width: 160px;
  }

  .type-dropdown a {
    display: block;
    padding: 0.75rem 1rem;
    text-decoration: none;
    color: #333;
    border-bottom: 1px solid #eee;
  }

  .type-dropdown a:last-child {
    border-bottom: none;
  }

  .type-dropdown a:hover {
    background: #f0f0f0;
  }

  .right-nav {
    display: flex;
    gap: 1.5rem;
    padding-bottom: 0.5rem;
  }

  .right-nav a {
    text-decoration: none;
    color: #555;
    font-weight: 500;
    transition: color 0.2s;
  }

  .right-nav a:hover {
    color: #007bff;
  }

  .breadcrumb {
    padding: 0.75rem 1rem;
    background: #f0f0f0;
    border-radius: 8px;
    margin: 0.5rem 0 1.5rem 0;
    font-size: 0.9rem;
    color: #555;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
  }

  th, td {
    border: 1px solid #ddd;
    padding: 8px;
    text-align: left;
  }

  th {
    background-color: #f2f2f2;
  }

  .loading, .error {
    padding: 2rem;
    text-align: center;
    color: #666;
  }

  .error {
    color: #d9534f;
  }
</style>