<script lang="ts">
  import { onMount } from 'svelte';
  import { v4 as uuidv4 } from 'uuid';
  import { faker } from '@faker-js/faker';
  import QueryCollection from '$lib/components/QueryCollection.svelte';
  import type { Query, QueryCondition } from '$lib/components/QueryCollection.svelte';
  import DataGrid from '$lib/components/DataGrid.svelte';
  import type { Column, SortConfig, PageInfo } from '$lib/components/DataGrid.svelte';

  // Define the data structure
  interface UserData {
    id: number;
    name: string;
    email: string;
    company: string;
    status: 'Active' | 'Inactive' | 'Pending';
    lastActive: Date;
    [key: string]: number | string | Date;
  }

  // Example columns for the data grid
  const columns: Column[] = [
    { key: 'id', label: 'ID', width: '80px', sortable: true },
    { key: 'name', label: 'Name', sortable: true },
    { key: 'email', label: 'Email', sortable: true },
    { key: 'company', label: 'Company', sortable: true },
    { key: 'status', label: 'Status', width: '100px', sortable: true },
    { key: 'lastActive', label: 'Last Active', width: '150px', sortable: true, align: 'right' }
  ];

  // State
  let queries: Query[] = [];
  let currentQueryId: string | null = null;
  let showResults = false;
  let data: UserData[] = [];
  let pageInfo: PageInfo = {
    currentPage: 1,
    pageSize: 25,
    totalItems: 0,
    totalPages: 0
  };
  let loading = false;
  let currentSort: SortConfig | null = null;

  // Generate example queries
  function generateExampleQueries(): Query[] {
    return [
      {
        id: uuidv4(),
        name: 'High Salary Tech Companies',
        conditions: [
          {
            id: uuidv4(),
            type: 'range' as const,
            label: 'Salary Range',
            value: { min: 120000, max: 200000 }
          },
          {
            id: uuidv4(),
            type: 'companies' as const,
            label: 'Companies',
            value: ['Google', 'Microsoft', 'Apple', 'Amazon', 'Meta']
          }
        ],
        lastExecuted: new Date('2024-03-15T10:30:00'),
        resultCount: 245
      },
      {
        id: uuidv4(),
        name: 'Recent Deceased',
        conditions: [
          {
            id: uuidv4(),
            type: 'boolean' as const,
            label: 'Deceased',
            value: true
          },
          {
            id: uuidv4(),
            type: 'range' as const,
            label: 'Date Range',
            value: { min: 2024, max: 2024 }
          }
        ],
        lastExecuted: new Date('2024-03-14T15:45:00'),
        resultCount: 12
      }
    ];
  }

  // Generate fake data using Faker
  function generateFakeData(page: number, pageSize: number, sort: SortConfig | null) {
    const totalItems = faker.number.int({ min: 50, max: 500 });
    const totalPages = Math.ceil(totalItems / pageSize);
    
    // Generate page of data
    const startIndex = (page - 1) * pageSize;
    const items = Array.from({ length: pageSize }, (_, i) => {
      const id = startIndex + i + 1;
      return {
        id,
        name: faker.person.fullName(),
        email: faker.internet.email(),
        company: faker.company.name(),
        status: faker.helpers.arrayElement(['Active', 'Inactive', 'Pending'] as const),
        lastActive: faker.date.recent({ days: 30 })
      };
    });

    // Sort data if needed
    if (sort) {
      items.sort((a, b) => {
        const aVal = a[sort.key];
        const bVal = b[sort.key];
        
        if (aVal instanceof Date && bVal instanceof Date) {
          return sort.direction === 'asc' ? aVal.getTime() - bVal.getTime() : bVal.getTime() - aVal.getTime();
        }
        
        if (typeof aVal === 'string' && typeof bVal === 'string') {
          return sort.direction === 'asc' ? aVal.localeCompare(bVal) : bVal.localeCompare(aVal);
        }
        
        return sort.direction === 'asc' ? Number(aVal) - Number(bVal) : Number(bVal) - Number(aVal);
      });
    }

    return {
      items,
      totalItems,
      totalPages
    };
  }

  // Load data
  async function loadData(page: number, pageSize: number, sort: SortConfig | null) {
    loading = true;
    
    // Simulate API delay
    await new Promise(resolve => setTimeout(resolve, 500));
    
    const result = generateFakeData(page, pageSize, sort);
    data = result.items;
    pageInfo = {
      currentPage: page,
      pageSize,
      totalItems: result.totalItems,
      totalPages: result.totalPages
    };
    
    loading = false;
  }

  // Event handlers
  function handlePageChange(page: number) {
    loadData(page, pageInfo.pageSize, currentSort);
  }

  function handlePageSizeChange(size: number) {
    loadData(1, size, currentSort);
  }

  function handleSort(sort: SortConfig) {
    currentSort = sort;
    loadData(pageInfo.currentPage, pageInfo.pageSize, sort);
  }

  function handleQueryUpdate(updatedQuery: Query) {
    queries = queries.map(q => q.id === updatedQuery.id ? updatedQuery : q);
  }

  function handleCreateQuery(newQuery: Query) {
    queries = [...queries, newQuery];
    currentQueryId = newQuery.id;
  }

  function handleExecuteQuery(queryId: string) {
    currentQueryId = queryId;
    showResults = true;
    loadData(1, 25, null);
  }

  onMount(() => {
    queries = generateExampleQueries();
  });
