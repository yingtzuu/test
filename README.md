# Orbit — Vue 3 SaaS Product Dashboard

A responsive product-management dashboard built with Vue 3 + Vite.

## Requirements
- Node.js 20.19+ or 22.12+
- npm

## Run locally
```bash
npm install
npm run dev
```
Open the local URL printed in the terminal (usually http://localhost:5173).

## Production build
```bash
npm run build
npm run preview
```
The production-ready static files are generated in `dist/`.

## Deploy to Vercel
1. Push this project to a GitHub repository.
2. Import that repository at https://vercel.com/new.
3. Vercel should detect Vite automatically. Confirm:
   - Build command: `npm run build`
   - Output directory: `dist`
4. Click Deploy.

## Demo data and limitations
- Product data is stored in this browser's `localStorage`.
- This is a front-end demo. It does not include authentication, a server, or a shared database.
- Different browsers/devices do not share the saved data. Do not enter confidential or real customer information.
