# Build A Desktop Application Using NW.js and React

## Summary
This is a basic guide for building a desktop application using <a href="https://nwjs.io/">NW.js</a> and <a href="https://reactjs.org/">React</a>.
It should work in both Windows and Linux and most of it will possibly also work in macOS. This is not meant to be an end-to-end build solution, but more
as a guide to getting started with using NW.js together with React.

## Requirements
The following is expected to already be installed/configured before starting this guide.
- <a target="_blank" href="https://nodejs.org/">Node.js</a> - The latest Long-Term Support (LTS) release of Node.js should be installed and in your <span class="code">PATH</span>.
- <a target="_blank" href="https://code.visualstudio.com/">Visual Studio Code</a> - A decent JavaScript editor/IDE should be installed. Some steps in this guide may assume Visual Studio Code is being used but other options are available.

## Getting Started
The following steps will result in a development environment, where your React application will be running in NW.js and automatically reload for any changes. Note that all instances of `nw-react` should be replaced with the name of your application.

1. Open a terminal, navigate to a directory where you have write permission, then run the following commands:

    ```sh
    npx create-react-app nw-react
    cd nw-react
    npm i concurrently wait-on react-devtools
    npm i --save-exact nw@0.65.1-sdk
    ```

    Note: The latest available version of NW.js should be installed above.

2. Open the file `nw-react/package.json` and make the following changes:
    - Rename `dependencies` to `devDependencies`.
    - Add the following:
        ```json
        "main": "main.js",
        "homepage": ".",
        "chromium-args": "--enable-logging=stderr",
        "node-remote": [
          "http://localhost:3042",
          "file://*"
        ],
        "eslintConfig": {
            "globals": {
              "nw": true
            }
        },
        "scripts": {
            "dev": "concurrently \"npm start\" \"wait-on http://localhost:3042 && set NWJS_START_URL=http://localhost:3042 && nw --enable-logging=stderr .\"",
            "dev:tools": "concurrently \"react-devtools\" \"set REACT_APP_DEVTOOLS=enabled && npm start\" \"wait-on http://localhost:3042 && set NWJS_START_URL=http://localhost:3042 && nw --enable-logging=stderr .\"",
            "dev:linux": "concurrently \"npm start\" \"wait-on http://localhost:3042 && export NWJS_START_URL=http://localhost:3042; nw --enable-logging=stderr --remote-debugging-port=3043 .\"",
            "dev:linuxtools": "concurrently \"react-devtools\" \"export REACT_APP_DEVTOOLS=enabled; npm start\" \"wait-on http://localhost:3042 && export NWJS_START_URL=http://localhost:3042; nw --enable-logging=stderr .\"",
        }
        ```

3. Add the following to `nw-react\.env` (new file):
    ```
    PORT=3042
    BROWSER=none
    ```

4. Add the following to `nw-react\main.js` (new file):
    ```js
    const url = require('node:url');

    const baseUri = url.pathToFileURL(__dirname).toString();

    const interfaceUri = process.env.NWJS_START_URL
      ? process.env.NWJS_START_URL.trim()
      : `${baseUri}/build/`;

    const startUri = `${interfaceUri}/index.html`;

    nw.Window.open(startUri);
    ```

5. Add the following at the top of the `<head>` block in `nw-react\public\index.html`:
    ```html
    <script>if ('%REACT_APP_DEVTOOLS%'.trim() === 'enabled') document.write('<script src="http:\/\/localhost:8097"><\/script>')</script>
    ```

6. Add the following after the `import` statements at the top of `nw-react\src\index.js`:
    ```js
    // Bring nw to React namespace
    const nw = global.nw;
    ```

## Development Notes
- At this point, you can run `npm run dev` (Windows) or `npm run dev:linux` (Linux). The React development "live" server will be started and NW.js will be launched, connecting to that "live" server. Any updates to your React application will automatically be reflected in the NW.js window.

- To access Chrome developer tools, right-click on the window that opens. Selecting "Inspect" will show DevTools for the current window. Selecting "Inspect background page" shows DevTools for the Node.js process running `main.js`.

- Running `npm run dev:tools` (Windows) or `npm run dev:linuxtools` (Linux) will behave the same as above, but will also start a standalone version of React DevTools which the React application will connect to.

- Any NPM packages used with the React portion of your application should be installed as `devDependencies` (`npm install --save-dev <package>`). Any NPM packages use by `main.js` (or any other Node.js-context scripts) that need to be included in the "production" application, should be installed as `dependencies` (`npm install <package>`). This will be further-clarified in the "Production Build" section below.

- **NOTE:** There is currently an issue (https://github.com/nwjs/nw.js/issues/7852) where "zombie" NW.js processes stick around and chew up system memory, when the application is closed by pressing CTRL-C in the terminal where `npm run dev` was executed. A decent workaround for this behavior would be much appreciated!

## Production Build
The following steps can be followed to manually create a "production" build of your application. This will use the "built" version of your React application and the "normal" (non-SDK) build of NW.js (disabling DevTools).

1. Run the command `npm run build`.
2. Download the "Normal" (non-SDK) build of NW.js, which matches the version being used for development (0.65.1 in the example above). Extract this into a new directory.
3. Rename the file NW.js executable to match your application name (Windows: `nw.exe` to `nw-react.exe` | Linux: `nw` to `nw-react`).
4. Create a directory named `package.nw` inside the directory created in step #2 above.
5. Copy the following files to this new `package.nw` directory: `main.js`, `package.json`, `build/` (the entire directory).
6. If the Node.js context of your application uses any NPM packages, these need to be installed in the `package.nw` directory (e.g. `npm install --production --global --prefix "<path_to_package.nw_directory>"`).

The main directory from step #2 will now include your fully-functional application. It can be copied anywhere and used without needing anything else pre-installed. Ideally, this directory would now be turned into an "installer" for easy distribution. This can be done with tools like <a href="https://jrsoftware.org/isinfo.php">InnoSetup</a> for Windows or building a DEB/RPM for Linux.
