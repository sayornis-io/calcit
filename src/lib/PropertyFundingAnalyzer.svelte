<script>
  import { formatCurrency, formatPercent } from "../utils/formatters";

  let totalProjectCost = $state(200000);
  let interestRate = $state(7.25);
  let loanTermYears = $state(5);
  let originalDistributionAmount = $state(28000);
  let distributionAmount = $state(32000);

  let partners = $state([
    { percentage: 25, isDirectFunding: true, label: "Partner 1" },
    { percentage: 25, isDirectFunding: false, label: "Partner 2" },
    { percentage: 17.5, isDirectFunding: false, label: "Partner 3" },
    { percentage: 19.3125, isDirectFunding: true, label: "Partner 4" },
    { percentage: 8.5, isDirectFunding: true, label: "Partner 5" },
    { percentage: 2.8125, isDirectFunding: true, label: "Partner 6" },
    { percentage: 1.875, isDirectFunding: true, label: "Partner 7" },
  ]);

  function addPartner() {
    partners = [
      ...partners,
      {
        percentage: 0,
        isDirectFunding: false,
        label: `Partner ${partners.length + 1}`,
      },
    ];
  }

  function removePartner(index) {
    partners = partners.filter((_, i) => i !== index);
  }

  let totalPercentage = $derived(
    partners.reduce((sum, p) => sum + (p.percentage || 0), 0),
  );

  // Calculate each partner's share and direct contributions
  let partnerShares = $derived(
    partners.map((p) => {
      const percentage = p.percentage || 0;
      const share = (percentage / 100) * totalProjectCost;
      const directContribution = p.isDirectFunding ? share : 0;
      return { ...p, share, directContribution };
    }),
  );

  let totalDirectContributions = $derived(
    partnerShares.reduce((sum, p) => sum + p.directContribution, 0),
  );
  let loanAmount = $derived(totalProjectCost - totalDirectContributions);

  let monthlyRate = $derived(interestRate / 100 / 12);
  let numPayments = $derived(loanTermYears * 12);
  let monthlyPayment = $derived(
    (loanAmount * (monthlyRate * Math.pow(1 + monthlyRate, numPayments))) /
      (Math.pow(1 + monthlyRate, numPayments) - 1),
  );
  let totalPaid = $derived(monthlyPayment * numPayments);
  let totalInterest = $derived(totalPaid - loanAmount);

  let directFundingPartners = $derived(
    partners.filter((p) => p.isDirectFunding),
  );
  let loanParticipatingPartners = $derived(
    partners.filter((p) => !p.isDirectFunding),
  );
  let participatingTotal = $derived(
    loanParticipatingPartners.reduce((sum, p) => sum + (p.percentage || 0), 0),
  );

  // Calculate suggested distribution to keep loan participants at near-original levels
  let suggestedDistribution = $derived(
    participatingTotal > 0
      ? originalDistributionAmount + monthlyPayment * (100 / participatingTotal)
      : originalDistributionAmount,
  );

  let results = $derived(
    partners.map((partner) => {
      const percentage = partner.percentage || 0;
      const share = (percentage / 100) * totalProjectCost;
      const directContribution = partner.isDirectFunding ? share : 0;
      const originalDistribution =
        (percentage / 100) * originalDistributionAmount;

      if (partner.isDirectFunding) {
        // Direct funding partner gets their normal distribution, no loan payment
        const distribution = (percentage / 100) * distributionAmount;
        const paymentReturnDeduction = distribution - originalDistribution;

        // Calculate months to payoff: investment / monthly return
        // Only calculate if there's a positive return, otherwise it will never pay off
        const monthsToPayoff =
          paymentReturnDeduction > 0
            ? directContribution / paymentReturnDeduction
            : null; // null means never pays off (no increase or decrease in distribution)

        // Calculate annualized ROI over the loan term
        // Total extra income over loan term - initial investment / initial investment / years
        let annualizedROI = null;
        if (directContribution > 0 && paymentReturnDeduction > 0) {
          const totalExtraIncome = paymentReturnDeduction * numPayments;
          const netProfit = totalExtraIncome - directContribution;
          const simpleROI = (netProfit / directContribution) * 100;
          annualizedROI = simpleROI / loanTermYears;
        } else if (directContribution > 0 && paymentReturnDeduction <= 0) {
          // Negative return
          const totalLoss = Math.abs(paymentReturnDeduction * numPayments);
          annualizedROI =
            -((totalLoss / directContribution) * 100) / loanTermYears;
        }

        return {
          ...partner,
          share: share,
          directContribution: directContribution,
          originalDistribution: originalDistribution,
          adjustedPercentage: 0,
          monthlyPayment: 0,
          paymentReturnDeduction: paymentReturnDeduction,
          monthsToPayoff: monthsToPayoff,
          annualizedROI: annualizedROI,
          netDistribution: distribution,
        };
      } else {
        // Loan participating partners
        const directFundingDistribution = directFundingPartners.reduce(
          (sum, p) => sum + ((p.percentage || 0) / 100) * distributionAmount,
          0,
        );
        const remainingDistribution =
          distributionAmount - directFundingDistribution;
        const adjustedPercentage =
          participatingTotal > 0 ? (percentage / participatingTotal) * 100 : 0;
        const distribution = (adjustedPercentage / 100) * remainingDistribution;
        const monthlyPaymentShare = (adjustedPercentage / 100) * monthlyPayment;
        const netDistribution = distribution - monthlyPaymentShare;

        return {
          ...partner,
          share: share,
          directContribution: directContribution,
          originalDistribution: originalDistribution,
          adjustedPercentage: adjustedPercentage,
          monthlyPayment: monthlyPaymentShare,
          paymentReturnDeduction: -monthlyPaymentShare,
          monthsToPayoff: null, // N/A for loan participants
          annualizedROI: null, // N/A for loan participants
          netDistribution: netDistribution,
        };
      }
    }),
  );
