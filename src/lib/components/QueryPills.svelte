<script lang="ts" context="module">
  export type QueryCondition = {
    id: string;
    type: 'boolean' | 'range' | 'text' | 'companies';
    label: string;
    value: boolean | { min: number; max: number } | string | string[];
  };
</script>

<script lang="ts">
  import { v4 as uuidv4 } from 'uuid';

  export let conditions: QueryCondition[] = [];
  export let editingCondition: QueryCondition | null = null;
  export let showModal = false;
  export let selectedType: QueryCondition['type'] | null = null;
  export let newLabel = '';
  export let newValue: QueryCondition['value'] = '';
  export let newCompanies: string[] = [];
  export let newCompany = '';

  function handleRemove(id: string) {
    conditions = conditions.filter(c => c.id !== id);
  }

  function handleUpdate(event: CustomEvent<QueryCondition[]>) {
    conditions = event.detail;
  }

  function openModal(condition: QueryCondition | null = null) {
    editingCondition = condition;
    if (condition) {
      selectedType = condition.type;
      newLabel = condition.label;
      newValue = condition.value;
      if (condition.type === 'companies') {
        newCompanies = [...condition.value as string[]];
      }
    } else {
      selectedType = null;
      newLabel = '';
      newValue = '';
      newCompanies = [];
    }
    showModal = true;
  }

  function handleTypeSelect(type: QueryCondition['type']) {
    selectedType = type;
    if (type === 'boolean') {
      newValue = false;
    } else if (type === 'range') {
      newValue = { min: 0, max: 100 };
    } else if (type === 'companies') {
      newValue = [];
    } else {
      newValue = '';
    }
  }

  function handleSubmit() {
    if (!selectedType || !newLabel) return;

    const newCondition: QueryCondition = {
      id: editingCondition?.id ?? uuidv4(),
      type: selectedType,
      label: newLabel,
      value: selectedType === 'companies' ? [...newCompanies] : newValue
    };

    if (editingCondition) {
      conditions = conditions.map(c => 
        c.id === editingCondition!.id ? newCondition : c
      );
    } else {
      conditions = [...conditions, newCondition];
    }

    showModal = false;
  }

  function addCompany() {
    if (newCompany.trim() && !newCompanies.includes(newCompany.trim())) {
      newCompanies = [...newCompanies, newCompany.trim()];
      newCompany = '';
    }
  }

  function removeCompany(company: string) {
    newCompanies = newCompanies.filter(c => c !== company);
  }

  function formatValue(condition: QueryCondition): string {
    if (condition.type === 'boolean') {
      return condition.value ? 'Yes' : 'No';
    } else if (condition.type === 'range') {
      const range = condition.value as { min: number; max: number };
      return `${range.min.toLocaleString()} - ${range.max.toLocaleString()}`;
    } else if (condition.type === 'companies') {
      const companies = condition.value as string[];
      return companies.length > 3 
        ? `${companies.slice(0, 3).join(', ')} +${companies.length - 3} more`
        : companies.join(', ');
    }
    return condition.value as string;
  }
</script>

