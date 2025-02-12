<script>
    import { onMount, getContext } from 'svelte'
    import { init } from './utils.js'
    import { Row, Col, Button, Icon, Card, Spinner } from 'sveltestrap'
    import Filters from './filters/Filters.svelte'
    import JobList from './joblist/JobList.svelte'
    import Refresher from './joblist/Refresher.svelte'
    import Sorting from './joblist/SortSelection.svelte'
    import MetricSelection from './MetricSelection.svelte'
    import UserOrProject from './filters/UserOrProject.svelte'

    const { query: initq } = init()

    const ccconfig = getContext('cc-config')

    export let filterPresets = {}

    let filters, jobList, matchedJobs = null
    let sorting = { field: 'startTime', order: 'DESC' }, isSortingOpen = false, isMetricsSelectionOpen = false
    let metrics = filterPresets.cluster
        ? ccconfig[`plot_list_selectedMetrics:${filterPresets.cluster}`] || ccconfig.plot_list_selectedMetrics
        : ccconfig.plot_list_selectedMetrics

    // The filterPresets are handled by the Filters component,
    // so we need to wait for it to be ready before we can start a query.
    // This is also why JobList component starts out with a paused query.
    onMount(() => filters.update())
</script>

<Row>
    {#if $initq.fetching}
        <Col xs="auto">
            <Spinner/>
        </Col>
    {:else if $initq.error}
        <Col xs="auto">
            <Card body color="danger">{$initq.error.message}</Card>
        </Col>
    {/if}
</Row>
<Row>
    <Col xs="auto">
        <Button
            outline color="primary"
            on:click={() => (isSortingOpen = true)}>
            <Icon name="sort-up"/> Sorting
        </Button>
        <Button
            outline color="primary"
            on:click={() => (isMetricsSelectionOpen = true)}>
            <Icon name="graph-up"/> Metrics
        </Button>
        <Button disabled outline>{matchedJobs == null ? 'Loading...' : `${matchedJobs} jobs`}</Button>
    </Col>
    <Col xs="auto">
        <Filters
            filterPresets={filterPresets}
            bind:this={filters}
            on:update={({ detail }) => jobList.update(detail.filters)} />
    </Col>

    <Col xs="3" style="margin-left: auto;">
        <UserOrProject on:update={({ detail }) => filters.update(detail)}/>
    </Col>
    <Col xs="2">
        <Refresher on:reload={() => jobList.update()} />
    </Col>
</Row>
<br/>
<Row>
    <Col>
        <JobList
            bind:metrics={metrics}
            bind:sorting={sorting}
            bind:matchedJobs={matchedJobs}
            bind:this={jobList} />
    </Col>
</Row>

<Sorting
    bind:sorting={sorting}
    bind:isOpen={isSortingOpen} />

<MetricSelection
    cluster={filterPresets.cluster}
    configName="plot_list_selectedMetrics"
    bind:metrics={metrics}
    bind:isOpen={isMetricsSelectionOpen} />
