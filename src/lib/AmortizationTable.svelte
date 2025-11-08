<script lang="ts">
  import { formatCurrencyDetailed } from '../utils/formatters';

  interface AmortizationRow {
    month: number;
    payment: number;
    principal: number;
    interest: number;
    balance: number;
  }

  let { amortizationSchedule = [] }: { amortizationSchedule: AmortizationRow[] } = $props();

  let selectedYear = $state(1);

  // Calculate the number of years available
  let totalYears = $derived(Math.ceil(amortizationSchedule.length / 12));

  // Generate array of years for the dropdown
  let years = $derived(Array.from({ length: totalYears }, (_, i) => i + 1));

  // Get the rows for the selected year
  let startMonth = $derived((selectedYear - 1) * 12);
  let endMonth = $derived(startMonth + 12);
  let yearRows = $derived(amortizationSchedule.slice(startMonth, endMonth));
</script>

<div class="w-full max-w-7xl mx-auto md:px-4 md:py-8 pb-8">
  <div class="bg-white rounded-xl shadow-lg overflow-hidden">
    <div class="p-8">
      <div class="flex justify-between items-center mb-6">
        <h2 class="text-2xl font-semibold text-gray-800">Amortization Schedule</h2>
        {#if years.length > 0}
          <div class="flex items-center gap-2">
            <label for="year-select" class="text-sm font-medium text-gray-700">Year:</label>
            <select
              id="year-select"
              bind:value={selectedYear}
              class="px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              {#each years as year (year)}
                <option value={year}>{year}</option>
              {/each}
            </select>
          </div>
        {/if}
      </div>

      <div class="overflow-x-auto">
        <table class="w-full">
          <thead>
            <tr class="border-b-2 border-gray-300 bg-gray-50">
              <th class="px-4 py-3 text-left text-sm font-semibold text-gray-700">Month</th>
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700">Payment</th>
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700">Principal</th>
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700">Interest</th>
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700">Balance</th>
            </tr>
          </thead>
          <tbody>
            {#each yearRows as row (row.month)}
              <tr class="border-b border-gray-100 hover:bg-gray-50 transition-colors">
                <td class="px-4 py-3 text-sm text-gray-900">{row.month}</td>
                <td class="px-4 py-3 text-sm text-right text-gray-900 font-medium">
                  {formatCurrencyDetailed(row.payment)}
                </td>
                <td class="px-4 py-3 text-sm text-right text-gray-900 font-medium">
                  {formatCurrencyDetailed(row.principal)}
                </td>
                <td class="px-4 py-3 text-sm text-right text-gray-900 font-medium">
                  {formatCurrencyDetailed(row.interest)}
                </td>
                <td class="px-4 py-3 text-sm text-right text-gray-900 font-medium">
                  {formatCurrencyDetailed(row.balance)}
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>

      {#if yearRows.length === 0}
        <div class="text-center py-8">
          <p class="text-gray-500">No amortization data available</p>
        </div>
      {/if}
    </div>
  </div>
</div>

<style>
  table {
    border-collapse: collapse;
  }
</style>