<div class="query-pills">
  {#each conditions as condition (condition.id)}
    <div 
      class="pill" 
      class:boolean={condition.type === 'boolean'} 
      class:range={condition.type === 'range'} 
      class:companies={condition.type === 'companies'}
      role="button"
      tabindex="0"
      on:click={() => openModal(condition)}
      on:keydown={(e) => e.key === 'Enter' && openModal(condition)}
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
          <span class="value">{formatValue(condition)}</span>
        {/if}
      </div>
      <button class="remove" on:click|stopPropagation={() => handleRemove(condition.id)}>
        ×
      </button>
    </div>
  {/each}
  <button class="add" on:click={() => openModal()}>
    <span>+</span> Add Condition
  </button>
</div>

{#if showModal}
  <div class="modal-backdrop" on:click={() => showModal = false}>
    <div class="modal" on:click|stopPropagation>
      <h2>{editingCondition ? 'Edit Condition' : 'Add Condition'}</h2>
      
      {#if !selectedType}
        <div class="type-selection">
          <h3>Select Condition Type</h3>
          <div class="type-options">
            <button class="type-option" on:click={() => handleTypeSelect('boolean')}>
              <span class="icon">✓</span>
              <div class="type-info">
                <strong>Boolean</strong>
                <span>Yes/No condition</span>
              </div>
            </button>
            <button class="type-option" on:click={() => handleTypeSelect('range')}>
              <span class="icon">↔</span>
              <div class="type-info">
                <strong>Range</strong>
                <span>Min/Max values</span>
              </div>
            </button>
            <button class="type-option" on:click={() => handleTypeSelect('text')}>
              <span class="icon">T</span>
              <div class="type-info">
                <strong>Text</strong>
                <span>Single text value</span>
              </div>
            </button>
            <button class="type-option" on:click={() => handleTypeSelect('companies')}>
              <span class="icon">🏢</span>
              <div class="type-info">
                <strong>Companies</strong>
                <span>List of companies</span>
              </div>
            </button>
          </div>
        </div>
      {:else}
        <form on:submit|preventDefault={handleSubmit}>
          <div class="form-group">
            <label for="condition-label">Label</label>
            <input
              id="condition-label"
              type="text"
              bind:value={newLabel}
              placeholder="Enter a label for this condition"
              required
            />
          </div>

          {#if selectedType === 'boolean'}
            <div class="form-group">
              <label for="boolean-value">Default Value</label>
              <div class="boolean-input">
                <label class="toggle" for="boolean-value">
                  <input
                    id="boolean-value"
                    type="checkbox"
                    bind:checked={newValue as boolean}
                  />
                  <span class="slider"></span>
                </label>
                <span>{newValue ? 'Yes' : 'No'}</span>
              </div>
            </div>
          {:else if selectedType === 'range'}
            <div class="form-group range-inputs">
              <div>
                <label for="min-value">Min Value</label>
                <input
                  id="min-value"
                  type="number"
                  bind:value={(newValue as { min: number; max: number }).min}
                  required
                />
              </div>
              <div>
                <label for="max-value">Max Value</label>
                <input
                  id="max-value"
                  type="number"
                  bind:value={(newValue as { min: number; max: number }).max}
                  required
                />
              </div>
            </div>
          {:else if selectedType === 'companies'}
            <div class="form-group">
              <label for="company-input">Companies</label>
              <div class="companies-input">
                <div class="companies-list">
                  {#each newCompanies as company}
                    <span class="company">
                      {company}
                      <button type="button" class="remove" on:click={() => removeCompany(company)}>×</button>
                    </span>
                  {/each}
                </div>
                <div class="add-company">
                  <input
                    id="company-input"
                    type="text"
                    bind:value={newCompany}
                    placeholder="Add a company"
                    on:keydown={(e) => e.key === 'Enter' && (e.preventDefault(), addCompany())}
                  />
                  <button type="button" on:click={addCompany}>Add</button>
                </div>
              </div>
            </div>
          {:else}
            <div class="form-group">
              <label for="text-value">Value</label>
              <input
                id="text-value"
                type="text"
                bind:value={newValue as string}
                placeholder="Enter a value"
                required
              />
            </div>
          {/if}

          <div class="modal-actions">
            <button type="button" class="secondary" on:click={() => showModal = false}>
              Cancel
            </button>
            <button type="submit" class="primary">
              {editingCondition ? 'Save Changes' : 'Add Condition'}
            </button>
          </div>
        </form>
      {/if}
    </div>
  </div>
{/if}

<style>
  .query-pills {
    display: flex;
    flex-wrap: wrap;
    gap: 0.75rem;
    padding: 1rem;
    background: #f8fafc;
    border-radius: 0.5rem;
    min-height: 100px;
  }

  .pill {
    display: flex;
    align-items: flex-start;
    gap: 0.75rem;
    padding: 0.75rem;
    background: #f1f5f9;
    border-radius: 0.5rem;
    font-size: 0.875rem;
    min-width: 200px;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .pill:hover {
    background: #e2e8f0;
  }

  .pill.boolean {
    background: #e0f2fe;
  }

  .pill.boolean:hover {
    background: #bae6fd;
  }

  .pill.range {
    background: #f0fdf4;
  }

  .pill.range:hover {
    background: #dcfce7;
  }

  .pill.companies {
    background: #f8fafc;
  }

  .pill.companies:hover {
    background: #f1f5f9;
  }

  .pill-content {
    flex: 1;
    min-width: 0;
  }

  .label {
    display: block;
    font-weight: 500;
    color: #475569;
    margin-bottom: 0.25rem;
  }

  .value {
    color: #64748b;
  }

  .companies-list {
    display: flex;
    flex-wrap: wrap;
    gap: 0.25rem;
    margin-top: 0.25rem;
  }

  .company {
    background: #e2e8f0;
    padding: 0.125rem 0.375rem;
    border-radius: 0.25rem;
    font-size: 0.75rem;
    color: #475569;
    display: flex;
    align-items: center;
    gap: 0.25rem;
  }

  .company .remove {
    font-size: 1rem;
    color: #64748b;
    padding: 0;
    background: none;
    border: none;
    cursor: pointer;
    line-height: 1;
  }

  .company .remove:hover {
    color: #ef4444;
  }

  .more {
    font-size: 0.75rem;
    color: #64748b;
    margin-left: 0.25rem;
  }

  .remove {
    background: none;
    border: none;
    color: #94a3b8;
    font-size: 1.25rem;
    padding: 0;
    cursor: pointer;
    line-height: 1;
    transition: color 0.2s ease;
  }

  .remove:hover {
    color: #ef4444;
  }

  .add {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.75rem;
    background: #f1f5f9;
    border: 2px dashed #cbd5e1;
    border-radius: 0.5rem;
    color: #64748b;
    font-size: 0.875rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .add:hover {
    background: #e2e8f0;
    border-color: #94a3b8;
    color: #475569;
  }

  .add span {
    font-size: 1.25rem;
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

  .type-selection {
    margin-bottom: 1.5rem;
  }

  .type-selection h3 {
    margin: 0 0 1rem;
    color: #475569;
    font-size: 1rem;
  }

  .type-options {
    display: grid;
    gap: 0.75rem;
  }

  .type-option {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 0.5rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .type-option:hover {
    background: #f1f5f9;
    border-color: #cbd5e1;
  }

  .type-option .icon {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 2rem;
    height: 2rem;
    background: #e2e8f0;
    border-radius: 0.375rem;
    font-size: 1.25rem;
    color: #475569;
  }

  .type-info {
    text-align: left;
  }

  .type-info strong {
    display: block;
    color: #1e293b;
    margin-bottom: 0.25rem;
  }

  .type-info span {
    color: #64748b;
    font-size: 0.875rem;
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

  .form-group input[type="text"],
  .form-group input[type="number"] {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    font-size: 0.875rem;
  }

  .boolean-input {
    display: flex;
    align-items: center;
    gap: 1rem;
  }

  .toggle {
    position: relative;
    display: inline-block;
    width: 3rem;
    height: 1.5rem;
  }

  .toggle input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #cbd5e1;
    transition: .4s;
    border-radius: 1.5rem;
  }

  .slider:before {
    position: absolute;
    content: "";
    height: 1.25rem;
    width: 1.25rem;
    left: 0.125rem;
    bottom: 0.125rem;
    background-color: white;
    transition: .4s;
    border-radius: 50%;
  }

  input:checked + .slider {
    background-color: #3b82f6;
  }

  input:checked + .slider:before {
    transform: translateX(1.5rem);
  }

  .range-inputs {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }

  .companies-input {
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
  }

  .companies-input .companies-list {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    padding: 0.5rem;
    background: #f8fafc;
    border: 1px solid #e2e8f0;
    border-radius: 0.375rem;
    min-height: 2.5rem;
  }

  .add-company {
    display: flex;
    gap: 0.5rem;
  }

  .add-company input {
    flex: 1;
  }

  .add-company button {
    padding: 0.5rem 1rem;
    background: #3b82f6;
    color: white;
    border: none;
    border-radius: 0.375rem;
    font-size: 0.875rem;
    font-weight: 500;
    cursor: pointer;
  }

  .add-company button:hover {
    background: #2563eb;
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