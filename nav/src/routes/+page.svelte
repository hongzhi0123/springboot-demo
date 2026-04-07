<!-- src/routes/+page.svelte -->
<script>
	// State management
	let currentSource = $state("bafin");
	let currentType = $state("all");
	let isClicking = $state(false);
	let submenuOpen = $state(false);
	let hoveredSource = $state(null);

	// Mock Data
	const tppData = {
		eba: {
			name: "EBA",
			fullName: "European Banking Authority",
			counts: { all: 1240, credit: 892, payment: 348 },
			data: [
				{
					name: "Deutsche Bank AG",
					bic: "DEUTDEFF",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2018-09-14",
				},
				{
					name: "Commerzbank AG",
					bic: "COBADEFF",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2018-10-01",
				},
				{
					name: "Wirecard Bank AG",
					bic: "WIREDEMM",
					type: "payment",
					country: "DE",
					status: "suspended",
					date: "2019-06-01",
				},
				{
					name: "N26 GmbH",
					bic: "NTSBDEB1",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2016-07-22",
				},
				{
					name: "PayPal (Europe) S.a.r.l.",
					bic: "PPABFRPP",
					type: "payment",
					country: "FR",
					status: "active",
					date: "2015-03-12",
				},
				{
					name: "BNP Paribas",
					bic: "BNPAFRPP",
					type: "credit",
					country: "FR",
					status: "active",
					date: "2018-01-01",
				},
				{
					name: "ING Bank NV",
					bic: "INGBNL2A",
					type: "credit",
					country: "NL",
					status: "active",
					date: "2018-04-15",
				},
				{
					name: "Adyen NV",
					bic: "ADYBNL2A",
					type: "payment",
					country: "NL",
					status: "active",
					date: "2017-11-30",
				},
			],
		},
		bafin: {
			name: "BaFin",
			fullName: "Federal Financial Supervisory Authority (Germany)",
			counts: { all: 856, credit: 634, payment: 222 },
			data: [
				{
					name: "Deutsche Bank AG",
					bic: "DEUTDEFF",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2018-09-14",
				},
				{
					name: "Commerzbank AG",
					bic: "COBADEFF",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2018-10-01",
				},
				{
					name: "DZ BANK AG",
					bic: "GENODEFF",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2018-11-15",
				},
				{
					name: "Wirecard Bank AG",
					bic: "WIREDEMM",
					type: "payment",
					country: "DE",
					status: "suspended",
					date: "2019-06-01",
				},
				{
					name: "N26 GmbH",
					bic: "NTSBDEB1",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2016-07-22",
				},
				{
					name: "Solarisbank AG",
					bic: "SOBKDEBB",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2019-02-28",
				},
				{
					name: "Fidor Bank AG",
					bic: "FDDODEMM",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2018-01-15",
				},
				{
					name: "Klarna Bank AB (publ)",
					bic: "KLRNSESS",
					type: "payment",
					country: "SE",
					status: "active",
					date: "2018-03-01",
				},
			],
		},
		acpr: {
			name: "ACPR",
			fullName:
				"Prudential Supervision and Resolution Authority (France)",
			counts: { all: 723, credit: 521, payment: 202 },
			data: [
				{
					name: "BNP Paribas",
					bic: "BNPAFRPP",
					type: "credit",
					country: "FR",
					status: "active",
					date: "2018-01-01",
				},
				{
					name: "Société Générale",
					bic: "SOGEFRPP",
					type: "credit",
					country: "FR",
					status: "active",
					date: "2018-01-01",
				},
				{
					name: "Crédit Agricole",
					bic: "AGRIFRPP",
					type: "credit",
					country: "FR",
					status: "active",
					date: "2018-02-01",
				},
				{
					name: "PayPal (Europe) S.a.r.l.",
					bic: "PPABFRPP",
					type: "payment",
					country: "FR",
					status: "active",
					date: "2015-03-12",
				},
				{
					name: "Lydia Solutions",
					bic: "LYDSFR22",
					type: "payment",
					country: "FR",
					status: "active",
					date: "2019-09-01",
				},
				{
					name: "Qonto (Olinda)",
					bic: "QNTOFRP1",
					type: "payment",
					country: "FR",
					status: "active",
					date: "2017-12-01",
				},
			],
		},
		fma: {
			name: "FMA",
			fullName: "Financial Market Authority (Austria)",
			counts: { all: 198, credit: 142, payment: 56 },
			data: [
				{
					name: "Erste Bank und Sparkassen",
					bic: "GIBAATWW",
					type: "credit",
					country: "AT",
					status: "active",
					date: "2018-06-01",
				},
				{
					name: "Raiffeisen Bankengruppe Österreich",
					bic: "RZBAATWW",
					type: "credit",
					country: "AT",
					status: "active",
					date: "2018-07-15",
				},
				{
					name: "UniCredit Bank Austria AG",
					bic: "BKAUATWW",
					type: "credit",
					country: "AT",
					status: "active",
					date: "2018-08-20",
				},
				{
					name: "N26 GmbH",
					bic: "NTSBDEB1",
					type: "credit",
					country: "DE",
					status: "active",
					date: "2016-07-22",
				},
				{
					name: "PayPal (Europe) S.a.r.l.",
					bic: "PPABFRPP",
					type: "payment",
					country: "FR",
					status: "active",
					date: "2015-03-12",
				},
				{
					name: "Klarna Bank AB (publ)",
					bic: "KLRNSESS",
					type: "payment",
					country: "SE",
					status: "active",
					date: "2018-03-01",
				},
			],
		},
		dnb: {
			name: "DNB",
			fullName: "De Nederlandsche Bank",
			counts: { all: 412, credit: 298, payment: 114 },
			data: [
				{
					name: "ING Bank NV",
					bic: "INGBNL2A",
					type: "credit",
					country: "NL",
					status: "active",
					date: "2018-04-15",
				},
				{
					name: "ABN AMRO Bank NV",
					bic: "ABNANL2A",
					type: "credit",
					country: "NL",
					status: "active",
					date: "2018-05-01",
				},
				{
					name: "Rabobank",
					bic: "RABONL2U",
					type: "credit",
					country: "NL",
					status: "active",
					date: "2018-03-20",
				},
				{
					name: "Adyen NV",
					bic: "ADYBNL2A",
					type: "payment",
					country: "NL",
					status: "active",
					date: "2017-11-30",
				},
				{
					name: "Bunq BV",
					bic: "BUNQNL2A",
					type: "credit",
					country: "NL",
					status: "active",
					date: "2016-11-01",
				},
			],
		},
	};

	const sources = [
		{ key: "eba", label: "🏛️ EBA" },
		{ key: "bafin", label: "🇩🇪 BaFin" },
		{ key: "acpr", label: "🇫🇷 ACPR" },
		{ key: "fma", label: "🇦🇹 FMA" },
		{ key: "dnb", label: "🇳🇱 DNB" },
	];

	const types = [
		{ key: "all", label: "All TPPs" },
		{ key: "credit", label: "Credit Institutes" },
		{ key: "payment", label: "Payment Institutions" },
	];

	// Derived values
	const currentData = $derived(tppData[currentSource]);
	const filteredData = $derived(
		currentType === "all"
			? currentData.data
			: currentData.data.filter((item) => item.type === currentType),
	);
	const typeLabel = $derived(
		types.find((t) => t.key === currentType)?.label || currentType,
	);

	// Handlers
	function handleSourceBtnClick(sourceKey) {
		if (currentSource === sourceKey) {
			submenuOpen = !submenuOpen;
		} else {
			currentSource = sourceKey;
			submenuOpen = true;
		}
	}

	function handleSourceMouseEnter(sourceKey) {
		hoveredSource = sourceKey;
		submenuOpen = true;
	}

	function handleSourceMouseLeave() {
		hoveredSource = null;
		submenuOpen = false;
	}

	function handleTypeClick(sourceKey, typeKey, event) {
		event.preventDefault();
		currentSource = sourceKey;
		currentType = typeKey;
		submenuOpen = false;
	}

	function handleBreadcrumbClick() {
		currentType = "all";
	}

	function formatDate(dateStr) {
		return new Date(dateStr).toLocaleDateString("en-GB");
	}

	function getTypeBadge(type) {
		const classes = type === "credit" ? "badge-credit" : "badge-payment";
		const label = type === "credit" ? "Credit" : "Payment";
		return `<span class="badge ${classes}">${label}</span>`;
	}

	function getStatusBadge(status) {
		const classes =
			status === "active" ? "badge-active" : "badge-suspended";
		return `<span class="badge ${classes}">${status}</span>`;
	}
