# How to Deploy Your Dashboard Online

To access your dashboard from "outside" (the internet), you have two main options depending on which Data Mode you use.

## Option 1: Cloud Hosting (Recommended for MQTT Mode)
Since you are using a **Public MQTT Broker** (`broker.emqx.io`), you can host your `index.html` file on any free static hosting service. This works because your dashboard doesn't need to talk directly to your private factory network; it talks to the public MQTT broker.

### Steps:
1.  **Prepare Files**: Ensure you have `index.html`, `styles.css`, and `tabs.css` in one folder.
2.  **Use GitHub Pages / Vercel / Netlify**:
    *   **GitHub Pages**: Upload your code to a GitHub repository. Go to Settings -> Pages -> Select "main" branch. You'll get a URL like `https://yourname.github.io/dashboard`.
    *   **Vercel/Netlify**: Just drag and drop your folder onto their website to deploy.
3.  **Operation**:
    *   Your Node-RED (Local) sends data to -> `broker.emqx.io` (Cloud).
    *   Your Website (Cloud) reads data from -> `broker.emqx.io`.
    *   **Result**: You can view the dashboard from anywhere in the world.

### Important:
*   You MUST use **MQTT Mode** in the dashboard settings.
*   **HTTP Mode** will NOT work with this method (unless your Node-RED is also on a public IP).

---

## Option 2: Hosting on Node-RED (Locally or with Tunnel)
If you want to host the file directly on your Node-RED server (e.g., for local LAN access or if you open ports).

### 方法 A: Serve Static Files via Node-RED Config
1.  Locate your Node-RED `settings.js` file (usually in `C:\Users\User\.node-red\` or `/data/`).
2.  Uncomment `httpStatic: '/home/nodered/public',` and point it to your project folder.
3.  Restart Node-RED.
4.  Access via `http://[YOUR_IP]:1880/index.html`.

### Method B: Serve HTML via "HTTP In" Node (No Config Changes)
1.  Drag an `http in` node (Method: GET, URL: `/dashboard`).
2.  Drag a `template` node. Paste your entire `index.html` content into it.
    *   *Note*: You'll need to embed your CSS inside the HTML `<style>` tags for this transparency to work easily, or create separate endpoints for CSS.
3.  Drag an `http response` node.
4.  Connect them: `[http in] -> [template] -> [http response]`.
5.  Access via `http://[YOUR_IP]:1880/dashboard`.

## Option 3: External Access to Local Server (Advanced)
If you host via Option 2 but want to access from outside:
1.  **Ngrok**: Run `ngrok http 1880`. It gives you a temporary public URL.
2.  **Tailscale/ZeroTier**: Install VPN on both the server and your phone/laptop.
3.  **Port Forwarding**: Forward external port 1880 to your PC's internal IP (Riskier security-wise).