</script>

<div class="container">
  <h1>Query Builder Demo</h1>
  <p class="description">
    Create and manage queries, then view the results in a performant data grid.
  </p>

  <div class="view-toggle">
    <button
      class="toggle-button"
      class:active={!showResults}
      on:click={() => showResults = false}
    >
      Queries
    </button>
    <button
      class="toggle-button"
      class:active={showResults}
      on:click={() => showResults = true}
      disabled={!currentQueryId}
    >
      Results
    </button>
  </div>

  {#if !showResults}
    <QueryCollection
      {queries}
      {currentQueryId}
      on:queryUpdate={({ detail }) => handleQueryUpdate(detail)}
      on:createQuery={({ detail }) => handleCreateQuery(detail)}
      on:executeQuery={({ detail }) => handleExecuteQuery(detail)}
    />
  {:else}
    <div class="results-container">
      <div class="results-header">
        <h2>Query Results</h2>
        <button class="back-button" on:click={() => showResults = false}>
          Back to Queries
        </button>
      </div>
      <DataGrid
        {columns}
        {data}
        {pageInfo}
        {loading}
        onPageChange={handlePageChange}
        onPageSizeChange={handlePageSizeChange}
        onSort={handleSort}
      />
    </div>
  {/if}
</div>

<style>
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  h1 {
    font-size: 2rem;
    font-weight: 600;
    color: #1e293b;
    margin-bottom: 0.5rem;
  }

  .description {
    color: #64748b;
    margin-bottom: 2rem;
  }

  .view-toggle {
    display: flex;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .toggle-button {
    padding: 0.5rem 1rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    background: white;
    color: #475569;
    font-size: 0.875rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .toggle-button:hover:not(:disabled) {
    background: #f1f5f9;
    border-color: #cbd5e1;
  }

  .toggle-button.active {
    background: #3b82f6;
    border-color: #3b82f6;
    color: white;
  }

  .toggle-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .results-container {
    background: white;
    border: 1px solid #e2e8f0;
    border-radius: 0.5rem;
    overflow: hidden;
  }

  .results-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    border-bottom: 1px solid #e2e8f0;
    background: #f8fafc;
  }

  .results-header h2 {
    font-size: 1.25rem;
    font-weight: 600;
    color: #1e293b;
    margin: 0;
  }

  .back-button {
    padding: 0.5rem 1rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    background: white;
    color: #475569;
    font-size: 0.875rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .back-button:hover {
    background: #f1f5f9;
    border-color: #cbd5e1;
  }
</style>
