<script lang="ts" context="module">
  import type { QueryCondition } from './QueryPills.svelte';

  export interface Query {
    id: string;
    name: string;
    conditions: QueryCondition[];
    lastExecuted?: Date;
    resultCount?: number;
  }

  export function queryToJSON(query: Query): string {
    return JSON.stringify({
      ...query,
      lastExecuted: query.lastExecuted?.toISOString()
    });
  }

  export function queryFromJSON(json: string): Query {
    const parsed = JSON.parse(json);
    return {
      ...parsed,
      lastExecuted: parsed.lastExecuted ? new Date(parsed.lastExecuted) : undefined
    };
  }
</script>

<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import QueryPills from './QueryPills.svelte';

  const dispatch = createEventDispatcher<{
    queryUpdate: Query;
    executeQuery: string;
  }>();

  export let query: Query;
  export let isCurrent = false;

  function handleQueryUpdate(event: CustomEvent<QueryCondition[]>) {
    const updatedQuery = { ...query, conditions: event.detail };
    query = updatedQuery;
    dispatch('queryUpdate', updatedQuery);
  }

  function executeQuery() {
    const updatedQuery = {
      ...query,
      lastExecuted: new Date(),
      resultCount: Math.floor(Math.random() * 1000) // Simulated result count
    };
    query = updatedQuery;
    dispatch('executeQuery', query.id);
  }

  function formatDate(date: Date): string {
    return new Intl.DateTimeFormat('en-US', {
      month: 'short',
      day: 'numeric',
      hour: 'numeric',
      minute: 'numeric'
    }).format(date);
  }
</script>

<div class="query-card" class:current={isCurrent}>
  <div class="query-header">
    <h3>{query.name}</h3>
    <div class="query-actions">
      {#if isCurrent}
        <button class="execute" on:click={executeQuery}>
          Execute
        </button>
      {:else}
        <button class="set-current" on:click={() => dispatch('setCurrent', query.id)}>
          Set as Current
        </button>
      {/if}
    </div>
  </div>

  <QueryPills 
    conditions={query.conditions} 
    on:update={handleQueryUpdate}
  />

  <div class="query-footer">
    {#if query.lastExecuted}
      <span class="timestamp">
        Last executed: {formatDate(query.lastExecuted)}
      </span>
    {/if}
    {#if query.resultCount !== undefined}
      <span class="results">
        {query.resultCount.toLocaleString()} results
      </span>
    {/if}
  </div>
</div>

<style>
  .query-card {
    background-color: white;
    border: 1px solid #e2e8f0;
    border-radius: 0.5rem;
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .query-card.current {
    border-color: #3b82f6;
  }

  .query-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .query-header h3 {
    margin: 0;
    color: #1e293b;
    font-size: 1.25rem;
  }

  .query-actions {
    display: flex;
    gap: 0.5rem;
  }

  .execute, .set-current {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 0.375rem;
    cursor: pointer;
    font-weight: 500;
  }

  .execute {
    background-color: #3b82f6;
    color: white;
  }

  .execute:hover {
    background-color: #2563eb;
  }

  .set-current {
    background-color: #f1f5f9;
    color: #1e293b;
  }

  .set-current:hover {
    background-color: #e2e8f0;
  }

  .query-footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #64748b;
    font-size: 0.875rem;
  }
</style> 