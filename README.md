# Welcome to your Lovable project

## Project info

**URL**: https://lovable.dev/projects/79034c31-dd18-408b-bc42-c48d02bd1301

## How can I edit this code?

There are several ways of editing your application.

**Use Lovable**

Simply visit the [Lovable Project](https://lovable.dev/projects/79034c31-dd18-408b-bc42-c48d02bd1301) and start prompting.

Changes made via Lovable will be committed automatically to this repo.

**Use your preferred IDE**

If you want to work locally using your own IDE, you can clone this repo and push changes. Pushed changes will also be reflected in Lovable.

The only requirement is having Node.js & npm installed - [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating)

Follow these steps:

```sh
# Step 1: Clone the repository using the project's Git URL.
git clone <YOUR_GIT_URL>

# Step 2: Navigate to the project directory.
cd <YOUR_PROJECT_NAME>

# Step 3: Install the necessary dependencies.
npm i

# Step 4: Start the development server with auto-reloading and an instant preview.
npm run dev
```

**Edit a file directly in GitHub**

- Navigate to the desired file(s).
- Click the "Edit" button (pencil icon) at the top right of the file view.
- Make your changes and commit the changes.

**Use GitHub Codespaces**

- Navigate to the main page of your repository.
- Click on the "Code" button (green button) near the top right.
- Select the "Codespaces" tab.
- Click on "New codespace" to launch a new Codespace environment.
- Edit files directly within the Codespace and commit and push your changes once you're done.

## What technologies are used for this project?

This project is built with:

- Vite
- TypeScript
- React
- shadcn-ui
- Tailwind CSS

## How can I deploy this project?

Simply open [Lovable](https://lovable.dev/projects/79034c31-dd18-408b-bc42-c48d02bd1301) and click on Share -> Publish.

## Can I connect a custom domain to my Lovable project?

Yes, you can!

To connect a domain, navigate to Project > Settings > Domains and click Connect Domain.

Read more here: [Setting up a custom domain](https://docs.lovable.dev/features/custom-domain#custom-domain)

## 🎵 Presence Beacon

This project includes an advanced spread-spectrum audio presence beacon system at `/presence-beacon.html`.

### What it does
Over-air presence verification using imperceptible spectral modifications at 12 kHz. Encodes PN-31 Gold code via differential A/B modulation, detected via correlation matching with adaptive baseline calibration.

### Quick Start
1. **RX (Listener)**: Open `/presence-beacon.html` → Click "Start Audio" → "Start RX" → Wait for "✅ Calibrated"
2. **TX (Encoder)**: Open `/presence-beacon.html` in another tab/device → "Start Audio" → "Start TX"
3. **Verify**: RX should show "🔓 UNLOCKED" within 10-16 seconds

### Documentation
See [SPREAD-SPECTRUM-PRESENCE-BEACON.md](./SPREAD-SPECTRUM-PRESENCE-BEACON.md) for comprehensive technical documentation.

### Current Status
✅ **Working configuration** - Encoded audio unlocks (corr=0.410, z=4.61), unencoded audio correctly rejected (z<2.5).

### Testing Checklist
See [CHECKLIST.md](./CHECKLIST.md) for validation procedures.
