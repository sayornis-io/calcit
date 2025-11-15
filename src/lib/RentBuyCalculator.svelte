<script lang="ts">
  import {
    formatCurrency,
    formatCurrencyDetailed,
    formatPercent,
  } from "../utils/formatters";

  // ===== BUYING INPUTS =====
  let homePrice = $state(640000);
  let downPayment = $state(128000); // 20%
  let interestRate = $state(5.6);
  let loanTerm = $state(30);
  let propertyTaxRate = $state(1.17);
  let homeInsuranceAnnual = $state(3000);
  let hoaMonthly = $state(277);
  let utilitiesBuying = $state(300);

  // Buying - Additional costs
  let closingCostPercent = $state(3);
  let realtorFeePercent = $state(6);
  let homeAppreciationRate = $state(3);

  // Buying - Optional costs
  let includeMaintenanceCosts = $state(true);
  let maintenancePercent = $state(1.5);

  let includeTaxBenefits = $state(true);
  let marginalTaxRate = $state(24);

  // ===== RENTING INPUTS =====
  let monthlyRent = $state(3200);
  let rentIncreaseRate = $state(3.5);
  let rentersInsuranceAnnual = $state(300);
  let utilitiesRenting = $state(200);
  let investDownPayment = $state(true);

  // ===== INVESTMENT & TIMELINE =====
  let investmentReturnRate = $state(7);
  let yearsToCompare = $state(10);

  // Ensure down payment never exceeds home price
  $effect(() => {
    if (downPayment > homePrice) {
      downPayment = homePrice;
    }
  });

  // ===== MORTGAGE CALCULATIONS =====
  let principal = $derived(homePrice - downPayment);
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

  let annualPropertyTax = $derived((homePrice * propertyTaxRate) / 100);
  let monthlyPropertyTax = $derived(annualPropertyTax / 12);
  let monthlyHomeInsurance = $derived(homeInsuranceAnnual / 12);

  // PMI Calculation
  let currentLTV = $derived(principal / homePrice);
  let isPMIApplicable = $derived(currentLTV > 0.8);
  let pmiMonthly = $derived(
    isPMIApplicable ? (principal * 0.005) / 12 : 0,
  ); // 0.5% annually

  // Closing costs (one-time)
  let closingCosts = $derived((homePrice * closingCostPercent) / 100);

  // Monthly maintenance
  let monthlyMaintenance = $derived(
    includeMaintenanceCosts ? (homePrice * maintenancePercent) / 100 / 12 : 0,
  );

  // Total monthly cost of buying
  let totalMonthlyBuying = $derived(
    monthlyPrincipalInterest +
      monthlyPropertyTax +
      monthlyHomeInsurance +
      hoaMonthly +
      pmiMonthly +
      utilitiesBuying +
      monthlyMaintenance,
  );

  // Total monthly cost of renting (initial)
  let totalMonthlyRenting = $derived(
    monthlyRent + rentersInsuranceAnnual / 12 + utilitiesRenting,
  );

  // ===== AMORTIZATION SCHEDULE =====
  let amortizationSchedule = $derived.by(() => {
    const schedule = [];
    let balance = principal;

    for (let i = 1; i <= numberOfPayments; i++) {
      const interestPayment = balance * monthlyInterestRate;
      const principalPayment = monthlyPrincipalInterest - interestPayment;
      balance -= principalPayment;

      // Check if PMI stops (at 78% LTV)
      const ltv = balance / homePrice;
      const pmi = ltv > 0.78 ? pmiMonthly : 0;

      schedule.push({
        month: i,
        payment: monthlyPrincipalInterest,
        principal: principalPayment,
        interest: interestPayment,
        balance: Math.max(0, balance),
        pmi: pmi,
      });
    }

    return schedule;
  });

  // ===== YEAR-BY-YEAR COMPARISON =====
  let comparisonData = $derived.by(() => {
    const data = [];
    const monthsToCompare = yearsToCompare * 12;

    // Track cumulative costs and values
    let buyingTotalCost = closingCosts + downPayment; // Start with closing costs + down payment
    let rentingTotalCost = 0;
    let buyerInvestmentAccount = 0; // What the buyer invests (if monthly costs are lower)
    let renterInvestmentAccount = investDownPayment ? downPayment : 0; // Renter invests down payment if enabled
    let currentHomeValue = homePrice;
    let currentRent = monthlyRent;

    // Monthly investment return rate
    const monthlyInvestmentRate = investmentReturnRate / 100 / 12;

    for (let month = 1; month <= monthsToCompare; month++) {
      // Get mortgage data for this month
      const mortgageData =
        month <= amortizationSchedule.length
          ? amortizationSchedule[month - 1]
          : amortizationSchedule[amortizationSchedule.length - 1];

      // Calculate tax benefits for this month (if applicable)
      let monthlyTaxSavings = 0;
      if (includeTaxBenefits && month <= amortizationSchedule.length) {
        const annualInterest = mortgageData.interest * 12;
        const annualPropertyTaxPaid = annualPropertyTax;
        const totalDeduction = annualInterest + annualPropertyTaxPaid;
        const annualTaxSavings = totalDeduction * (marginalTaxRate / 100);
        monthlyTaxSavings = annualTaxSavings / 12;
      }

      // Calculate monthly costs for buying
      const monthlyBuyingCost =
        monthlyPrincipalInterest +
        monthlyPropertyTax +
        monthlyHomeInsurance +
        hoaMonthly +
        mortgageData.pmi +
        utilitiesBuying +
        monthlyMaintenance -
        monthlyTaxSavings;

      // Increase rent annually
      if (month % 12 === 0 && month > 0) {
        currentRent = currentRent * (1 + rentIncreaseRate / 100);
      }

      const monthlyRentingCost =
        currentRent + rentersInsuranceAnnual / 12 + utilitiesRenting;

      // Add to cumulative costs
      buyingTotalCost += monthlyBuyingCost;
      rentingTotalCost += monthlyRentingCost;

      // Calculate monthly savings and invest accordingly
      const monthlyDifference = monthlyBuyingCost - monthlyRentingCost;
      
      // Apply growth to both accounts first
      buyerInvestmentAccount = buyerInvestmentAccount * (1 + monthlyInvestmentRate);
      renterInvestmentAccount = renterInvestmentAccount * (1 + monthlyInvestmentRate);
      
      if (monthlyDifference < 0) {
        // Buying costs LESS than renting: buyer invests the difference
        buyerInvestmentAccount += Math.abs(monthlyDifference);
      } else {
        // Renting costs LESS than buying: renter invests the difference
        renterInvestmentAccount += monthlyDifference;
      }

      // Appreciate home value monthly
      const monthlyAppreciationRate = homeAppreciationRate / 100 / 12;
      currentHomeValue = currentHomeValue * (1 + monthlyAppreciationRate);

      // Calculate equity (home value - remaining balance)
      const homeEquity = currentHomeValue - mortgageData.balance;

      // Calculate net position for buying (equity + any investments - initial costs)
      const sellingCosts = (currentHomeValue * realtorFeePercent) / 100;
      const netEquityAfterSale = homeEquity - sellingCosts;
      const buyingNetPosition = netEquityAfterSale + buyerInvestmentAccount - downPayment - closingCosts;

      // Calculate net position for renting (investment account - nothing, since we already tracked costs in the account)
      const rentingNetPosition = renterInvestmentAccount;

      // Save data point at end of each year
      if (month % 12 === 0 || month === monthsToCompare) {
        const year = Math.ceil(month / 12);
        data.push({
          year: year,
          buyingCost: Math.round(buyingTotalCost),
          rentingCost: Math.round(rentingTotalCost),
          homeEquity: Math.round(homeEquity),
          homeValue: Math.round(currentHomeValue),
          mortgageBalance: Math.round(mortgageData.balance),
          buyerInvestments: Math.round(buyerInvestmentAccount),
          renterInvestments: Math.round(renterInvestmentAccount),
          buyingNetPosition: Math.round(buyingNetPosition),
          rentingNetPosition: Math.round(rentingNetPosition),
          netAdvantage: Math.round(buyingNetPosition - rentingNetPosition),
        });
      }
    }

    return data;
  });

  // Get final comparison
  let finalComparison = $derived(
    comparisonData.length > 0
      ? comparisonData[comparisonData.length - 1]
      : null,
  );

  // Determine winner
  let winner = $derived(
    finalComparison && finalComparison.netAdvantage > 0 ? "buying" : "renting",
  );
  let advantageAmount = $derived(
    finalComparison ? Math.abs(finalComparison.netAdvantage) : 0,
  );
