<script lang="ts">
  import { formatCurrency, formatCurrencyDetailed, formatPercent } from '../utils/formatters';

  let loanAmount = $state(650000);
  let interestRate = $state(5.8);
  let loanTerm = $state(30);
  let downPayment = $state(100000);
  let propertyTax = $state(6500);
  let propertyTaxPercent = $state(1.17);
  let isPropertyTaxFixed = $state(false);
  let homeInsurance = $state(3000);
  let pmi = $state(0);
  let hoa = $state(200);

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
      : 0
  );

  let annualPropertyTax = $derived(
    isPropertyTaxFixed
      ? propertyTax
      : (loanAmount * propertyTaxPercent) / 100
  );
  let monthlyPropertyTax = $derived(annualPropertyTax / 12);
  let monthlyInsurance = $derived(homeInsurance / 12);
  let totalMonthlyPayment = $derived(
    monthlyPrincipalInterest +
    monthlyPropertyTax +
    monthlyInsurance +
    pmi +
    hoa
  );

  let totalInterestPaid = $derived(
    monthlyPrincipalInterest * numberOfPayments - principal
  );
  let totalPaid = $derived(totalMonthlyPayment * numberOfPayments);
</script>

<div class="w-full max-w-7xl mx-auto md:px-4 md:py-8 pb-8">
  <div
    class="grid grid-cols-1 md:grid-cols-2 gap-8 bg-white p-8 rounded-xl shadow-lg"
  >
    <div>
      <h2 class="text-2xl font-semibold mb-6 text-gray-800">Loan Details</h2>

      <div class="mb-6">
        <label
          for="loan-amount"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-base">Home Price</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(loanAmount)}</span
          >
        </label>
        <input
          id="loan-amount"
          type="range"
          min="50000"
          max="2000000"
          step="10000"
          bind:value={loanAmount}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="down-payment"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-base">Down Payment</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(downPayment)}</span
          >
        </label>
        <input
          id="down-payment"
          type="range"
          min="0"
          max={loanAmount}
          step="5000"
          bind:value={downPayment}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
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
          <span class="text-base">Interest Rate</span>
          <span class="font-semibold text-gray-900"
            >{formatPercent(interestRate)}</span
          >
        </label>
        <input
          id="interest-rate"
          type="range"
          min="2"
          max="12"
          step="0.1"
          bind:value={interestRate}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="loan-term"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-base">Loan Term</span>
          <span class="font-semibold text-gray-900">{loanTerm} years</span>
        </label>
        <input
          id="loan-term"
          type="range"
          min="5"
          max="30"
          step="5"
          bind:value={loanTerm}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <h3 class="text-xl font-semibold mt-8 mb-4 text-gray-800">
        Additional Costs
      </h3>

      <div class="mb-6">
        <div class="flex justify-between items-center mb-3">
          <label for="property-tax" class="font-medium text-gray-700 text-base">
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
            <span class="text-sm text-gray-600">Annual Amount</span>
            <span class="font-semibold text-gray-900"
              >{formatCurrency(propertyTax)}</span
            >
          </label>
          <input
            id="property-tax-fixed"
            type="range"
            min="0"
            max="20000"
            step="100"
            bind:value={propertyTax}
            class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
          />
        {:else}
          <label
            for="property-tax-percent"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-sm text-gray-600">Tax Rate</span>
            <span class="font-semibold text-gray-900"
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
            class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
          />
        {/if}
      </div>

      <div class="mb-6">
        <label
          for="home-insurance"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-base">Annual Home Insurance</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(homeInsurance)}</span
          >
        </label>
        <input
          id="home-insurance"
          type="range"
          min="0"
          max="10000"
          step="100"
          bind:value={homeInsurance}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="pmi"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-base">Monthly PMI</span>
          <span class="font-semibold text-gray-900">{formatCurrency(pmi)}</span>
        </label>
        <input
          id="pmi"
          type="range"
          min="0"
          max="2000"
          step="1"
          bind:value={pmi}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <div class="mb-6">
        <label
          for="hoa"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-base">Monthly HOA</span>
          <span class="font-semibold text-gray-900">{formatCurrency(hoa)}</span>
        </label>
        <input
          id="hoa"
          type="range"
          min="0"
          max="2000"
          step="1"
          bind:value={hoa}
          class="w-full h-1.5 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>
    </div>

    <div class="sticky top-8 h-fit">
      <h2 class="text-2xl font-semibold mb-6 text-gray-800 text-center">
        Monthly Payment
      </h2>
      <div class="text-5xl font-bold text-green-600 mb-8 text-center">
        {formatCurrencyDetailed(totalMonthlyPayment)}
      </div>

      <div class="border border-gray-200 rounded-lg p-4 mb-8">
        <div class="flex justify-between py-3 border-b border-gray-100">
          <span class="text-gray-600 text-base">Principal & Interest</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(monthlyPrincipalInterest)}</span
          >
        </div>
        <div class="flex justify-between py-3 border-b border-gray-100">
          <span class="text-gray-600 text-base">Property Tax</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(monthlyPropertyTax)}</span
          >
        </div>
        <div class="flex justify-between py-3 border-b border-gray-100">
          <span class="text-gray-600 text-base">Home Insurance</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrencyDetailed(monthlyInsurance)}</span
          >
        </div>
        {#if pmi > 0}
          <div class="flex justify-between py-3">
            <span class="text-gray-600 text-base">PMI</span>
            <span class="font-semibold text-gray-900"
              >{formatCurrencyDetailed(pmi)}</span
            >
          </div>
        {/if}
        {#if hoa > 0}
          <div class="flex justify-between py-3">
            <span class="text-gray-600 text-base">HOA</span>
            <span class="font-semibold text-gray-900"
              >{formatCurrencyDetailed(hoa)}</span
            >
          </div>
        {/if}
      </div>

      <h3 class="text-xl font-semibold mb-4 text-gray-800">Loan Summary</h3>
      <div class="bg-gray-50 rounded-lg p-4">
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-base">Loan Amount</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(principal)}</span
          >
        </div>
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-base">Total Interest</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(totalInterestPaid)}</span
          >
        </div>
        <div class="flex justify-between py-3">
          <span class="text-gray-600 text-base">Total Paid</span>
          <span class="font-semibold text-gray-900"
            >{formatCurrency(totalPaid)}</span
          >
        </div>
      </div>
    </div>
  </div>
</div>
