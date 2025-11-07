<script lang="ts">
	let display = $state('0');
	let previousValue = $state<number | null>(null);
	let operation = $state<string | null>(null);
	let shouldResetDisplay = $state(false);

	function handleNumber(num: string) {
		if (shouldResetDisplay) {
			display = num;
			shouldResetDisplay = false;
		} else {
			display = display === '0' ? num : display + num;
		}
	}

	function handleOperator(op: string) {
		if (previousValue !== null && operation !== null && !shouldResetDisplay) {
			calculate();
		}
		previousValue = parseFloat(display);
		operation = op;
		shouldResetDisplay = true;
	}

	function calculate() {
		if (previousValue === null || operation === null) return;

		const current = parseFloat(display);
		let result: number;

		switch (operation) {
			case '+':
				result = previousValue + current;
				break;
			case '-':
				result = previousValue - current;
				break;
			case '×':
				result = previousValue * current;
				break;
			case '÷':
				result = current !== 0 ? previousValue / current : 0;
				break;
			default:
				return;
		}

		display = result.toString();
		previousValue = null;
		operation = null;
		shouldResetDisplay = true;
	}

	function clear() {
		display = '0';
		previousValue = null;
		operation = null;
		shouldResetDisplay = false;
	}

	function handleDecimal() {
		if (shouldResetDisplay) {
			display = '0.';
			shouldResetDisplay = false;
		} else if (!display.includes('.')) {
			display = display + '.';
		}
	}

	function toggleSign() {
		if (display !== '0') {
			display = display.startsWith('-') ? display.slice(1) : '-' + display;
		}
	}

	function handlePercent() {
		display = (parseFloat(display) / 100).toString();
	}
</script>

<div class="flex items-center justify-center min-h-scree p-4">
	<div class="w-full max-w-md">
		<div class="bg-gray-800 rounded-3xl shadow-2xl p-6 border border-gray-700">
			<!-- Display -->
			<div class="bg-gray-900 rounded-2xl p-6 mb-6 border border-gray-700">
				<div class="text-right text-5xl font-light text-white overflow-hidden break-words">
					{display}
				</div>
			</div>

			<!-- Calculator Buttons -->
			<div class="grid grid-cols-4 gap-3">
				<!-- Row 1 -->
				<button
					onclick={() => clear()}
					class="bg-gray-600 hover:bg-gray-500 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-xl"
				>
					AC
				</button>
				<button
					onclick={() => toggleSign()}
					class="bg-gray-600 hover:bg-gray-500 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-xl"
				>
					+/-
				</button>
				<button
					onclick={() => handlePercent()}
					class="bg-gray-600 hover:bg-gray-500 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-xl"
				>
					%
				</button>
				<button
					onclick={() => handleOperator('÷')}
					class="bg-orange-500 hover:bg-orange-400 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					÷
				</button>

				<!-- Row 2 -->
				<button
					onclick={() => handleNumber('7')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					7
				</button>
				<button
					onclick={() => handleNumber('8')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					8
				</button>
				<button
					onclick={() => handleNumber('9')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					9
				</button>
				<button
					onclick={() => handleOperator('×')}
					class="bg-orange-500 hover:bg-orange-400 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					×
				</button>

				<!-- Row 3 -->
				<button
					onclick={() => handleNumber('4')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					4
				</button>
				<button
					onclick={() => handleNumber('5')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					5
				</button>
				<button
					onclick={() => handleNumber('6')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					6
				</button>
				<button
					onclick={() => handleOperator('-')}
					class="bg-orange-500 hover:bg-orange-400 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					-
				</button>

				<!-- Row 4 -->
				<button
					onclick={() => handleNumber('1')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					1
				</button>
				<button
					onclick={() => handleNumber('2')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					2
				</button>
				<button
					onclick={() => handleNumber('3')}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					3
				</button>
				<button
					onclick={() => handleOperator('+')}
					class="bg-orange-500 hover:bg-orange-400 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					+
				</button>

				<!-- Row 5 -->
				<button
					onclick={() => handleNumber('0')}
					class="col-span-2 bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					0
				</button>
				<button
					onclick={() => handleDecimal()}
					class="bg-gray-700 hover:bg-gray-600 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					.
				</button>
				<button
					onclick={() => calculate()}
					class="bg-orange-500 hover:bg-orange-400 text-white font-semibold py-6 px-4 rounded-xl transition-colors duration-150 text-2xl"
				>
					=
				</button>
			</div>
		</div>
	</div>
</div>