</script>

<div class="w-full max-w-7xl mx-auto lg:px-4 md:py-8 pb-8">
  <!-- Results Summary -->
  {#if finalComparison}
    <div class="mb-8 p-6 bg-gradient-to-r from-blue-50 to-green-50 rounded-xl border border-gray-200">
      <h2 class="text-2xl font-semibold mb-4 text-center">
        After {yearsToCompare} years, <span
          class={winner === "buying"
            ? "text-green-600"
            : "text-blue-600"}
        >
          {winner === "buying" ? "buying" : "renting"}
        </span> is better by:
      </h2>
      <div class="text-5xl font-bold text-center mb-2" class:text-green-600={winner === "buying"} class:text-blue-600={winner === "renting"}>
        {formatCurrency(advantageAmount)}
      </div>
      <p class="text-center text-gray-600 text-sm">
        Net position difference (equity + investments - costs)
      </p>
    </div>
  {/if}

  <!-- Timeline Slider -->
  <div class="mb-8 p-6 bg-white rounded-xl shadow-lg">
    <label
      for="years-slider"
      class="flex justify-between items-center mb-4 font-medium text-gray-700"
    >
      <span class="text-xl">Years to Compare</span>
      <span class="text-2xl font-bold text-gray-900">{yearsToCompare}</span>
    </label>
    <input
      id="years-slider"
      type="range"
      min="1"
      max="30"
      step="1"
      bind:value={yearsToCompare}
      class="w-full h-3 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
    />
  </div>

  <!-- Input Sections -->
  <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 mb-8">
    <!-- BUYING INPUTS -->
    <div class="bg-white p-6 rounded-xl shadow-lg">
      <h2 class="text-2xl font-semibold mb-6 text-gray-800 border-b pb-3">
        Buying a Home
      </h2>

      <!-- Home Price -->
      <div class="mb-6">
        <label
          for="home-price"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Home Price</span>
          <span class="text-gray-900"
            >$
            <input
              id="home-price-input"
              type="number"
              inputmode="numeric"
              bind:value={homePrice}
              class="w-32 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="home-price"
          type="range"
          min="100000"
          max="2000000"
          step="10000"
          bind:value={homePrice}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- Down Payment -->
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
          max={homePrice}
          step="5000"
          bind:value={downPayment}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
        <div class="text-sm text-gray-500 mt-1">
          {((downPayment / homePrice) * 100).toFixed(1)}% down
        </div>
      </div>

      <!-- Interest Rate -->
      <div class="mb-6">
        <label
          for="interest-rate"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Interest Rate</span>
          <input
            id="interest-rate-input"
            type="number"
            inputmode="decimal"
            step="0.1"
            bind:value={interestRate}
            class="w-18 px-2 border border-gray-300 bg-white rounded text-right mr-1"
          />%
        </label>
        <input
          id="interest-rate"
          type="range"
          min="2"
          max="12"
          step="0.1"
          bind:value={interestRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- Loan Term -->
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
          min="10"
          max="30"
          step="5"
          bind:value={loanTerm}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <h3 class="text-xl font-semibold mt-8 mb-4 text-gray-800">
        Recurring Costs
      </h3>

      <!-- Property Tax -->
      <div class="mb-6">
        <label
          for="property-tax"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Property Tax Rate</span>
          <span class="text-gray-900"
            >{formatPercent(propertyTaxRate)}
            <span class="text-sm text-gray-500"
              >({formatCurrency(annualPropertyTax)}/yr)</span
            >
          </span>
        </label>
        <input
          id="property-tax"
          type="range"
          min="0"
          max="5"
          step="0.01"
          bind:value={propertyTaxRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- Home Insurance -->
      <div class="mb-6">
        <label
          for="home-insurance"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Annual Home Insurance</span>
          <span class="text-gray-900"
            >$
            <input
              id="home-insurance-input"
              type="number"
              inputmode="numeric"
              bind:value={homeInsuranceAnnual}
              class="w-24 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="home-insurance"
          type="range"
          min="500"
          max="10000"
          step="100"
          bind:value={homeInsuranceAnnual}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- HOA -->
      <div class="mb-6">
        <label
          for="hoa"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Monthly HOA</span>
          <span class="text-gray-900"
            >$
            <input
              id="hoa-input"
              type="number"
              inputmode="numeric"
              bind:value={hoaMonthly}
              class="w-20 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="hoa"
          type="range"
          min="0"
          max="1000"
          step="10"
          bind:value={hoaMonthly}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- Utilities (Buying) -->
      <div class="mb-6">
        <label
          for="utilities-buying"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Monthly Utilities</span>
          <span class="text-gray-900"
            >$
            <input
              id="utilities-buying-input"
              type="number"
              inputmode="numeric"
              bind:value={utilitiesBuying}
              class="w-20 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="utilities-buying"
          type="range"
          min="0"
          max="1000"
          step="10"
          bind:value={utilitiesBuying}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <h3 class="text-xl font-semibold mt-8 mb-4 text-gray-800">
        One-Time & Appreciation
      </h3>

      <!-- Closing Costs -->
      <div class="mb-6">
        <label
          for="closing-costs"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Closing Costs</span>
          <span class="text-gray-900"
            >{formatPercent(closingCostPercent)}
            <span class="text-sm text-gray-500"
              >({formatCurrency(closingCosts)})</span
            >
          </span>
        </label>
        <input
          id="closing-costs"
          type="range"
          min="0"
          max="8"
          step="0.1"
          bind:value={closingCostPercent}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- Realtor Fees -->
      <div class="mb-6">
        <label
          for="realtor-fees"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Selling/Realtor Fees</span>
          <span class="text-gray-900"
            >{formatPercent(realtorFeePercent)}
          </span>
        </label>
        <input
          id="realtor-fees"
          type="range"
          min="0"
          max="10"
          step="0.1"
          bind:value={realtorFeePercent}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <!-- Home Appreciation -->
      <div class="mb-6">
        <label
          for="home-appreciation"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span>Annual Home Appreciation</span>
          <span class="text-gray-900"
            >{formatPercent(homeAppreciationRate)}
          </span>
        </label>
        <input
          id="home-appreciation"
          type="range"
          min="-5"
          max="10"
          step="0.1"
          bind:value={homeAppreciationRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
        />
      </div>

      <h3 class="text-xl font-semibold mt-8 mb-4 text-gray-800">
        Optional Costs & Benefits
      </h3>

      <!-- Maintenance Costs Toggle -->
      <div class="mb-6">
        <div class="flex justify-between items-center mb-3">
          <label
            for="maintenance-toggle"
            class="font-medium text-gray-700 text-lg"
          >
            Include Maintenance Costs
          </label>
          <label class="inline-flex items-center cursor-pointer">
            <input
              id="maintenance-toggle"
              type="checkbox"
              bind:checked={includeMaintenanceCosts}
              class="sr-only peer"
            />
            <div
              class="relative w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-green-300 rounded-full peer peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-green-600"
            ></div>
          </label>
        </div>

        {#if includeMaintenanceCosts}
          <label
            for="maintenance-percent"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-sm text-gray-600"
              >Annual Maintenance (% of home value)</span
            >
            <span class="text-gray-900"
              >{formatPercent(maintenancePercent)}
              <span class="text-sm text-gray-500"
                >({formatCurrency(monthlyMaintenance * 12)}/yr)</span
              >
            </span>
          </label>
          <input
            id="maintenance-percent"
            type="range"
            min="0"
            max="5"
            step="0.1"
            bind:value={maintenancePercent}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
          />
        {/if}
      </div>

      <!-- Tax Benefits Toggle -->
      <div class="mb-6">
        <div class="flex justify-between items-center mb-3">
          <label
            for="tax-benefits-toggle"
            class="font-medium text-gray-700 text-lg"
          >
            Include Tax Benefits
          </label>
          <label class="inline-flex items-center cursor-pointer">
            <input
              id="tax-benefits-toggle"
              type="checkbox"
              bind:checked={includeTaxBenefits}
              class="sr-only peer"
            />
            <div
              class="relative w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-green-300 rounded-full peer peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-green-600"
            ></div>
          </label>
        </div>

        {#if includeTaxBenefits}
          <label
            for="tax-rate"
            class="flex justify-between items-center mb-2 font-medium text-gray-700"
          >
            <span class="text-sm text-gray-600">Marginal Tax Rate</span>
            <span class="text-gray-900">{formatPercent(marginalTaxRate)}</span>
          </label>
          <input
            id="tax-rate"
            type="range"
            min="10"
            max="37"
            step="1"
            bind:value={marginalTaxRate}
            class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-green-500"
          />
          <p class="text-xs text-gray-500 mt-2">
            Deducts mortgage interest and property tax from taxable income
          </p>
        {/if}
      </div>

      <!-- Current Monthly Cost -->
      <div class="mt-8 p-4 bg-green-50 border border-green-200 rounded-lg">
        <div class="text-center">
          <p class="text-sm text-gray-600 mb-1">Total Monthly Cost</p>
          <p class="text-3xl font-bold text-green-700">
            {formatCurrency(totalMonthlyBuying)}
          </p>
          {#if isPMIApplicable}
            <p class="text-xs text-gray-500 mt-1">Includes PMI</p>
          {/if}
        </div>
      </div>
    </div>

    <!-- RENTING INPUTS -->
    <div class="bg-white p-6 rounded-xl shadow-lg">
      <h2 class="text-2xl font-semibold mb-6 text-gray-800 border-b pb-3">
        Renting
      </h2>

      <!-- Monthly Rent -->
      <div class="mb-6">
        <label
          for="monthly-rent"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Monthly Rent</span>
          <span class="text-gray-900"
            >$
            <input
              id="monthly-rent-input"
              type="number"
              inputmode="numeric"
              bind:value={monthlyRent}
              class="w-28 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="monthly-rent"
          type="range"
          min="500"
          max="10000"
          step="50"
          bind:value={monthlyRent}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <!-- Rent Increase Rate -->
      <div class="mb-6">
        <label
          for="rent-increase"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Annual Rent Increase</span>
          <span class="text-gray-900">{formatPercent(rentIncreaseRate)}</span>
        </label>
        <input
          id="rent-increase"
          type="range"
          min="0"
          max="10"
          step="0.1"
          bind:value={rentIncreaseRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <!-- Renters Insurance -->
      <div class="mb-6">
        <label
          for="renters-insurance"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Annual Renter's Insurance</span>
          <span class="text-gray-900"
            >$
            <input
              id="renters-insurance-input"
              type="number"
              inputmode="numeric"
              bind:value={rentersInsuranceAnnual}
              class="w-24 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="renters-insurance"
          type="range"
          min="0"
          max="1000"
          step="25"
          bind:value={rentersInsuranceAnnual}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <!-- Utilities (Renting) -->
      <div class="mb-6">
        <label
          for="utilities-renting"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Monthly Utilities</span>
          <span class="text-gray-900"
            >$
            <input
              id="utilities-renting-input"
              type="number"
              inputmode="numeric"
              bind:value={utilitiesRenting}
              class="w-20 px-2 border border-gray-300 bg-white rounded text-right"
            />
          </span>
        </label>
        <input
          id="utilities-renting"
          type="range"
          min="0"
          max="1000"
          step="10"
          bind:value={utilitiesRenting}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
      </div>

      <h3 class="text-xl font-semibold mt-8 mb-4 text-gray-800">Investment</h3>

      <!-- Invest Down Payment Toggle -->
      <div class="mb-6">
        <div class="flex justify-between items-center mb-3">
          <label
            for="invest-down-payment-toggle"
            class="font-medium text-gray-700 text-lg"
          >
            Invest Down Payment Equivalent
          </label>
          <label class="inline-flex items-center cursor-pointer">
            <input
              id="invest-down-payment-toggle"
              type="checkbox"
              bind:checked={investDownPayment}
              class="sr-only peer"
            />
            <div
              class="relative w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 rounded-full peer peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"
            ></div>
          </label>
        </div>
        <p class="text-xs text-gray-500">
          When renting, invest {formatCurrency(downPayment)} (the down payment you didn't spend on a home) and let it grow at the investment return rate
        </p>
      </div>

      <!-- Investment Return Rate -->
      <div class="mb-6">
        <label
          for="investment-return"
          class="flex justify-between items-center mb-2 font-medium text-gray-700"
        >
          <span class="text-lg">Annual Investment Return</span>
          <span class="text-gray-900"
            >{formatPercent(investmentReturnRate)}</span
          >
        </label>
        <input
          id="investment-return"
          type="range"
          min="0"
          max="15"
          step="0.1"
          bind:value={investmentReturnRate}
          class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-blue-500"
        />
        <p class="text-xs text-gray-500 mt-2">
          Monthly cost differences are invested at this rate
        </p>
      </div>

      <!-- Current Monthly Cost -->
      <div class="mt-8 p-4 bg-blue-50 border border-blue-200 rounded-lg">
        <div class="text-center">
          <p class="text-sm text-gray-600 mb-1">Total Monthly Cost (Initial)</p>
          <p class="text-3xl font-bold text-blue-700">
            {formatCurrency(totalMonthlyRenting)}
          </p>
          <p class="text-xs text-gray-500 mt-1">
            Increases {formatPercent(rentIncreaseRate)} annually
          </p>
        </div>
      </div>

      <!-- Info Box -->
      <div class="mt-8 p-4 bg-amber-50 border border-amber-200 rounded-lg">
        <h4 class="font-semibold text-amber-900 mb-2">How it works:</h4>
        <ul class="text-sm text-amber-800 space-y-2">
          {#if investDownPayment}
            <li>
              • Renter starts with down payment ({formatCurrency(downPayment)}) invested
            </li>
          {/if}
          <li>
            • Each month, whoever pays less invests the difference
          </li>
          <li>• Rent increases annually while mortgage P&I stays fixed</li>
          <li>
            • Home equity builds through mortgage paydown + appreciation
          </li>
          <li>
            • Final comparison includes selling costs when calculating net
            position
          </li>
        </ul>
      </div>
    </div>
  </div>

  <!-- Detailed Comparison Table -->
  <div class="bg-white p-6 rounded-xl shadow-lg overflow-x-auto">
    <h2 class="text-2xl font-semibold mb-6 text-gray-800">
      Year-by-Year Breakdown
    </h2>
    <table class="w-full text-sm">
      <thead>
        <tr class="border-b-2 border-gray-300">
          <th class="text-left py-3 px-2">Year</th>
          <th class="text-right py-3 px-2">Buying<br />Total Cost</th>
          <th class="text-right py-3 px-2">Renting<br />Total Cost</th>
          <th class="text-right py-3 px-2">Home<br />Value</th>
          <th class="text-right py-3 px-2">Home<br />Equity</th>
          <th class="text-right py-3 px-2">Mortgage<br />Balance</th>
          <th class="text-right py-3 px-2">Buyer<br />Investments</th>
          <th class="text-right py-3 px-2">Renter<br />Investments</th>
          <th class="text-right py-3 px-2 font-bold">Net Advantage<br />(Buying)</th>
        </tr>
      </thead>
      <tbody>
        {#each comparisonData as row, i}
          <tr class="border-b border-gray-200 hover:bg-gray-50">
            <td class="py-3 px-2 font-medium">{row.year}</td>
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.buyingCost)}</td
            >
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.rentingCost)}</td
            >
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.homeValue)}</td
            >
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.homeEquity)}</td
            >
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.mortgageBalance)}</td
            >
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.buyerInvestments)}</td
            >
            <td class="text-right py-3 px-2"
              >{formatCurrency(row.renterInvestments)}</td
            >
            <td
              class="text-right py-3 px-2 font-bold"
              class:text-green-600={row.netAdvantage > 0}
              class:text-red-600={row.netAdvantage < 0}
            >
              {row.netAdvantage >= 0 ? "+" : ""}{formatCurrency(
                row.netAdvantage,
              )}
            </td>
          </tr>
        {/each}
      </tbody>
    </table>
    <div class="mt-4 text-xs text-gray-600 space-y-2">
      <p>
        <strong>Buyer Investments:</strong> Money invested when buying costs less than renting each month.
      </p>
      <p>
        <strong>Renter Investments:</strong> {investDownPayment ? 'Down payment equivalent + money' : 'Money'} invested when renting costs less than buying each month.
      </p>
      <p>
        <strong>Net Advantage:</strong> (Home Equity - Selling Costs + Buyer Investments) minus (Renter Investments). Positive means buying is better.
      </p>
    </div>
  </div>
</div>

<style>
  /* Custom range slider styles for better appearance */
  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: currentColor;
    cursor: pointer;
  }

  input[type="range"]::-moz-range-thumb {
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background: currentColor;
    cursor: pointer;
    border: none;
  }
</style>