</script>

<div class="w-full max-w-7xl mx-auto md:px-4 md:py-8 pb-8">
  <!-- Input Section -->
  <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
    <!-- Loan Details -->
    <div class="bg-white rounded-lg shadow p-6">
      <h2 class="text-xl font-semibold text-gray-800 mb-4">
        Project & Loan Details
      </h2>

      <div class="space-y-4">
        <div class="mb-6">
          <label
            for="totalProjectCost"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-lg">Total Project Cost</span>
            <span class="text-gray-900"
              >$
              <input
                id="totalProjectCost-input"
                type="number"
                inputmode="numeric"
                bind:value={totalProjectCost}
                class="w-32 px-2 border border-gray-300 bg-white rounded text-right"
              />
            </span>
          </label>
          <input
            id="totalProjectCost"
            type="range"
            min="10000"
            max="5000000"
            step="10000"
            bind:value={totalProjectCost}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
          />
        </div>
        <div>
          <label
            for="interestRate"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-lg">Interest Rate</span>
            <input
              id="interestRate-input"
              type="number"
              inputmode="decimal"
              step="0.1"
              bind:value={interestRate}
              class="w-20 px-2 border border-gray-300 bg-white rounded text-right mr-1"
            />
          </label>
        </div>

        <div>
          <label
            for="loanTermYears"
            class="flex justify-between items-center mb-6 font-medium text-gray-700"
          >
            <span class="text-lg">Loan Term (Years)</span>
            <input
              id="loanTermYears-input"
              type="number"
              inputmode="numeric"
              step="1"
              bind:value={loanTermYears}
              class="w-18 px-2 border border-gray-300 bg-white rounded text-right mr-1"
            />
          </label>
        </div>

        <h2 class="text-xl font-semibold text-gray-800 mb-4">
          Distribution Amounts
        </h2>

        <div>
          <label
            for="originalDistributionAmount"
            class="text-base text-gray-700 mb-1"
            >Original Distribution (Before Loan) ($)</label
          >
          <input
            id="originalDistributionAmount"
            type="number"
            bind:value={originalDistributionAmount}
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
          />
        </div>

        <div>
          <label
            for="suggestedDistribution"
            class="text-base text-gray-700 mb-1"
            >Suggested Distribution (To Maintain Loan Participant Levels)</label
          >
          <div class="flex gap-2">
            <input
              id="suggestedDistribution"
              type="number"
              value={suggestedDistribution.toFixed(2)}
              readonly
              class="flex-1 px-3 py-2 border border-gray-300 rounded-md bg-gray-100 text-gray-700"
            />
            <button
              onclick={() =>
                (distributionAmount =
                  Math.round(suggestedDistribution * 100) / 100)}
              class="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 text-base whitespace-nowrap"
            >
              Use This
            </button>
          </div>
          <p class="text-sm text-gray-500 mt-1">
            This amount keeps loan participants near their original distribution
            levels after loan payments
          </p>
        </div>

        <div>
          <label for="distributionAmount" class="text-base text-gray-700 mb-1"
            >New Distribution Amount ($)</label
          >
          <input
            id="distributionAmount"
            type="number"
            bind:value={distributionAmount}
            class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
          />
        </div>
      </div>
    </div>

    <!-- Partners -->
    <div class="bg-white rounded-lg shadow p-6">
      <div class="flex items-center justify-between mb-4">
        <h2 class="text-xl font-semibold text-gray-800">Partners</h2>
        <button
          onclick={addPartner}
          class="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 text-sm font-medium"
        >
          Add Partner
        </button>
      </div>

      <div class="space-y-3 max-h-96 overflow-y-auto">
        {#each partners as partner, index}
          <div class="flex items-center gap-3 p-3 bg-gray-50 rounded-md">
            <input
              type="text"
              bind:value={partner.label}
              class="flex-1 px-3 py-2 border border-gray-300 rounded-md text-sm"
              placeholder="Partner name"
            />
            <input
              type="number"
              step="0.0001"
              bind:value={partner.percentage}
              class="w-24 px-3 py-2 border border-gray-300 rounded-md text-sm"
              placeholder="%"
            />
            <label class="flex items-center gap-2 text-sm whitespace-nowrap">
              <input
                type="checkbox"
                bind:checked={partner.isDirectFunding}
                class="rounded border-gray-300"
              />
              <span class="text-gray-700">Direct Funding</span>
            </label>
            <button
              onclick={() => removePartner(index)}
              class="px-3 py-2 bg-red-500 text-white rounded-md hover:bg-red-600 text-sm"
              disabled={partners.length === 1}
            >
              Remove
            </button>
          </div>
        {/each}
      </div>

      <div class="mt-4 p-3 bg-gray-100 rounded-md">
        <div class="flex justify-between text-sm">
          <span class="font-medium">Total Percentage:</span>
          <span
            class={totalPercentage === 100
              ? "text-green-600 font-semibold"
              : "text-red-600 font-semibold"}
          >
            {totalPercentage.toFixed(4)}%
          </span>
        </div>
      </div>
    </div>
  </div>

  <!-- Summary Section -->
  <div class="mb-8">
    <h2 class="text-2xl font-bold mb-4">Project Summary</h2>
    <!-- Project Overview - Full Width Hero Card -->
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-6">
        <div class="bg-white rounded-lg shadow p-4">
          <div class="text-sm text-gray-700 font-medium mb-1">
            Total Project Cost
          </div>
          <div class="text-3xl font-bold text-gray-800">
            {formatCurrency(totalProjectCost)}
          </div>
        </div>
        <div class="bg-white rounded-lg shadow p-4">
          <div class="text-sm text-gray-700 font-medium mb-1">
            Direct Funding
          </div>
          <div class="text-3xl font-bold text-green-600">
            {formatCurrency(totalDirectContributions)}
          </div>
          <div class="text-xs text-gray-600 mt-1">
            {formatPercent((totalDirectContributions / totalProjectCost) * 100)}
            of total
          </div>
        </div>
        <div class="bg-white rounded-lg shadow p-4">
          <div class="text-sm text-gray-700 font-medium mb-1">
            Loan Required
          </div>
          <div class="text-3xl font-bold text-red-600">
            {formatCurrency(loanAmount)}
          </div>
          <div class="text-xs text-gray-600 mt-1">
            {formatPercent((loanAmount / totalProjectCost) * 100)} of total
          </div>
        </div>
      </div>

    <!-- Detailed Breakdown - Two Columns -->
    <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
      <!-- Loan Impact -->
      <div class="bg-white rounded-lg shadow p-6">
        <div class="flex items-center gap-2 mb-4">
          <div class="w-1 h-6 bg-orange-500 rounded"></div>
          <h3 class="text-lg font-semibold text-gray-800">Loan Analysis</h3>
        </div>
        <div class="space-y-4">
          <div class="bg-gray-100 rounded-lg p-4">
            <div class="text-xs uppercase tracking-wide text-gray-600 mb-1">
              Monthly Payment
            </div>
            <div class="text-2xl font-bold text-orange-600">
              {formatCurrency(monthlyPayment)}
            </div>
            <div class="text-xs text-gray-600 mt-1">
              Shared by {loanParticipatingPartners.length} loan participant{loanParticipatingPartners.length !==
              1
                ? "s"
                : ""}
            </div>
          </div>

          <div class="grid grid-cols-2 gap-4">
            <div>
              <div class="text-xs text-gray-500 mb-1">Interest Rate</div>
              <div class="text-lg font-semibold text-gray-900">
                {formatPercent(interestRate)}
              </div>
            </div>
            <div>
              <div class="text-xs text-gray-500 mb-1">Loan Term</div>
              <div class="text-lg font-semibold text-gray-900">
                {loanTermYears} years
              </div>
            </div>
          </div>

          <div class="pt-4 border-t border-gray-200">
            <div class="flex justify-between text-sm mb-1">
              <span class="text-gray-600">Total Repayment</span>
              <span class="font-semibold text-gray-900"
                >{formatCurrency(totalPaid)}</span
              >
            </div>
            <div class="flex justify-between text-sm">
              <span class="text-gray-600">Total Interest</span>
              <span class="font-semibold text-red-600"
                >{formatCurrency(totalInterest)}</span
              >
            </div>
          </div>
        </div>
      </div>

      <!-- Distribution Impact -->
      <div class="bg-white rounded-lg shadow p-6">
        <div class="flex items-center gap-2 mb-4">
          <div class="w-1 h-6 bg-blue-500 rounded"></div>
          <h3 class="text-lg font-semibold text-gray-800">
            Distribution Analysis
          </h3>
        </div>
        <div class="space-y-4">
          <div class="grid grid-cols-2 gap-4">
            <div class="bg-gray-100 rounded-lg p-3">
              <div class="text-xs text-gray-600 mb-1">Direct Funders</div>
              <div class="text-2xl font-bold text-green-600">
                {directFundingPartners.length}
              </div>
              <div class="text-xs text-gray-600 mt-1">
                Receive increased distributions
              </div>
            </div>
            <div class="bg-gray-100 rounded-lg p-3">
              <div class="text-xs text-gray-600 mb-1">Loan Participants</div>
              <div class="text-2xl font-bold text-green-600">
                {loanParticipatingPartners.length}
              </div>
              <div class="text-xs text-gray-600 mt-1">
                Pay monthly loan portion
              </div>
            </div>
          </div>

          <div
            class="bg-gray-100 rounded-lg p-4"
          >
            <div class="flex justify-between items-start mb-3">
              <div>
                <div class="text-xs uppercase tracking-wide text-gray-600 mb-1">
                  Original Distribution
                </div>
                <div class="text-xl font-semibold text-gray-700">
                  {formatCurrency(originalDistributionAmount)}
                </div>
              </div>
              <div class="text-right">
                <div class="text-xs uppercase tracking-wide text-gray-600 mb-1">
                  New Distribution
                </div>
                <div class="text-xl font-semibold text-blue-600">
                  {formatCurrency(distributionAmount)}
                </div>
              </div>
            </div>
            <div class="pt-3 border-t border-gray-300">
              <div class="flex justify-between items-center">
                <span class="text-xs font-medium text-gray-600">Net Change</span
                >
                <span
                  class={distributionAmount >= originalDistributionAmount
                    ? "text-lg font-bold text-green-600"
                    : "text-lg font-bold text-red-600"}
                >
                  {distributionAmount >= originalDistributionAmount
                    ? "+"
                    : ""}{formatCurrency(
                    distributionAmount - originalDistributionAmount,
                  )}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Results Section -->
  <div class="bg-white rounded-lg shadow overflow-hidden">
    <div class="px-6 py-4 bg-white border-b border-gray-200">
      <div class="flex items-center gap-2">
        <div class="w-1 h-6 bg-blue-500 rounded"></div>
        <h2 class="text-xl font-semibold text-gray-800">Individual Partner Analysis</h2>
      </div>
    </div>

    <div class="overflow-x-auto">
      <table class="w-full">
        <thead class="bg-gray-100">
          <tr>
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Partner</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Equity %</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Project Share</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Direct Contribution</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Status</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Original Distribution</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Loan Share %</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Payment Return/Deduction</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Months to Payoff</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >ROI (Annual %)</th
            >
            <th
              class="px-6 py-3 text-left text-xs font-medium text-gray-700 uppercase tracking-wider"
              >Net Distribution</th
            >
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-200">
          {#each results as result}
            <tr class="hover:bg-gray-50">
              <td
                class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900"
              >
                {result.label}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
                {formatPercent(result.percentage)}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
                {formatCurrency(result.share)}
              </td>
              <td
                class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
              >
                {result.isDirectFunding
                  ? formatCurrency(result.directContribution)
                  : "-"}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm">
                {#if result.isDirectFunding}
                  <span
                    class="px-2 py-1 text-xs font-semibold rounded-full bg-green-100 text-green-800"
                  >
                    Direct Funding
                  </span>
                {:else}
                  <span
                    class="px-2 py-1 text-xs font-semibold rounded-full bg-blue-100 text-blue-800"
                  >
                    Loan Participant
                  </span>
                {/if}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                {formatCurrency(result.originalDistribution)}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
                {result.isDirectFunding
                  ? "N/A"
                  : formatPercent(result.adjustedPercentage)}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold">
                <span
                  class={result.paymentReturnDeduction >= 0
                    ? "text-green-600"
                    : "text-red-600"}
                >
                  {formatCurrency(result.paymentReturnDeduction)}
                </span>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-700">
                {#if result.monthsToPayoff !== null}
                  {#if result.monthsToPayoff === Infinity || result.paymentReturnDeduction <= 0}
                    <span class="text-red-600 font-semibold">Never</span>
                  {:else}
                    {result.monthsToPayoff.toFixed(1)} months ({(
                      result.monthsToPayoff / 12
                    ).toFixed(1)} years)
                  {/if}
                {:else}
                  <span class="text-gray-400">N/A</span>
                {/if}
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-semibold">
                {#if result.annualizedROI !== null}
                  <span
                    class={result.annualizedROI >= 0
                      ? "text-green-600"
                      : "text-red-600"}
                  >
                    {result.annualizedROI.toFixed(2)}%
                  </span>
                {:else}
                  <span class="text-gray-400">N/A</span>
                {/if}
              </td>
              <td
                class="px-6 py-4 whitespace-nowrap text-sm font-semibold text-gray-900"
              >
                {formatCurrency(result.netDistribution)}
              </td>
            </tr>
          {/each}
          <tr class="bg-gray-100 font-semibold">
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              Total
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatPercent(results.reduce((sum, r) => sum + r.percentage, 0))}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatCurrency(totalProjectCost)}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatCurrency(totalDirectContributions)}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900"> </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatCurrency(originalDistributionAmount)}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatPercent(
                results
                  .filter((r) => !r.isDirectFunding)
                  .reduce((sum, r) => sum + r.adjustedPercentage, 0),
              )}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatCurrency(
                results.reduce((sum, r) => sum + r.paymentReturnDeduction, 0),
              )}
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              -
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              -
            </td>
            <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
              {formatCurrency(
                results.reduce((sum, r) => sum + r.netDistribution, 0),
              )}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</div>
