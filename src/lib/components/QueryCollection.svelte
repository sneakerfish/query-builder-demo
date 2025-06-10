<script lang="ts" context="module">
  import type { QueryCondition } from './QueryPills.svelte';

  export interface Query {
    id: string;
    name: string;
    conditions: QueryCondition[];
    lastExecuted?: Date;
    resultCount?: number;
  }
</script>

<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import QueryPills from './QueryPills.svelte';

  const dispatch = createEventDispatcher<{
    queryUpdate: Query;
    createQuery: Query;
    executeQuery: string;
  }>();

  export let queries: Query[] = [];
  export let currentQueryId: string | null = null;

  let showNewQueryModal = false;
  let newQueryName = '';

  function handleQueryUpdate(event: CustomEvent<QueryCondition[]>) {
    if (!currentQueryId) return;
    
    const updatedQuery = queries.find(q => q.id === currentQueryId);
    if (!updatedQuery) return;

    const newQuery = { ...updatedQuery, conditions: event.detail };
    queries = queries.map(q => q.id === currentQueryId ? newQuery : q);
    dispatch('queryUpdate', newQuery);
  }

  function createNewQuery() {
    if (!newQueryName.trim()) return;
    
    const newQuery: Query = {
      id: crypto.randomUUID(),
      name: newQueryName.trim(),
      conditions: [],
      lastExecuted: new Date(),
      resultCount: 0
    };

    queries = [...queries, newQuery];
    currentQueryId = newQuery.id;
    dispatch('createQuery', newQuery);
    newQueryName = '';
    showNewQueryModal = false;
  }

  function executeQuery() {
    if (!currentQueryId) return;
    
    const updatedQuery = queries.find(q => q.id === currentQueryId);
    if (!updatedQuery) return;

    const newQuery = {
      ...updatedQuery,
      lastExecuted: new Date(),
      resultCount: Math.floor(Math.random() * 1000) // Simulated result count
    };

    queries = queries.map(q => q.id === currentQueryId ? newQuery : q);
    dispatch('executeQuery', currentQueryId);
  }

  function setAsCurrent(queryId: string) {
    currentQueryId = queryId;
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

<div class="query-collection">
  <div class="header">
    <h2>Queries</h2>
    <button class="new-query" on:click={() => showNewQueryModal = true}>
      <span>+</span> New Query
    </button>
  </div>

  <div class="queries">
    {#if currentQueryId}
      {#each queries.filter(q => q.id === currentQueryId) as query (query.id)}
        <div class="query-card current">
          <div class="query-header">
            <h3>{query.name}</h3>
            <div class="query-actions">
              <button class="execute" on:click={executeQuery}>
                Execute
              </button>
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
      {/each}
    {:else}
      <div class="no-current-query">
        <p>No current query selected. Create a new query or select an existing one.</p>
      </div>
    {/if}

    {#if queries.filter(q => q.id !== currentQueryId).length > 0}
      <div class="historical-queries">
        <h3>Historical Queries</h3>
        <div class="query-grid">
          {#each queries.filter(q => q.id !== currentQueryId) as query (query.id)}
            <div class="query-card historical">
              <div class="query-header">
                <h3>{query.name}</h3>
                <div class="query-actions">
                  <button class="set-current" on:click={() => setAsCurrent(query.id)}>
                    Set as Current
                  </button>
                </div>
              </div>

              <div class="readonly-pills">
                {#each query.conditions as condition (condition.id)}
                  <div 
                    class="pill" 
                    class:boolean={condition.type === 'boolean'} 
                    class:range={condition.type === 'range'} 
                    class:companies={condition.type === 'companies'}
                    role="button"
                    tabindex="0"
                    on:click={() => setAsCurrent(query.id)}
                    on:keydown={(e) => e.key === 'Enter' && setAsCurrent(query.id)}
                  >
                    <div class="pill-content">
                      <span class="label">{condition.label}</span>
                      {#if condition.type === 'companies'}
                        <div class="companies-list">
                          {#each (condition.value as string[]).slice(0, 3) as company}
                            <span class="company">{company}</span>
                          {/each}
                          {#if (condition.value as string[]).length > 3}
                            <span class="more">+{(condition.value as string[]).length - 3} more</span>
                          {/if}
                        </div>
                      {:else}
                        <span class="value">
                          {#if condition.type === 'boolean'}
                            {condition.value ? 'Yes' : 'No'}
                          {:else if condition.type === 'range'}
                            {`${(condition.value as { min: number; max: number }).min.toLocaleString()} - ${(condition.value as { min: number; max: number }).max.toLocaleString()}`}
                          {:else}
                            {condition.value}
                          {/if}
                        </span>
                      {/if}
                    </div>
                  </div>
                {/each}
              </div>

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
          {/each}
        </div>
      </div>
    {/if}
  </div>
</div>

{#if showNewQueryModal}
  <div class="modal-backdrop" on:click={() => showNewQueryModal = false}>
    <div class="modal" on:click|stopPropagation>
      <h2>New Query</h2>
      <form on:submit|preventDefault={createNewQuery}>
        <div class="form-group">
          <label for="query-name">Query Name</label>
          <input
            id="query-name"
            type="text"
            bind:value={newQueryName}
            placeholder="Enter a name for your query"
            required
          />
        </div>

        <div class="modal-actions">
          <button type="button" class="secondary" on:click={() => showNewQueryModal = false}>
            Cancel
          </button>
          <button type="submit" class="primary">
            Create Query
          </button>
        </div>
      </form>
    </div>
  </div>
{/if}

<style>
  .query-collection {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .header h2 {
    margin: 0;
    color: #1e293b;
    font-size: 1.5rem;
  }

  .new-query {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 1rem;
    background-color: #3b82f6;
    color: white;
    border: none;
    border-radius: 0.375rem;
    cursor: pointer;
    font-weight: 500;
  }

  .new-query:hover {
    background-color: #2563eb;
  }

  .queries {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

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

  .historical-queries {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .historical-queries h3 {
    margin: 0;
    color: #1e293b;
    font-size: 1.25rem;
  }

  .query-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 1rem;
  }

  .readonly-pills {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .pill {
    display: inline-flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.5rem 0.75rem;
    background-color: #f1f5f9;
    border: 1px solid #e2e8f0;
    border-radius: 9999px;
    font-size: 0.875rem;
    color: #1e293b;
    cursor: pointer;
  }

  .pill:hover {
    background-color: #e2e8f0;
  }

  .pill.boolean {
    background-color: #f0fdf4;
    border-color: #86efac;
  }

  .pill.range {
    background-color: #eff6ff;
    border-color: #93c5fd;
  }

  .pill.companies {
    background-color: #faf5ff;
    border-color: #d8b4fe;
  }

  .pill-content {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .label {
    font-weight: 500;
  }

  .value {
    color: #64748b;
  }

  .companies-list {
    display: flex;
    flex-wrap: wrap;
    gap: 0.25rem;
  }

  .company {
    background-color: white;
    padding: 0.125rem 0.375rem;
    border-radius: 0.25rem;
    font-size: 0.75rem;
  }

  .more {
    color: #64748b;
    font-size: 0.75rem;
  }

  .remove {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 1.25rem;
    height: 1.25rem;
    padding: 0;
    background: none;
    border: none;
    color: #64748b;
    cursor: pointer;
    font-size: 1.25rem;
    line-height: 1;
  }

  .remove:hover {
    color: #ef4444;
  }

  .no-current-query {
    text-align: center;
    padding: 2rem;
    background-color: #f8fafc;
    border: 1px dashed #e2e8f0;
    border-radius: 0.5rem;
    color: #64748b;
  }

  .modal-backdrop {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }

  .modal {
    background: white;
    border-radius: 0.5rem;
    padding: 1.5rem;
    width: 100%;
    max-width: 500px;
    box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  }

  .modal h2 {
    margin: 0 0 1.5rem;
    color: #1e293b;
    font-size: 1.25rem;
  }

  .form-group {
    margin-bottom: 1rem;
  }

  .form-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: #475569;
    font-weight: 500;
  }

  .form-group input[type="text"] {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    font-size: 0.875rem;
  }

  .modal-actions {
    display: flex;
    justify-content: flex-end;
    gap: 0.75rem;
    margin-top: 1.5rem;
  }

  .modal-actions button {
    padding: 0.5rem 1rem;
    border-radius: 0.375rem;
    font-size: 0.875rem;
    font-weight: 500;
    cursor: pointer;
  }

  .modal-actions .secondary {
    background: #f1f5f9;
    border: 1px solid #e2e8f0;
    color: #475569;
  }

  .modal-actions .secondary:hover {
    background: #e2e8f0;
  }

  .modal-actions .primary {
    background: #3b82f6;
    border: 1px solid #3b82f6;
    color: white;
  }

  .modal-actions .primary:hover {
    background: #2563eb;
  }
</style> 