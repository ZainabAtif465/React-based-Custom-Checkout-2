Custom Checkout React SDK– Plan & Documentation  

Author: Zainab Atif    

Audience: Eng / PM / QA  

Purpose: End‑to‑end plan for building, shipping, and maintaining a custom checkout using BigCommerce’s Checkout SDK (with React), leveraging Open Checkout as a reference. Includes constraints, required integrations, security, QA, and rollback. 

 

Premium Points to start with:  

Make a separate folder for building “React SDK Custom Checkout App”, to prevent mix mess.  

Once you Build your Custom Checkout Page, you can upload that folder within your theme to make that a functioning part of your storefront.  

 

Step 1:  

React + TS + SDK installation using the following flow of commands: 

Create a folder named “React-SDK-for-CC (or whatever you want)”, then open gitbash in that folder and run: 

npm create vite@latest . -- --template react-ts 

vite → the tool we are using to scaffold a new project. 

@latest → tells npm to use the latest published version of create-vite. 

This whole command means: 

Using npm, run the latest version of Vite’s project generator in the current folder, and use the React + TypeScript template. 

React apps are built with tools like Vite/Webpack. 

custom-checkout.js → This is the React bundle. 

**React Structure**

First of all 
 
