<script lang="ts">
  import {
    formatCurrency,
    formatCurrencyDetailed,
    formatPercent,
  } from "../utils/formatters";
  import AmortizationTable from "./AmortizationTable.svelte";

  let loanAmount = $state(640000);
  let interestRate = $state(5.6);
  let loanTerm = $state(30);
  let downPayment = $state(40000);
  let propertyTax = $state(6400);
  let propertyTaxPercent = $state(1.17);
  let isPropertyTaxFixed = $state(false);
  let homeInsurance = $state(3000);
  let pmi = $state(0);
  let hoa = $state(277);

  // Ensure down payment never exceeds loan amount
  $effect(() => {
    if (downPayment > loanAmount) {
      downPayment = loanAmount;
    }
  });

  let principal = $derived(loanAmount - downPayment);
  let monthlyInterestRate = $derived(interestRate / 100 / 12);
  let numberOfPayments = $derived(loanTerm * 12);

  let monthlyPrincipalInterest = $derived(
    principal > 0 && monthlyInterestRate > 0
      ? (principal *
          monthlyInterestRate *
          Math.pow(1 + monthlyInterestRate, numberOfPayments)) /
          (Math.pow(1 + monthlyInterestRate, numberOfPayments) - 1)
      : 0,
  );

  let annualPropertyTax = $derived(
    isPropertyTaxFixed ? propertyTax : (loanAmount * propertyTaxPercent) / 100,
  );
  let monthlyPropertyTax = $derived(annualPropertyTax / 12);
  let monthlyInsurance = $derived(homeInsurance / 12);
  let totalMonthlyPayment = $derived(
    monthlyPrincipalInterest +
      monthlyPropertyTax +
      monthlyInsurance +
      pmi +
      hoa,
  );

  let totalInterestPaid = $derived(
    monthlyPrincipalInterest * numberOfPayments - principal,
  );
  let totalPaid = $derived(monthlyPrincipalInterest * numberOfPayments);

  // Calculate current LTV (Loan-to-Value)
  let currentLTV = $derived(principal / loanAmount);

  // Check if PMI is currently applicable (LTV > 80%)
  let isPMIApplicable = $derived(currentLTV > 0.8);

  // Calculate when PMI stops (at 78% LTV or lower)
  let monthPMIStops = $derived.by(() => {
    if (pmi === 0 || !isPMIApplicable) return 0;

    let balance = principal;
    const initialLTV = principal / loanAmount;

    // If already at 78% LTV or lower, PMI doesn't apply
    if (initialLTV <= 0.78) return 0;

    for (let i = 1; i <= numberOfPayments; i++) {
      const interestPayment = balance * monthlyInterestRate;
      const principalPayment = monthlyPrincipalInterest - interestPayment;
      balance -= principalPayment;

      const ltv = balance / loanAmount;
      if (ltv <= 0.78) {
        return i;
      }
    }
    return numberOfPayments; // PMI for full term if never reaches 78% LTV
  });

  let amortizationSchedule = $derived.by(() => {
    const schedule = [];
    let balance = principal;

    for (let i = 1; i <= numberOfPayments; i++) {
      const interestPayment = balance * monthlyInterestRate;
      const principalPayment = monthlyPrincipalInterest - interestPayment;
      balance -= principalPayment;

      schedule.push({
        month: i,
        payment: monthlyPrincipalInterest,
        principal: principalPayment,
        interest: interestPayment,
        balance: Math.max(0, balance),
      });
    }

    return schedule;
  });
</script>

