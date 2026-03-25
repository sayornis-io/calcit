<script lang="ts">
  import {
    formatCurrency,
    formatCurrencyDetailed,
    formatPercent,
  } from "../utils/formatters";

  let principal = $state(10000);
  let interestRate = $state(5.5);
  let years = $state(10);
  let monthlyContribution = $state(100);
  let compoundingFrequency = $state<"annually" | "semiannually" | "quarterly" | "monthly" | "daily">("monthly");

  const compoundingMap = {
    annually: 1,
    semiannually: 2,
    quarterly: 4,
    monthly: 12,
    daily: 365,
  };

  let compoundsPerYear = $derived(compoundingMap[compoundingFrequency]);

  // Calculate future value with compound interest and regular contributions
  let futureValue = $derived.by(() => {
    const r = interestRate / 100;
    const n = compoundsPerYear;
    const t = years;
    const P = principal;
    const PMT = monthlyContribution;

    // Future value of principal with compound interest
    const fvPrincipal = P * Math.pow(1 + r / n, n * t);

    // Future value of monthly contributions (annuity)
    // We need to adjust for monthly contributions with different compounding frequencies
    const monthlyRate = r / 12;
    const totalMonths = t * 12;
    
    let fvContributions = 0;
    if (PMT > 0 && monthlyRate > 0) {
      fvContributions = PMT * ((Math.pow(1 + monthlyRate, totalMonths) - 1) / monthlyRate);
    } else if (PMT > 0) {
      fvContributions = PMT * totalMonths;
    }

    return fvPrincipal + fvContributions;
  });

  let totalContributions = $derived(principal + monthlyContribution * years * 12);
  let totalInterest = $derived(futureValue - totalContributions);
  let effectiveAnnualRate = $derived(
    Math.pow(1 + interestRate / 100 / compoundsPerYear, compoundsPerYear) - 1
  );

  // Calculate year-by-year breakdown
  let yearlyBreakdown = $derived.by(() => {
    const breakdown = [];
    const r = interestRate / 100;
    const monthlyRate = r / 12;
    
    let balance = principal;
    let totalContributionsSoFar = principal;

    for (let year = 1; year <= years; year++) {
      let yearStartBalance = balance;
      let yearContributions = 0;
      let yearInterest = 0;

      // Calculate month by month for this year
      for (let month = 1; month <= 12; month++) {
        const interestThisMonth = balance * monthlyRate;
        balance += interestThisMonth + monthlyContribution;
        yearInterest += interestThisMonth;
        yearContributions += monthlyContribution;
      }

      totalContributionsSoFar += yearContributions;

      breakdown.push({
        year,
        startBalance: yearStartBalance,
        contributions: yearContributions,
        interest: yearInterest,
        endBalance: balance,
        totalContributions: totalContributionsSoFar,
        totalInterest: balance - totalContributionsSoFar,
      });
    }

    return breakdown;
  });
</script>

