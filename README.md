# CalcIt.io

A collection of useful calculators built with Svelte 5, TypeScript, Tailwind, and Vite.

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

## Available Scripts

- `pnpm dev` - Start the development server with hot module replacement
- `pnpm build` - Build the project for production
- `pnpm preview` - Preview the production build locally

## Project Structure

```
calcit/
├── src/
│   ├── App.svelte                 # Main UI component to get to calculators
│   ├── main.ts                    # Entry point
│   ├── app.css                    # Global styles (Tailwind)
│   ├── lib/
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
- **[TypeScript](https://www.typescriptlang.org/)**
- **[Vite](https://vitejs.dev/)**
- **[Tailwind CSS](https://tailwindcss.com/)**
- **[pnpm](https://pnpm.io/)**

## Features

- **Mortgage Calculator** - Calculate monthly payments with taxes, insurance, PMI, and HOA
- **Partnership Calculator** - Distribute project costs and loans among partners
- **Simple Calculator** - Basic arithmetic operations
- **Responsive Design** - Works on desktop and mobile devices
- **Dark Mode** - Built with Tailwind's dark color palette

## IDE Setup

Recommended setup: [VS Code](https://code.visualstudio.com/) + [Svelte](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode)

## License

See LICENSE file for details.