<div class="w-full max-w-7xl mx-auto md:py-8">
  <div
    class="md:bg-white grid grid-cols-1 md:grid-cols-2 gap-8 md:p-8 md:rounded-xl md:shadow-lg"
  >
    <div>
      <h2 class="text-2xl font-semibold mb-6 text-gray-800">Loan Details</h2>

      <div class="mb-6">
        <label
          for="loan-amount"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Home Price</span>
          <span class="text-gray-900"
            >$
            <!-- field-sizing-content tailwind property needs to be looked at again once webkit supports it -->
            <input
              id="loan-amount-input"
              type="number"
              inputmode="numeric"
              bind:value={loanAmount}
              class="w-32 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="loan-amount"
          type="range"
          min="50000"
          max="2000000"
          step="10000"
          bind:value={loanAmount}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="down-payment"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Down Payment</span>
          <span class="text-gray-900"
            >$
            <input
              id="down-payment-input"
              type="number"
              inputmode="numeric"
              bind:value={downPayment}
              class="w-32 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="down-payment"
          type="range"
          min="0"
          max={loanAmount}
          step="5000"
          bind:value={downPayment}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
        <div class="text-sm text-gray-500 mt-1">
          {((downPayment / loanAmount) * 100).toFixed(1)}% of home price
        </div>
      </div>

      <div class="mb-6">
        <label
          for="interest-rate"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Interest Rate (%)</span>
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
          min="2"
          max="12"
          step="0.1"
          bind:value={interestRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="loan-term"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Loan Term</span>
          <span class="text-gray-900">{loanTerm} years</span>
        </label>
        <input
          id="loan-term"
          type="range"
          min="5"
          max="30"
          step="5"
          bind:value={loanTerm}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <h3 class="text-2xl font-semibold mt-8 mb-6 text-gray-800">
        Additional Costs
      </h3>

      <div class="mb-6">
        <div class="flex justify-between items-center mb-3">
          <label for="property-tax" class="font-medium text-gray-700 text-lg">
            Annual Property Tax
          </label>
          <label class="inline-flex items-center cursor-pointer">
            <span class="mr-2 text-sm text-gray-600"
              >{isPropertyTaxFixed ? "Fixed" : "Percentage"}</span
            >
            <input
              type="checkbox"
              bind:checked={isPropertyTaxFixed}
              class="sr-only peer"
            />
            <div
              class="relative w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 rounded-full peer peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"
            ></div>
          </label>
        </div>

        {#if isPropertyTaxFixed}
          <label
            for="property-tax-fixed"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-base text-gray-600">Annual Amount</span>
            <span class="text-gray-900"
              >$
              <input
                id="property-tax-fixed-input"
                type="number"
                inputmode="numeric"
                bind:value={propertyTax}
                class="w-24 px-2 border border-gray-300 bg-white rounded text-right"
              />
            </span>
          </label>
          <input
            id="property-tax-fixed"
            type="range"
            min="0"
            max="20000"
            step="100"
            bind:value={propertyTax}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
          />
        {:else}
          <label
            for="property-tax-percent"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-base text-gray-600">Tax Rate</span>
            <span class="text-gray-900"
              >{formatPercent(propertyTaxPercent)} ({formatCurrency(
                annualPropertyTax,
              )}/year)</span
            >
          </label>
          <input
            id="property-tax-percent"
            type="range"
            min="0"
            max="5"
            step="0.01"
            bind:value={propertyTaxPercent}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
          />
        {/if}
      </div>

      <div class="mb-6">
        <label
          for="home-insurance"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Annual Home Insurance</span>
          <span class="text-gray-900"
            >$
            <input
              id="home-insurance-input"
              type="number"
              inputmode="numeric"
              bind:value={homeInsurance}
              class="w-24 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="home-insurance"
          type="range"
          min="0"
          max="10000"
          step="100"
          bind:value={homeInsurance}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>
      <div class="mb-6">
        <label
          for="hoa"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Monthly HOA</span>
          <span class="text-gray-900"
            >$
            <input
              id="hoa-input"
              type="number"
              inputmode="numeric"
              bind:value={hoa}
              class="w-20 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="hoa"
          type="range"
          min="0"
          max="2000"
          step="1"
          bind:value={hoa}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>
      {#if isPMIApplicable}
        <div class="mb-6">
          <label
            for="pmi"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-lg">Monthly PMI</span>
            <span class="text-gray-900"
              >$
              <input
                id="pmi-input"
                type="number"
                inputmode="numeric"
                bind:value={pmi}
                class="w-20 px-2 border border-gray-300 bg-white rounded text-right"
              />
            </span>
          </label>
          <input
            id="pmi"
            type="range"
            min="0"
            max="2000"
            step="1"
            bind:value={pmi}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
          />
        </div>
      {:else}
        <div class="mb-6">
          <div
            class="p-4 text-center bg-green-50 border border-green-400 rounded-lg"
          >
            <p class="text-green-600 text-base">
              Private Mortgage Insurance (PMI) is not required because your
              Loan-to-Value (LTV) is greater than 80%
            </p>
          </div>
        </div>
      {/if}
    </div>

    <div class="sticky top-8 h-fit">
      <h2 class="text-3xl font-semibold mb-4 text-gray-800 text-center">
        Monthly Payment
      </h2>
      <div class="text-5xl font-bold text-green-600 mb-10 text-center">
        {formatCurrencyDetailed(totalMonthlyPayment)}
      </div>

      <div class="border bg-white border-gray-200 rounded-lg p-4 mb-8">
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-lg">Principal & Interest</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(monthlyPrincipalInterest)}</span
          >
        </div>
        <div class="flex justify-between py-3 border-t border-gray-100">
          <span class="text-gray-600 text-lg">Property Tax</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(monthlyPropertyTax)}</span
          >
        </div>
        <div class="flex justify-between py-3 border-t border-gray-100">
          <span class="text-gray-600 text-lg">Home Insurance</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(monthlyInsurance)}</span
          >
        </div>
        {#if hoa > 0}
          <div class="flex justify-between py-3 border-t border-gray-100">
            <span class="text-gray-600 text-lg">HOA</span>
            <span class="font-semibold text-gray-900"
              >{formatCurrencyDetailed(hoa)}</span
            >
          </div>
        {/if}
        {#if isPMIApplicable && pmi > 0}
          <div class="flex justify-between py-3 border-t border-gray-100">
            <span class="text-gray-600 text-lg">PMI</span>
            <span class="font-semibold text-gray-900"
              >{formatCurrencyDetailed(pmi)}</span
            >
          </div>
          <div class="text-sm text-gray-500">
            <p>
              PMI automatically ends after {monthPMIStops} payments when Loan-to-Value
              (LTV) reaches 78%.
            </p>
          </div>
        {/if}
      </div>

      <h3 class="text-2xl font-semibold mb-6 text-gray-800">Loan Summary</h3>
      <div class="border bg-white border-gray-200 rounded-lg p-4 mb-6">
        <div class="flex justify-between py-3 border-b border-gray-100">
          <span class="text-gray-600 text-lg">Loan Amount</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(principal)}</span
          >
        </div>
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-lg">Total Interest</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(totalInterestPaid)}</span
          >
        </div>
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-lg">Total Paid</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(totalPaid)}</span
          >
        </div>
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-lg">Total Payments</span>
          <span class="font-semibold text-gray-900">{numberOfPayments}</span>
        </div>
      </div>
    </div>
  </div>
</div>
<div class="w-full max-w-7xl mx-auto py-8 pb-8">
  <div class="md:bg-white md:rounded-xl md:shadow-lg overflow-hidden">
    <div class="md:p-8">
<!--       <AmortizationChart {amortizationSchedule} /> -->
      <AmortizationTable {amortizationSchedule} />
    </div>
  </div>
</div>


