## Testing a Next.js Component with TypeScript, Vitest, and react‑testing‑library**

---

## Objectives
- Create a Next.js project using TypeScript.
- Configure Vitest as the test runner with jsdom.
- Write tests for a simple TypeScript React component using react‑testing‑library.

---

## Step 1: Set Up a New Next.js Project with TypeScript

1. **Create the Project:**  
   Open your terminal and run:
   ```bash
   npx create-next-app@latest my-next-app --typescript
   ```
   **Note:** When Next.js prompts you with various questions (such as the project name confirmation, ESLint setup, Tailwind CSS, etc.), simply press **Enter** to accept the default answers.

2. **Navigate to the Project Directory:**
   ```bash
   cd my-next-app
   ```

---

## Step 2: Install Testing Dependencies

Install Vitest, react‑testing‑library, jest‑dom (for additional matchers), and jsdom (to simulate a browser environment):
```bash
npm install --save-dev vitest @testing-library/react @testing-library/jest-dom jsdom
```

---

## Step 3: Configure Vitest

Create a file named **vitest.config.ts** in the project root with the following content:
```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
});
```
This configuration tells Vitest to use jsdom as the testing environment, ensuring your tests have access to a simulated browser DOM.

---

## Step 4: Create a Simple TypeScript React Component

Create a new file at **components/Greeting.tsx** with the following content:
```tsx
import React from 'react';

interface GreetingProps {
  name: string;
}

const Greeting: React.FC<GreetingProps> = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

export default Greeting;
```

---

## Step 5: Write a Test for the Component

Create a test file at **components/Greeting.test.tsx**. Note that we explicitly import the testing functions from Vitest for clarity:
```tsx
import React from 'react';
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import '@testing-library/jest-dom';
import Greeting from './Greeting';

describe('Greeting Component', () => {
  it('renders the correct greeting message', () => {
    render(<Greeting name="Vitest" />);
    expect(screen.getByText('Hello, Vitest!')).toBeInTheDocument();
  });
});
```
Even though the `globals: true` option makes these functions available globally, explicitly importing them can improve clarity and maintainability.

---

## Step 6: Update the Test Script in package.json

Modify the **scripts** section in your **package.json** to add a test command:
```json
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "test": "vitest"
}
```

---

## Step 7: Run Your Tests

In your terminal, run:
```bash
npm run test
```
Review the output to ensure that your test passes successfully.

---

## Step 8: Discussion & Extensions

- **Understanding the Setup:**  
  - Accepting default prompts during project creation simplifies the setup process.
  - The project now uses TypeScript and the new JSX transform (with `"jsx": "react-jsx"` in your tsconfig), though you may need to import React explicitly in some test files.

- **Next Steps & Extensions:**  
  - Experiment with more complex components that include state or side effects.
  - Add user interaction tests using [@testing-library/user-event](https://testing-library.com/docs/user-event/intro/).
  - Compare the performance and configuration differences between Vitest and traditional Jest setups in Next.js.

This comprehensive exercise gives you a hands‑on introduction to a modern testing workflow in Next.js using TypeScript, Vitest, and react‑testing‑library. Enjoy coding and testing!