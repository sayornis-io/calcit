<script lang="ts">
  import MortgageCalculator from './lib/MortgageCalculator.svelte'
  import PartnershipCalculator from './lib/PropertyFundingAnalyzer.svelte'
  import SimpleCalculator from './lib/SimpleCalculator.svelte'

  const calculators = [
    { name: 'Mortgage', component: MortgageCalculator },
    { name: 'Partnership', component: PartnershipCalculator },
    { name: 'Simple', component: SimpleCalculator },
  ]

  let active = $state(0)
</script>

<main class="bg-gray-100 min-h-screen">
  <div class="mx-auto px-4 py-8">
    <div class="flex gap-2 mb-8">
      {#each calculators as calc, i}
        <button
          onclick={() => active = i}
          class="px-4 py-2 rounded font-semibold transition-colors {
            active === i
              ? 'bg-blue-600 text-white'
              : 'bg-gray-600 text-gray-200 hover:bg-gray-500'
          }"
        >
          {calc.name}
        </button>
      {/each}
    </div>

    <div class="card">
      {#key active}
        <svelte:component this={calculators[active].component} />
      {/key}
    </div>
  </div>
</main>