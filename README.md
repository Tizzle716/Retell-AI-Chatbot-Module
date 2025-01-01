# WebContainer API Starter

WebContainer API is a browser-based runtime for executing Node.js applications and operating system commands. It enables you to build applications that previously required a server running.

WebContainer API is perfect for building interactive coding experiences. Among its most common use cases are production-grade IDEs, programming tutorials, or employee onboarding platforms.

## How To

For an up-to-date documentation, please refer to [our documentation](https://webcontainers.io).

## Cross-Origin Isolation

WebContainer _requires_ [SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) to function. In turn, this requires your website to be [cross-origin isolated](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer#security_requirements). Among other things, the root document must be served with:

```bash
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Opener-Policy: same-origin
```

You can check [our article](https://blog.stackblitz.com/posts/cross-browser-with-coop-coep/) on the subject and our [docs on browser support](https://developer.stackblitz.com/docs/platform/browser-support) for more details.

## Serve over HTTPS

Please note that your deployed page must be served over HTTPS. This is not necessary when developing locally, as `localhost` is exempt from some browser restrictions, but there is no way around it once you deploy to production.

## Demo

Check [the WebContainer API demo app](https://webcontainer.new).

Here's an example `main.ts` file:

```ts
import { WebContainer } from "@webcontainer/api";

const files: FileSystemTree = {
  "index.js": {
    file: {
      contents: "",
    },
  },
};

let webcontainer: WebContainer;

// add a textarea (the editor) and an iframe (a preview window) to the document
document.querySelector("#app").innerHTML = `
  <div class="container">
    <div class="editor">
      <textarea>I am a textarea</textarea>
    </div>
    <div class="preview">
      <iframe></iframe>
    </div>
  </div>
`;

// the editor
const textarea = document.querySelector("textarea");

// the preview window
const iframe = document.querySelector("iframe");

window.addEventListener("load", async () => {
  textarea.value = files["index.js"].file.contents;

  textarea.addEventListener("input", (event) => {
    const content = event.currentTarget.value;
    webcontainer.fs.writeFile("/index.js", content);
  });

  // call only once
  webcontainer = await WebContainer.boot();

  await webcontainer.mount(files);

  const exitCode = await installDependencies();

  if (exitCode !== 0) {
    throw new Error("Installation failed");
  }

  startDevServer();
});

async function installDependencies() {
  // install dependencies
  const installProcess = await webcontainer.spawn("npm", ["install"]);

  installProcess.output.pipeTo(
    new WritableStream({
      write(data) {
        console.log(data);
      },
    })
  );

  // wait for install command to exit
  return installProcess.exit;
}

async function startDevServer() {
  // run `npm run start` to start the express app
  await webcontainer.spawn("npm", ["run", "start"]);

  // wait for `server-ready` event
  webcontainer.on("server-ready", (port, url) => {
    iframe.src = url;
  });
}
```

## Troubleshooting

Cookie blockers, either from third-party addons or built-in into the browser, can prevent WebContainer from running correctly. Check the `on('error')` event and our [docs](https://developer.stackblitz.com/docs/platform/third-party-blocker).

To troubleshoot other problems, check the [Troubleshooting page](https://webcontainers.io/guides/troubleshooting) in our docs.

## Adapting for Cloudflare Workers

To adapt this project for Cloudflare Workers, follow these steps:

1. **Convert the Application to a Serverless Function:**
   - Cloudflare Workers run serverless functions, so you'll need to convert your application into a function that can be executed by Cloudflare Workers. This typically involves creating a `worker.js` file that exports a `fetch` event handler.

2. **Install Wrangler:**
   - Install the Cloudflare Wrangler CLI by running `npm install -g @cloudflare/wrangler`.

3. **Initialize a Wrangler Project:**
   - Run `wrangler init` to create a new Wrangler project. Follow the prompts to set up your project.

4. **Write the Worker Script:**
   - Create a `worker.js` file with the following content:

     ```js
     export default {
       async fetch(request) {
         // Your serverless function logic here
         return new Response("Hello from Cloudflare Workers!", {
           headers: { "content-type": "text/plain" },
         });
       },
     };
     ```

5. **Configure Wrangler:**
   - Update the `wrangler.toml` file with your Cloudflare account details and other configurations.

6. **Deploy the Worker:**
   - Run `wrangler publish` to deploy your worker to Cloudflare.

# License

Copyright 2023 StackBlitz, Inc.

## How to Use This Template

To use this template, follow these steps:

1. **Fork the Repository:**
   - Click the "Fork" button at the top right of this repository to create a copy in your GitHub account.

2. **Clone Your Fork:**
   - Clone the forked repository to your local machine using the following command:
     ```bash
     git clone https://github.com/your-username/stackblitz-webcontainer-api-starter.git
     ```

3. **Navigate to the Project Directory:**
   - Change into the project directory:
     ```bash
     cd stackblitz-webcontainer-api-starter
     ```

4. **Install Dependencies:**
   - Install the project dependencies:
     ```bash
     npm install
     ```

5. **Run the Project:**
   - Start the development server:
     ```bash
     npm run dev
     ```

6. **Deploy to Cloudflare Workers (Optional):**
   - Follow the instructions in the [Adapting for Cloudflare Workers](#adapting-for-cloudflare-workers) section to deploy your project as a serverless function on Cloudflare Workers.