<div class="w-full max-w-7xl mx-auto md:py-8">
  <div
    class="md:bg-white grid grid-cols-1 md:grid-cols-2 gap-8 md:p-8 md:rounded-xl md:shadow-lg"
  >
    <div>
      <h2 class="text-2xl font-semibold mb-6 text-gray-800">
        Investment Details
      </h2>

      <div class="mb-6">
        <label
          for="principal"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Initial Investment</span>
          <span class="text-gray-900"
            >$
            <input
              id="principal-input"
              type="number"
              inputmode="numeric"
              bind:value={principal}
              class="w-32 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="principal"
          type="range"
          min="0"
          max="100000"
          step="1000"
          bind:value={principal}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="monthly-contribution"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Monthly Contribution</span>
          <span class="text-gray-900"
            >$
            <input
              id="monthly-contribution-input"
              type="number"
              inputmode="numeric"
              bind:value={monthlyContribution}
              class="w-32 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="monthly-contribution"
          type="range"
          min="0"
          max="5000"
          step="50"
          bind:value={monthlyContribution}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="interest-rate"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Annual Interest Rate (%)</span>
          <input
            id="interest-rate-input"
            type="number"
            inputmode="decimal"
            step="0.1"
            bind:value={interestRate}
            class="w-18 px-2 border border-gray-300 bg-white rounded text-right mr-1"
          />
        </label>
        <input
          id="interest-rate"
          type="range"
          min="0"
          max="20"
          step="0.1"
          bind:value={interestRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="years"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Time Period</span>
          <span class="text-gray-900">{years} years</span>
        </label>
        <input
          id="years"
          type="range"
          min="1"
          max="50"
          step="1"
          bind:value={years}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label for="compounding" class="font-medium text-gray-700 text-lg mb-2 block">
          Compounding Frequency
        </label>
        <select
          id="compounding"
          bind:value={compoundingFrequency}
          class="w-full px-4 py-2 border border-gray-300 bg-white rounded-lg text-gray-900 focus:ring-2 focus:ring-blue-500 focus:border-transparent"
        >
          <option value="annually">Annually</option>
          <option value="semiannually">Semi-annually</option>
          <option value="quarterly">Quarterly</option>
          <option value="monthly">Monthly</option>
          <option value="daily">Daily</option>
        </select>
      </div>

      <div class="bg-blue-50 border border-blue-200 rounded-lg p-4">
        <h3 class="font-semibold text-gray-800 mb-2">Quick Tips</h3>
        <ul class="text-sm text-gray-600 space-y-1">
          <li>• Higher compounding frequency = more interest earned</li>
          <li>• Regular contributions dramatically increase growth</li>
          <li>• Time in the market is your greatest advantage</li>
        </ul>
      </div>
    </div>

    <div class="sticky top-8 h-fit">
      <h2 class="text-3xl font-semibold mb-4 text-gray-800 text-center">
        Future Value
      </h2>
      <div class="text-5xl font-bold text-green-600 mb-10 text-center">
        {formatCurrencyDetailed(futureValue)}
      </div>

      <div class="border bg-white border-gray-200 rounded-lg p-4 mb-8">
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-lg">Total Contributions</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(totalContributions)}</span
          >
        </div>
        <div class="flex justify-between py-3 border-t border-gray-100">
          <span class="text-gray-600 text-lg">Total Interest Earned</span>
          <span class="font-semibold text-green-600"
            >{formatCurrencyDetailed(totalInterest)}</span
          >
        </div>
        <div class="flex justify-between py-3 border-t border-gray-100">
          <span class="text-gray-600 text-lg">Effective Annual Rate</span>
          <span class="font-semibold text-gray-900"
            >{formatPercent(effectiveAnnualRate * 100)}</span
          >
        </div>
      </div>

      <div class="bg-linear-to-r from-green-50 to-blue-50 border border-gray-200 rounded-lg p-6">
        <h3 class="text-xl font-semibold mb-4 text-gray-800">
          Growth Breakdown
        </h3>
        <div class="space-y-3">
          <div>
            <div class="flex justify-between text-sm mb-1">
              <span class="text-gray-600">Contributions</span>
              <span class="text-gray-900"
                >{((totalContributions / futureValue) * 100).toFixed(1)}%</span
              >
            </div>
            <div class="w-full bg-gray-200 rounded-full h-3">
              <div
                class="bg-blue-500 h-3 rounded-full transition-all duration-500"
                style="width: {(totalContributions / futureValue) * 100}%"
              ></div>
            </div>
          </div>
          <div>
            <div class="flex justify-between text-sm mb-1">
              <span class="text-gray-600">Interest Earned</span>
              <span class="text-gray-900"
                >{((totalInterest / futureValue) * 100).toFixed(1)}%</span
              >
            </div>
            <div class="w-full bg-gray-200 rounded-full h-3">
              <div
                class="bg-green-500 h-3 rounded-full transition-all duration-500"
                style="width: {(totalInterest / futureValue) * 100}%"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<div class="w-full max-w-7xl mx-auto py-8 pb-8">
  <div class="md:bg-white md:rounded-xl md:shadow-lg overflow-hidden">
    <div class="md:p-8">
      <h2 class="text-2xl font-semibold mb-6 text-gray-800">
        Year-by-Year Growth
      </h2>
      <div class="overflow-x-auto">
        <table class="w-full border-collapse">
          <thead>
            <tr class="bg-gray-100 border-b-2 border-gray-300">
              <th class="px-4 py-3 text-left text-sm font-semibold text-gray-700"
                >Year</th
              >
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700"
                >Start Balance</th
              >
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700"
                >Contributions</th
              >
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700"
                >Interest</th
              >
              <th class="px-4 py-3 text-right text-sm font-semibold text-gray-700"
                >End Balance</th
              >
            </tr>
          </thead>
          <tbody>
            {#each yearlyBreakdown as row}
              <tr class="border-b border-gray-200 hover:bg-gray-50">
                <td class="px-4 py-3 text-sm text-gray-900">{row.year}</td>
                <td class="px-4 py-3 text-sm text-right text-gray-900"
                  >{formatCurrency(row.startBalance)}</td
                >
                <td class="px-4 py-3 text-sm text-right text-blue-600"
                  >{formatCurrency(row.contributions)}</td
                >
                <td class="px-4 py-3 text-sm text-right text-green-600"
                  >{formatCurrency(row.interest)}</td
                >
                <td class="px-4 py-3 text-sm text-right font-semibold text-gray-900"
                  >{formatCurrency(row.endBalance)}</td
                >
              </tr>
            {/each}
          </tbody>
          <tfoot>
            <tr class="bg-gray-100 font-semibold border-t-2 border-gray-300">
              <td class="px-4 py-3 text-sm text-gray-900">Total</td>
              <td class="px-4 py-3 text-sm text-right text-gray-900">—</td>
              <td class="px-4 py-3 text-sm text-right text-blue-600"
                >{formatCurrency(totalContributions)}</td
              >
              <td class="px-4 py-3 text-sm text-right text-green-600"
                >{formatCurrency(totalInterest)}</td
              >
              <td class="px-4 py-3 text-sm text-right text-gray-900"
                >{formatCurrency(futureValue)}</td
              >
            </tr>
          </tfoot>
        </table>
      </div>
    </div>
  </div>
</div>
