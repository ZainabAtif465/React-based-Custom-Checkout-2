Custom Checkout React SDK– Plan & Documentation  

Author: Zainab Atif    

Audience: Eng / PM / QA  

Purpose: End‑to‑end plan for building, shipping, and maintaining a custom checkout using BigCommerce’s Checkout SDK (with React), leveraging Open Checkout as a reference. Includes constraints, required integrations, security, QA, and rollback. 

 
Premium Points to start with:  

Make a separate folder for building “React SDK Custom Checkout App”, to prevent mix mess.  
Once you Build your Custom Checkout Page, you can upload that renderfy it within your theme to make that a custom checkout of your storefront.  


React + TS + SDK installation using the following flow of commands: 

Create a folder named “React-SDK-for-CC (or whatever you want)”, then open gitbash in that folder and run: 

npm create vite@latest . -- --template react-ts 

vite → the tool we are using to scaffold a new project. 

@latest → tells npm to use the latest published version of create-vite. 

This whole command means: 

Using npm, run the latest version of Vite’s project generator in the current folder, and use the React + TypeScript template. 

React apps are built with tools like Vite/Webpack. 

custom-checkout.iife.js → This is the React bundle. 

**React Structure**

Step 1. Create React App (Vite/CRA) using
cd react-sdk-cc
npm install

Step 2. Install BigCommerce Checkout SDK
npm install @bigcommerce/checkout-sdk

Step 3. Project Structure
src/
  ├── App.tsx              
  ├── init.tsx             
  ├── checkoutService.ts   
  └── components/
       └── steps/
            ├── ShippingStep.tsx
            ├── PaymentStep.tsx
            └── ReviewStep.tsx

Step 4. Checkout Service Setup

File: src/checkoutService.ts

**import { createCheckoutService } from '@bigcommerce/checkout-sdk';

const checkoutService = createCheckoutService();
export default checkoutService; 
**

Step 5. init.tsx 
Basically we should use init only to mount what we've done in our app.tsx or main.tsx 
"App.tsx is an entry point for live/theme server and main.tsx is for locally run.(on vite or other servers)"

Code for init.tsx in my case 
import React from "react";
import { createRoot, Root } from "react-dom/client";
import App from "./App";

let root: Root | null = null;

function mountCheckout(containerId: string = "custom-checkout-root") {
  let el = document.getElementById(containerId);
  if (!el) {
    el = document.createElement("div");
    el.id = containerId;
    (document.querySelector("#main-content") || document.body).prepend(el);
  }
  if (!root) {
    root = createRoot(el);
  }
  root.render(<App />);
}

function unmountCheckout() {
  if (root) {
    root.unmount();
    root = null;
  }
}

(window as any).CustomCheckout = {
  mount: mountCheckout,
  unmount: unmountCheckout,
};

// auto-mount
if (document.readyState === "loading") {
  document.addEventListener("DOMContentLoaded", () => mountCheckout());
} else {
  mountCheckout();
}


Step 5. App.tsx (Main UI + Logic)
Key Features:
Stepper (Shipping → Payment → Review)
Order Summary
safeItemsCount() helper
ensureCartId() to resolve cart
checkoutService.loadCheckout(cartId)



(a) Cart ID Resolution (most important point)

async function ensureCartId(): Promise<string | undefined> {
  let cartId = new URLSearchParams(window.location.search).get("cartId");

  if (!cartId) {
    const res = await fetch("/api/storefront/carts", { credentials: "same-origin" });
    const carts = await res.json();
    if (Array.isArray(carts) && carts.length > 0) {
      cartId = carts[0].id;
      // update URL without reload
      const url = new URL(window.location.href);
      url.searchParams.set("cartId", cartId);
      window.history.replaceState({}, "", url);
    }
  }
  return cartId ?? undefined;
}


(b) Safe Items Count
const safeItemsCount = (cart: any) => {
  const lineItems = cart?.lineItems || {};
  const physical = lineItems.physicalItems || [];
  const digital = lineItems.digitalItems || [];
  const custom = lineItems.customItems || [];
  const gift = lineItems.giftCertificates || [];
  return physical.length + digital.length + custom.length + gift.length;
};


