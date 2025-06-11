<script lang="ts" context="module">
  import type { Query } from './Query.svelte';

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
  import Query from './Query.svelte';

  const dispatch = createEventDispatcher<{
    queryUpdate: Query;
    createQuery: Query;
    executeQuery: string;
  }>();

  export let queries: Query[] = [];
  export let currentQueryId: string | null = null;

  let showNewQueryModal = false;
  let newQueryName = '';

  function handleQueryUpdate(event: CustomEvent<Query>) {
    const updatedQuery = event.detail;
    queries = queries.map(q => q.id === updatedQuery.id ? updatedQuery : q);
    dispatch('queryUpdate', updatedQuery);
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

  function handleExecuteQuery(event: CustomEvent<string>) {
    dispatch('executeQuery', event.detail);
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
        <Query 
          {query}
          isCurrent={true}
          on:queryUpdate={handleQueryUpdate}
          on:executeQuery={handleExecuteQuery}
        />
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
            <Query 
              {query}
              isCurrent={false}
              on:queryUpdate={handleQueryUpdate}
              on:executeQuery={handleExecuteQuery}
              on:setCurrent={() => setAsCurrent(query.id)}
            />
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