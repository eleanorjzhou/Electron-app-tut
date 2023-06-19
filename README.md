# How to build an Electron App

[Documentation](https://www.electronjs.org/docs/latest/)

## Getting Started
1. In the project folder, run `npm init` on command line.
2. In the `package.json` file, the `main` field should be `main.js`.
3. Run `npm install electron --save-dev`
4. Don't forget to add the `.gitignore` file to avoid commiting the `node_modules` folder.
5. In `package.json` file, under `scripts` add `"start": "electron .",`
6. Add your `index.html` file
7. Add your `main.js` file that loads the `index.html` file when the app starts.


To start the app, run `npm run start`.


## Preload Scripts

1. Add a new `preload.js` script that exposes selected properties of Electron's `process.versions` object to the renderer process in a `versions` global variable:
2. To attach this script to your renderer process, pass its path to the `webPreferences.preload` option in the BrowserWindow constructor:
3. Create a `renderer.js` script that uses the `document.getElementById` DOM API to replace the displayed text for the HTML element with `info` as its `id` property:
4. modify your `index.html` by adding a new element with `info` as its `id` property, and attach your `renderer.js` script.

`main.js` code:
<pre><code>
const { app, BrowserWindow } = require('electron')
const path = require('path')

const createWindow = () => {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  win.loadFile('index.html')
}

app.whenReady().then(() => {
  createWindow()

  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow()
    }
  })
})

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit()
  }
})
</code></pre>

`preload.js` code:
<pre><code>
const { contextBridge } = require('electron')

contextBridge.exposeInMainWorld('versions', {
  node: () => process.versions.node,
  chrome: () => process.versions.chrome,
  electron: () => process.versions.electron
})
</code></pre>

`index.js` code:
<pre><code>
<body>
    <p id="info"></p>
</body>
<script src="./renderer.js"></script>
</code></pre>

`renderer.js` code:
<pre><code>
const information = document.getElementById('info')
information.innerText = `This app is using Chrome (v${window.versions.chrome()}), Node.js (v${window.versions.node()}), and Electron (v${window.versions.electron()})`
</code></pre>


## Packaging Your App

### Import the project
1. Add conversion script by running:
- `npm install --save-dev @electron-forge/cli`
- `npx electron-forge import`
2. Check `package.json` file to see if in `devDependencies` there are `start`, `package`, `make` added under `scripts`.
3. 

### Creating a distribute
To create a distributable, run `npm run make`.

### Signing Your Code
Create a file named `forge.config.js` with the following code:

For MacOS:
<pre><code>
module.exports = {
  packagerConfig: {
    osxSign: {},
    // ...
    osxNotarize: {
      tool: 'notarytool',
      appleId: process.env.APPLE_ID,
      appleIdPassword: process.env.APPLE_PASSWORD,
      teamId: process.env.APPLE_TEAM_ID
    }
    // ...
  }
}
</code></pre>

For Windows:
<pre><code>
module.exports = {
  // ...
  makers: [
    {
      name: '@electron-forge/maker-squirrel',
      config: {
        certificateFile: './cert.pfx',
        certificatePassword: process.env.CERTIFICATE_PASSWORD
      }
    }
  ]
  // ...
}
</code></pre>