(c) Loading Checkout

 useEffect(() => {
    setSnapshot(checkoutService.getState());
    const unsubscribe = checkoutService.subscribe(() => {
      setSnapshot(checkoutService.getState());
    });

  (async () => {
    try {
      const cid = "5a90185c-3b6e-46ee-a263-f890814f52a3"; // yahan apna cartId daal diya
      await checkoutService.loadCheckout(cid);
      console.log("[CustomCheckout] Checkout loaded with", cid);
    } catch (e: any) {
      console.error("[CustomCheckout] load error", e);
      setToast({ type: "err", msg: e?.message || "Failed to load checkout" });
    } finally {
      setLoading(false);
    }
  })();


    return () => unsubscribe();
  }, []);



Multiple methods for implementing a custom checkout experience for a BigCommerce store.

It includes the steps for using Netlify, Vercel, Checkout.js, and Ngrok methods, as well as an alternative approach using GitHub with port 8080. Each method has its own use cases.

Netlify Method

Netlify is a powerful platform for deploying front-end applications, especially for static sites. 
How to deploy a BigCommerce custom checkout flow using Netlify?

1. Set up your Project:
- Clone your React-based custom checkout repository. 
- Install dependencies:
  npm install

2. Configure Netlify:
- Create a new site on Netlify and link it to your GitHub repository.
- Set the build command to `npm run build`.
- Set the publish directory to `dist/`, and upload it on netlify for a public url.

3. Deploy:
- Push the changes to GitHub or deploy on Netlify. The platform will handle the build and deployment.
- You can use the provided domain or configure a custom domain.

4. Custom Checkout Configuration:
- Set your BigCommerce `storefront` domain and API keys as environment variables in Netlify's settings to ensure secure access to BigCommerce APIs.

Vercel Method
Vercel is another popular platform for deploying React applications. It’s known for its fast deployment times and serverless architecture, which fits perfectly with BigCommerce’s custom checkout.The process of deployment through it, is the same as in Netlify.



BigCommerce Checkout.js Method
BigCommerce provides a custom checkout solution using Checkout.js, a JavaScript SDK designed for integration with BigCommerce stores.
1. Integrate Checkout.js:
- Load the BigCommerce checkout script:
  <script src="https://cdn.bigcommerce.com/checkout/checkout.js"></script>

2. Configure the Checkout Flow:
- Customize the checkout process using the Checkout.js methods to handle cart data, shipping, payment, and order review.
- Implement the checkout logic using JavaScript, such as loading the cart:
  checkout.loadCart(cartId);

3. Add Payment and Shipping:
- Use the Checkout.js methods for adding and customizing payment gateways:
  checkout.addPaymentMethod(paymentMethodId);
  checkout.addShippingOption(shippingOptionId);

4. Finalize Order:
- Once all the steps are complete, you can submit the order using:
  checkout.submitOrder();


Ngrok Method (Most Irritating one, I prefer never doing it again)
Ngrok is a service that creates secure tunnels to your localhost. This is ideal for local testing and development, where you can run your custom checkout flow locally and expose it to external services.

1. Install Ngrok:
- Download and install Ngrok from https://ngrok.com/download.

2. Run Your Local Development Server:
- Start your local development server:
  npm run dev

3. Expose Local Server via Ngrok:
- Expose the server to the internet using Ngrok:
  ngrok http 3000

4. Access Checkout:
- Ngrok will provide a public URL that you can use to test the checkout flow externally.

5. Configure Webhooks:
- Use the Ngrok public URL to test webhooks or APIs that require external access.

Alternative GitHub Method with Port 8080(Better to go for persona use)

If you prefer to host your custom checkout on your own server, you can use GitHub and port 8080 for local development or staging.
1. Clone the Repository:
- Clone the repository:
  git clone https://github.com/your-repository.git
  cd your-repository

2. Install Dependencies:
- Install the necessary packages:
  npm install

3. Run the Application on Port 8080:
- Start the development server:
  npm start

4. Access the Custom Checkout:
- Your application will be accessible via `http://localhost:8080`, or if you're deploying to a server, `http://your-server-ip:8080`.

5. Update API and Configuration:
- Set environment variables for the BigCommerce `cartId`, `storefront`, and `checkoutService` URL to point to your BigCommerce store’s API.

Note: I use github's own codespaces as it was easy to generate a port on a live github server’s terminal and set it as public.
