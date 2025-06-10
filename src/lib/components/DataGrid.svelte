<script lang="ts" context="module">
  export interface Column {
    key: string;
    label: string;
    sortable?: boolean;
    width?: string;
    align?: 'left' | 'center' | 'right';
  }

  export interface SortConfig {
    key: string;
    direction: 'asc' | 'desc';
  }

  export interface PageInfo {
    currentPage: number;
    pageSize: number;
    totalItems: number;
    totalPages: number;
  }
</script>

<script lang="ts">
  import { onMount } from 'svelte';

  export let columns: Column[] = [];
  export let data: any[] = [];
  export let pageInfo: PageInfo;
  export let loading = false;
  export let onPageChange: (page: number) => void;
  export let onSort: (sort: SortConfig) => void;
  export let onPageSizeChange: (size: number) => void;

  let currentSort: SortConfig | null = null;
  let tableRef: HTMLTableElement;
  let virtualScrollContainer: HTMLDivElement;
  let rowHeight = 40; // Height of each row in pixels
  let visibleRows = 0;
  let startIndex = 0;
  let endIndex = 0;
  let columnWidths: { [key: string]: string } = {};

  // Calculate visible rows based on container height
  function updateVisibleRows() {
    if (!virtualScrollContainer) return;
    const containerHeight = virtualScrollContainer.clientHeight;
    visibleRows = Math.ceil(containerHeight / rowHeight) + 2; // Add buffer rows
    updateVisibleRange();
  }

  // Update the range of rows that should be visible
  function updateVisibleRange() {
    if (!virtualScrollContainer) return;
    const scrollTop = virtualScrollContainer.scrollTop;
    startIndex = Math.floor(scrollTop / rowHeight);
    endIndex = Math.min(startIndex + visibleRows, data.length);
    startIndex = Math.max(0, startIndex - 1); // Add one buffer row at the top
  }

  // Handle scroll events
  function handleScroll() {
    if (!virtualScrollContainer) return;
    requestAnimationFrame(updateVisibleRange);
  }

  // Handle sort click
  function handleSort(column: Column) {
    if (!column.sortable) return;
    
    if (!currentSort || currentSort.key !== column.key) {
      currentSort = { key: column.key, direction: 'asc' };
    } else {
      currentSort.direction = currentSort.direction === 'asc' ? 'desc' : 'asc';
    }
    
    onSort(currentSort);
  }

  // Format cell value based on data type
  function formatCellValue(value: any): string {
    if (value === null || value === undefined) return '';
    if (typeof value === 'boolean') return value ? 'Yes' : 'No';
    if (value instanceof Date) return value.toLocaleDateString();
    return String(value);
  }

  // Calculate column widths
  function calculateColumnWidths() {
    const widths: { [key: string]: string } = {};
    columns.forEach(column => {
      widths[column.key] = column.width || 'auto';
    });
    return widths;
  }

  // Handle page size change
  function handlePageSizeChange(event: Event) {
    const select = event.target as HTMLSelectElement;
    onPageSizeChange(Number(select.value));
  }

  // Calculate total height for virtual scrolling
  $: totalHeight = data.length * rowHeight;
  $: visibleData = data.slice(startIndex, endIndex);
  $: offsetY = startIndex * rowHeight;
  $: columnWidths = calculateColumnWidths();

  onMount(() => {
    updateVisibleRows();
    window.addEventListener('resize', updateVisibleRows);
    return () => {
      window.removeEventListener('resize', updateVisibleRows);
    };
  });
</script>

