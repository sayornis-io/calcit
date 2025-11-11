# CalcIt.io

A collection of useful calculators built with Svelte 5, Tailwind, TypeScript, and Vite.

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) (v22 or later)
- [pnpm](https://pnpm.io/)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/sayornis-io/calcit.git
cd calcit
```

2. Install dependencies:
```bash
pnpm install
```

3. Start the development server:
```bash
pnpm dev
```

The app will be available at `http://localhost:5173`


## Project Structure

```
calcit/
├── src/
│   ├── App.svelte                 # Main UI component to get to calculators
│   ├── main.ts                    # Entry point
│   ├── app.css                    # Global styles (Tailwind)
│   ├── lib/
│   │   ├── AmortizationTable.svelte       # Used in MortgageCalculator.svelte
│   │   ├── MortgageCalculator.svelte      # Mortgage payment calculator
│   │   ├── PartnershipCalculator.svelte   # Partnership distribution calculator
│   │   └── SimpleCalculator.svelte        # Basic arithmetic calculator
│   └── utils/
│       └── formatters.ts          # Currency and formatting utilities
├── public/                        # Static assets
├── vite.config.ts                 # Vite configuration
├── tsconfig.json                  # TypeScript configuration
├── tailwind.config.js             # Tailwind CSS configuration
└── package.json                   # Project dependencies
```

## Technology Stack

- **[Svelte 5](https://svelte.dev/)**
- **[Tailwind CSS](https://tailwindcss.com/)**
- **[TypeScript](https://www.typescriptlang.org/)**
- **[Vite](https://vitejs.dev/)**
- **[pnpm](https://pnpm.io/)**

## IDE Setup

Recommended setup: [VS Code](https://code.visualstudio.com/) + [Svelte](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode)

## License

See LICENSE file for details.