</script>

<div class="header">
	<nav class="source-nav">
		{#each sources as source}
			{@const isActive = currentSource === source.key}
			{@const isHovered = hoveredSource === source.key}
			<div
				class="source"
				class:active={isActive}
				data-source={source.key}
				onmouseenter={() => handleSourceMouseEnter(source.key)}
				onmouseleave={() => handleSourceMouseLeave()}
			>
				<button
					class="source-btn"
					onclick={() => handleSourceBtnClick(source.key)}
					>{source.label}</button
				>
				{#if submenuOpen && (isHovered || (isActive && hoveredSource === null))}
					<div class="type-menu">
						{#each types as type}
							{@const typeCount =
								tppData[source.key].counts[type.key]}
							<a
								href="/{source.key}/{type.key}"
								class="type-link"
								class:active={isActive &&
									currentType === type.key}
								onclick={(e) =>
									handleTypeClick(source.key, type.key, e)}
							>
								{type.label}
								<span class="count"
									>{typeCount.toLocaleString()}</span
								>
							</a>
						{/each}
					</div>
				{/if}
			</div>
		{/each}
	</nav>

	<div class="breadcrumb-bar">
		<span class="breadcrumb-item" onclick={handleBreadcrumbClick}>
			{currentData.name}
		</span>
		<span class="breadcrumb-separator">›</span>
		<span class="breadcrumb-current">{typeLabel}</span>
	</div>
</div>

<div class="container">
	<div class="page-title">{currentData.name} Registry</div>
	<div class="page-subtitle">
		Showing {typeLabel.toLowerCase()} registered with {currentData.fullName}
	</div>

	<table class="data-table">
		<thead>
			<tr>
				<th>TPP Name</th>
				<th>BIC / ID</th>
				<th>Type</th>
				<th>Country</th>
				<th>Status</th>
				<th>Registration Date</th>
			</tr>
		</thead>
		<tbody>
			{#if filteredData.length === 0}
				<tr>
					<td colspan="6" class="empty-state"
						>No TPPs found for this selection</td
					>
				</tr>
			{:else}
				{#each filteredData as item (item.bic)}
					<tr>
						<td><strong>{item.name}</strong></td>
						<td><span class="bic-code">{item.bic}</span></td>
						<td>{@html getTypeBadge(item.type)}</td>
						<td>{item.country}</td>
						<td>{@html getStatusBadge(item.status)}</td>
						<td>{formatDate(item.date)}</td>
					</tr>
				{/each}
			{/if}
		</tbody>
	</table>
</div>

<style>
	:global(*) {
		margin: 0;
		padding: 0;
		box-sizing: border-box;
	}

	:global(body) {
		font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
			"Helvetica Neue", Arial, sans-serif;
		background: #f5f7fa;
		color: #333;
	}

	.header {
		background: white;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	.source-nav {
		display: flex;
		background: #f8f9fa;
		border-bottom: 3px solid #003366;
		padding: 0 40px;
		position: relative;
	}

	.source {
		position: relative;
	}

	.source-btn {
		padding: 16px 32px;
		border: none;
		background: transparent;
		cursor: pointer;
		font-size: 14px;
		font-weight: 500;
		color: #666;
		border-bottom: 3px solid transparent;
		margin-bottom: -3px;
		transition: all 0.2s;
		display: flex;
		align-items: center;
		gap: 8px;
	}

	.source-btn:hover {
		background: white;
		color: #003366;
		border-bottom-color: #003366;
	}

	.source.active .source-btn {
		background: white;
		color: #003366;
		border-bottom-color: #003366;
		font-weight: 600;
	}

	.type-menu {
		position: absolute;
		top: 100%;
		left: 0;
		min-width: 220px;
		background: white;
		border: 1px solid #e0e0e0;
		border-top: 3px solid #003366;
		box-shadow: 0 8px 16px rgba(0, 0, 0, 0.15);
		display: flex;
		flex-direction: column;
		z-index: 1000;
		animation: fadeIn 0.2s ease;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: translateY(-5px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.type-link {
		padding: 12px 20px;
		text-decoration: none;
		color: #444;
		font-size: 13px;
		border-bottom: 1px solid #f0f0f0;
		transition: all 0.15s;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.type-link:last-child {
		border-bottom: none;
	}

	.type-link:hover {
		background: #e0e4ea;
		color: #003366;
		padding-left: 24px;
	}

	.type-link.active {
		background: lightblue;
		color: black;
		font-weight: 600;
	}

	.count {
		font-size: 11px;
		opacity: 0.7;
		background: rgba(0, 0, 0, 0.1);
		padding: 2px 6px;
		border-radius: 12px;
	}

	.type-link.active .count {
		background: rgba(255, 255, 255, 0.2);
	}

	.breadcrumb-bar {
		background: #fafbfc;
		padding: 12px 40px;
		border-bottom: 1px solid #e1e4e8;
		font-size: 13px;
		display: flex;
		align-items: center;
		gap: 10px;
	}

	.breadcrumb-item {
		color: #0366d6;
		cursor: pointer;
		font-weight: 500;
	}

	.breadcrumb-item:hover {
		text-decoration: underline;
	}

	.breadcrumb-separator {
		color: #959da5;
		font-size: 14px;
	}

	.breadcrumb-current {
		color: #24292e;
		font-weight: 600;
	}

	.container {
		max-width: 1400px;
		margin: 30px auto;
		padding: 0 40px;
	}

	.page-title {
		font-size: 24px;
		font-weight: 600;
		color: #1a1a1a;
		margin-bottom: 8px;
	}

	.page-subtitle {
		color: #666;
		font-size: 14px;
		margin-bottom: 24px;
	}

	.data-table {
		width: 100%;
		background: white;
		border-radius: 6px;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
		overflow: hidden;
		border-collapse: collapse;
	}

	.data-table th {
		background: #f6f8fa;
		padding: 14px 16px;
		text-align: left;
		font-size: 12px;
		font-weight: 600;
		color: #444;
		text-transform: uppercase;
		letter-spacing: 0.5px;
		border-bottom: 2px solid #e1e4e8;
	}

	.data-table td {
		padding: 14px 16px;
		border-bottom: 1px solid #e1e4e8;
		font-size: 14px;
	}

	.data-table tr:nth-child(even) {
		background: #f0f4f8;
	}

	.data-table tr:hover {
		background: #f6f8fa;
	}

	:global(.badge) {
		display: inline-block;
		padding: 4px 10px;
		border-radius: 12px;
		font-size: 12px;
		font-weight: 600;
		text-transform: uppercase;
		letter-spacing: 0.3px;
	}

	:global(.badge-credit) {
		background: #e3f2fd;
		color: #1565c0;
	}

	:global(.badge-payment) {
		background: #f3e5f5;
		color: #6a1b9a;
	}

	:global(.badge-active) {
		background: #d4edda;
		color: #155724;
	}

	:global(.badge-suspended) {
		background: #fff3cd;
		color: #856404;
	}

	.bic-code {
		font-family: "Courier New", monospace;
		font-size: 13px;
		color: #666;
		background: #f6f8fa;
		padding: 2px 6px;
		border-radius: 3px;
	}

	.empty-state {
		text-align: center;
		padding: 60px;
		color: #666;
	}
</style>