<div class="data-grid">
  <div class="grid-header">
    <div class="page-info">
      Showing {pageInfo.currentPage * pageInfo.pageSize - pageInfo.pageSize + 1} to {Math.min(pageInfo.currentPage * pageInfo.pageSize, pageInfo.totalItems)} of {pageInfo.totalItems.toLocaleString()} results
    </div>
    <div class="page-size">
      <label for="page-size">Rows per page:</label>
      <select
        id="page-size"
        value={pageInfo.pageSize}
        on:change={handlePageSizeChange}
      >
        <option value="10">10</option>
        <option value="25">25</option>
        <option value="50">50</option>
        <option value="100">100</option>
      </select>
    </div>
  </div>

  <div class="table-container">
    <table bind:this={tableRef}>
      <thead>
        <tr>
          {#each columns as column}
            <th
              style="width: {columnWidths[column.key]}; text-align: {column.align || 'left'}"
              class:sortable={column.sortable}
              class:sorted={currentSort?.key === column.key}
              class:asc={currentSort?.key === column.key && currentSort.direction === 'asc'}
              class:desc={currentSort?.key === column.key && currentSort.direction === 'desc'}
              on:click={() => handleSort(column)}
            >
              {column.label}
              {#if column.sortable}
                <span class="sort-icon">
                  {#if currentSort?.key === column.key}
                    {currentSort.direction === 'asc' ? '↑' : '↓'}
                  {:else}
                    ↕
                  {/if}
                </span>
              {/if}
            </th>
          {/each}
        </tr>
      </thead>
    </table>

    <div
      class="virtual-scroll-container"
      bind:this={virtualScrollContainer}
      on:scroll={handleScroll}
    >
      <div class="virtual-scroll-content" style="height: {totalHeight}px">
        <div class="virtual-scroll-rows" style="transform: translateY({offsetY}px)">
          {#each visibleData as row, i}
            <div class="row" style="height: {rowHeight}px">
              {#each columns as column}
                <div
                  class="cell"
                  style="width: {columnWidths[column.key]}; text-align: {column.align || 'left'}"
                >
                  {formatCellValue(row[column.key])}
                </div>
              {/each}
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>

  <div class="pagination">
    <button
      class="page-button"
      disabled={pageInfo.currentPage === 1}
      on:click={() => onPageChange(pageInfo.currentPage - 1)}
    >
      Previous
    </button>

    <div class="page-numbers">
      {#each Array(Math.min(5, pageInfo.totalPages)) as _, i}
        {@const pageNum = i + 1}
        <button
          class="page-number"
          class:active={pageNum === pageInfo.currentPage}
          on:click={() => onPageChange(pageNum)}
        >
          {pageNum}
        </button>
      {/each}
      {#if pageInfo.totalPages > 5}
        <span class="page-ellipsis">...</span>
        <button
          class="page-number"
          class:active={pageInfo.currentPage === pageInfo.totalPages}
          on:click={() => onPageChange(pageInfo.totalPages)}
        >
          {pageInfo.totalPages}
        </button>
      {/if}
    </div>

    <button
      class="page-button"
      disabled={pageInfo.currentPage === pageInfo.totalPages}
      on:click={() => onPageChange(pageInfo.currentPage + 1)}
    >
      Next
    </button>
  </div>

  {#if loading}
    <div class="loading-overlay">
      <div class="loading-spinner"></div>
    </div>
  {/if}
</div>

<style>
  .data-grid {
    position: relative;
    background-color: white;
    border: 1px solid #e2e8f0;
    border-radius: 0.5rem;
    overflow: hidden;
    min-height: 400px;
  }

  .grid-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    border-bottom: 1px solid #e2e8f0;
    background-color: #f8fafc;
  }

  .page-info {
    color: #64748b;
    font-size: 0.875rem;
  }

  .page-size {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .page-size label {
    color: #64748b;
    font-size: 0.875rem;
  }

  .page-size select {
    padding: 0.375rem 0.75rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    background-color: white;
    color: #1e293b;
    font-size: 0.875rem;
  }

  .table-container {
    position: relative;
    overflow: hidden;
    background-color: white;
  }

  table {
    width: 100%;
    border-collapse: collapse;
    table-layout: fixed;
  }

  thead {
    position: sticky;
    top: 0;
    z-index: 10;
    background-color: #f8fafc;
  }

  th {
    padding: 0.75rem 1rem;
    text-align: left;
    font-weight: 500;
    color: #1e293b;
    border-bottom: 1px solid #e2e8f0;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  th.sortable {
    cursor: pointer;
    user-select: none;
  }

  th.sortable:hover {
    background-color: #f1f5f9;
  }

  th.sorted {
    background-color: #f1f5f9;
  }

  .sort-icon {
    margin-left: 0.5rem;
    color: #64748b;
  }

  .virtual-scroll-container {
    height: 400px;
    overflow-y: auto;
    overflow-x: hidden;
    background-color: white;
  }

  .virtual-scroll-content {
    position: relative;
  }

  .virtual-scroll-rows {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
  }

  .row {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(0, 1fr));
    border-bottom: 1px solid #e2e8f0;
    background-color: white;
  }

  .row:hover {
    background-color: #f8fafc;
  }

  .cell {
    padding: 0.75rem 1rem;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: #1e293b;
    font-size: 0.875rem;
  }

  .pagination {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 1rem;
    border-top: 1px solid #e2e8f0;
    background-color: #f8fafc;
  }

  .page-numbers {
    display: flex;
    gap: 0.5rem;
    align-items: center;
  }

  .page-number {
    padding: 0.375rem 0.75rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    background-color: white;
    color: #1e293b;
    font-size: 0.875rem;
    cursor: pointer;
  }

  .page-number:hover {
    background-color: #f1f5f9;
  }

  .page-number.active {
    background-color: #3b82f6;
    color: white;
    border-color: #3b82f6;
  }

  .page-button {
    padding: 0.375rem 0.75rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    background-color: white;
    color: #1e293b;
    font-size: 0.875rem;
    cursor: pointer;
  }

  .page-button:hover:not(:disabled) {
    background-color: #f1f5f9;
  }

  .page-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .page-ellipsis {
    color: #64748b;
    font-size: 0.875rem;
  }

  .loading-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(255, 255, 255, 0.9);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 20;
  }

  .loading-spinner {
    width: 2rem;
    height: 2rem;
    border: 3px solid #e2e8f0;
    border-top-color: #3b82f6;
    border-radius: 50%;
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }
</style> 