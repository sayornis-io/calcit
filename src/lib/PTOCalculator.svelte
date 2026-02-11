<script lang="ts">
	let existingPTO = $state(12);
	let desiredPTODays = $state(6);
	let desiredPTOHours = $state(40);
	let inputMode = $state<'days' | 'hours'>('days');
	let accumulationFactor = $state(0.07);
	let shiftLength = $state(12);

	let effectiveDesiredPTOHours = $derived.by(() => {
		return inputMode === 'days' ? desiredPTODays * shiftLength : desiredPTOHours;
	});

	let shiftsNeeded = $derived.by(() => {
		if (accumulationFactor <= 0) return 0;
		const hoursNeeded = Math.max(0, effectiveDesiredPTOHours - existingPTO);
		return Math.ceil(hoursNeeded / (accumulationFactor * shiftLength));
	});

	let totalPTOAfter = $derived.by(() => {
		return existingPTO + shiftsNeeded * accumulationFactor * shiftLength;
	});

	let hoursToAccumulate = $derived.by(() => {
		return shiftsNeeded * accumulationFactor * shiftLength;
	});
</script>

<div class="w-full max-w-7xl mx-auto md:px-4 md:py-8 pb-8">
	<div class="bg-gray-50 p-6 rounded-lg shadow">
		<!-- Input Section -->
		<div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
			<!-- Left Column - Basic Inputs -->
			<div class="bg-white rounded-lg shadow p-6">
				<h2 class="text-xl font-semibold text-gray-800 mb-4">Your PTO Balance</h2>
				
				<div class="space-y-4">
					<!-- Existing PTO -->
					<div>
						<label for="existingPTO" class="block text-sm font-medium text-gray-700 mb-1">
							Existing PTO Hours
						</label>
						<input
							id="existingPTO"
							type="number"
							bind:value={existingPTO}
							step="0.5"
							min="0"
							class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
						/>
					</div>

					<!-- Shift Length -->
					<div>
						<label for="shiftLength" class="block text-sm font-medium text-gray-700 mb-1">
							Shift Length (hours)
						</label>
						<input
							id="shiftLength"
							type="number"
							bind:value={shiftLength}
							step="0.5"
							min="0.5"
							class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
						/>
					</div>

					<!-- PTO Accumulation Factor -->
					<div>
						<label for="accumulationFactor" class="block text-sm font-medium text-gray-700 mb-1">
							Accumulation Factor
						</label>
						<p class="text-xs text-gray-500 mb-2">
							Hours earned per hour worked (e.g., 0.07 = 7% accumulation)
						</p>
						<input
							id="accumulationFactor"
							type="number"
							bind:value={accumulationFactor}
							step="0.001"
							min="0"
							max="1"
							class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
						/>
					</div>
				</div>
			</div>

			<!-- Right Column - Desired PTO -->
			<div class="bg-white rounded-lg shadow p-6">
				<h2 class="text-xl font-semibold text-gray-800 mb-4">Target PTO Balance</h2>
				
				<fieldset class="space-y-4">
					<!-- Toggle Between Days/Hours -->
					<div class="flex gap-4 p-3 bg-gray-50 rounded-md">
						<label class="flex items-center cursor-pointer">
							<input
								type="radio"
								name="ptoMode"
								value="days"
								bind:group={inputMode}
								class="w-4 h-4 text-blue-600"
							/>
							<span class="ml-2 text-sm font-medium text-gray-700">Days</span>
						</label>
						<label class="flex items-center cursor-pointer">
							<input
								type="radio"
								name="ptoMode"
								value="hours"
								bind:group={inputMode}
								class="w-4 h-4 text-blue-600"
							/>
							<span class="ml-2 text-sm font-medium text-gray-700">Hours</span>
						</label>
					</div>

					<!-- Days Input -->
					{#if inputMode === 'days'}
						<div>
							<label for="desiredPTODays" class="block text-sm font-medium text-gray-700 mb-1">
								Desired PTO Days
							</label>
							<input
								id="desiredPTODays"
								type="number"
								bind:value={desiredPTODays}
								step="0.5"
								min="0"
								class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
							/>
							<p class="text-xs text-gray-500 mt-2">
								{desiredPTODays} days = {(desiredPTODays * shiftLength).toFixed(1)} hours (at {shiftLength} hr/day)
							</p>
						</div>
					{:else}
						<div>
							<label for="desiredPTOHours" class="block text-sm font-medium text-gray-700 mb-1">
								Desired PTO Hours
							</label>
							<input
								id="desiredPTOHours"
								type="number"
								bind:value={desiredPTOHours}
								step="0.5"
								min="0"
								class="w-full px-3 py-2 border border-gray-300 rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
							/>
							<p class="text-xs text-gray-500 mt-2">
								{desiredPTOHours.toFixed(1)} hours = {(desiredPTOHours / shiftLength).toFixed(2)} days (at {shiftLength} hr/day)
							</p>
						</div>
					{/if}
				</fieldset>
			</div>
		</div>

		<!-- Results Section -->
		<div class="bg-white rounded-lg shadow overflow-hidden">
			<div class="px-6 py-4 bg-gray-800 text-white">
				<h2 class="text-xl font-semibold">Calculation Results</h2>
			</div>

			<!-- Key Metrics -->
			<div class="grid grid-cols-1 md:grid-cols-3 gap-4 px-6 py-6">
				<!-- Shifts Needed -->
				<div class="bg-blue-50 rounded-lg p-6 border border-blue-200">
					<p class="text-xs font-semibold text-gray-600 uppercase tracking-wider mb-2">Shifts Needed</p>
					<p class="text-5xl font-bold text-blue-600 mb-1">{shiftsNeeded}</p>
					<p class="text-xs text-gray-500">
						to reach {inputMode === 'days' ? `${desiredPTODays} days` : `${desiredPTOHours.toFixed(1)} hours`}
					</p>
				</div>

				<!-- Hours to Accumulate -->
				<div class="bg-indigo-50 rounded-lg p-6 border border-indigo-200">
					<p class="text-xs font-semibold text-gray-600 uppercase tracking-wider mb-2">Hours to Accumulate</p>
					<p class="text-5xl font-bold text-indigo-600 mb-1">{hoursToAccumulate.toFixed(2)}</p>
					<p class="text-xs text-gray-500">from {shiftsNeeded} shifts</p>
				</div>

				<!-- Final PTO Balance -->
				<div class="bg-green-50 rounded-lg p-6 border border-green-200">
					<p class="text-xs font-semibold text-gray-600 uppercase tracking-wider mb-2">Final PTO Balance</p>
					<p class="text-5xl font-bold text-green-600 mb-1">{totalPTOAfter.toFixed(2)}</p>
					<p class="text-xs text-gray-500">after {shiftsNeeded} shifts</p>
				</div>
			</div>

			<!-- Breakdown Table -->
			<div class="px-6 py-6 border-t border-gray-200">
				<h3 class="text-lg font-semibold text-gray-800 mb-4">Calculation Breakdown</h3>
				<div class="overflow-x-auto">
					<table class="w-full text-sm">
						<thead class="bg-gray-100">
							<tr>
								<th class="px-4 py-3 text-left font-semibold text-gray-700">Item</th>
								<th class="px-4 py-3 text-right font-semibold text-gray-700">Value</th>
							</tr>
						</thead>
						<tbody class="divide-y divide-gray-200">
							<tr class="hover:bg-gray-50">
								<td class="px-4 py-3 text-gray-700">Starting PTO Balance</td>
								<td class="px-4 py-3 text-right text-gray-900 font-semibold">
									{existingPTO.toFixed(2)} hours
								</td>
							</tr>
							<tr class="hover:bg-gray-50">
								<td class="px-4 py-3 text-gray-700">Desired PTO Balance</td>
								<td class="px-4 py-3 text-right text-gray-900 font-semibold">
									{#if inputMode === 'days'}
										{desiredPTODays} days ({(desiredPTODays * shiftLength).toFixed(1)} hours)
									{:else}
										{desiredPTOHours.toFixed(1)} hours ({(desiredPTOHours / shiftLength).toFixed(2)} days)
									{/if}
								</td>
							</tr>
							<tr class="hover:bg-gray-50">
								<td class="px-4 py-3 text-gray-700">Shift Length</td>
								<td class="px-4 py-3 text-right text-gray-900 font-semibold">
									{shiftLength.toFixed(1)} hours
								</td>
							</tr>
							<tr class="hover:bg-gray-50">
								<td class="px-4 py-3 text-gray-700">Accumulation Rate</td>
								<td class="px-4 py-3 text-right text-gray-900 font-semibold">
									{(accumulationFactor * 100).toFixed(2)}% per hour
								</td>
							</tr>
							<tr class="hover:bg-gray-50">
								<td class="px-4 py-3 text-gray-700">PTO per Shift</td>
								<td class="px-4 py-3 text-right text-gray-900 font-semibold">
									{(accumulationFactor * shiftLength).toFixed(3)} hours
								</td>
							</tr>
							<tr class="bg-blue-50 hover:bg-blue-100">
								<td class="px-4 py-3 font-semibold text-gray-800">Shifts Needed</td>
								<td class="px-4 py-3 text-right text-blue-700 font-bold text-lg">{shiftsNeeded}</td>
							</tr>
							<tr class="hover:bg-gray-50">
								<td class="px-4 py-3 text-gray-700">Total Hours to Accumulate</td>
								<td class="px-4 py-3 text-right text-gray-900 font-semibold">
									{hoursToAccumulate.toFixed(2)} hours
								</td>
							</tr>
							<tr class="bg-green-50 hover:bg-green-100">
								<td class="px-4 py-3 font-semibold text-gray-800">Final PTO Balance</td>
								<td class="px-4 py-3 text-right text-green-700 font-bold text-lg">
									{totalPTOAfter.toFixed(2)} hours
								</td>
							</tr>
						</tbody>
					</table>
				</div>
			</div>

			<!-- Info Box -->
			<div class="px-6 py-4 bg-amber-50 border-t border-amber-200">
				<p class="text-sm text-gray-700">
					<span class="font-semibold">💡 Tip:</span> The accumulation factor is the percentage of hours worked that you earn as PTO. For example, 0.07 (7%) means you earn 0.84 hours of PTO for every 12-hour shift worked.
				</p>
			</div>
		</div>
	</div>
</div>
