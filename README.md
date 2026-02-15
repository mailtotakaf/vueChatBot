# vueChatBot

This template should help get you started developing with Vue 3 in Vite.

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## Customize configuration

See [Vite Configuration Reference](https://vite.dev/config/).

## Project Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```

### Compile and Minify for Production

```sh
npm run build
```


---
## Setup Instructions

### 1. Dify Setup (dify.ai)
Log in to [Dify.ai](https://dify.ai/) and prepare your environment:

- **Create an App:** Select "Create from Scratch" → "Chatbot".
- **Generate API Key:** Go to **API Access** in the left menu, click "Create API Key," and save it somewhere safe.
- **Check API Base URL:** On the same page, find your API Base URL (e.g., `https://api.dify.ai/v1`).

### 2. Prepare Vue for API Requests
To make API calls from the browser, install **axios** by running the following command in your terminal:

```bash
npm install axios
```

### 3. Configure Environment Variables
Create a file named .env in your project's root folder and add your API key:
```
VITE_DIFY_API_KEY=<your_api_key_here>
```

---
## スマホ向け起動
```
npm run preview -- --host
```
