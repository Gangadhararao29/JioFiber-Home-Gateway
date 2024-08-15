# Check Available Versions For JF Router Firmwares

*Disclaimer: - This is Only for educational purposes, No one is responsible for any type of damage.*

1. First go to `http://fota.slv.fxd.jiophone.net/` using your PC Browser.
2. Open Developer Tools and Click on the Console option.
3. At the top of the console window (just at the right of Filter box), you will find a spinner named `Custom levels`. Click it and turn off the `Error` checkbox.
4. Copy the script below and paste into the console.
5. In the console, edit the variables `router.manufacturer`, `router.model`, `router.firmwarePrefix`, `currentVersion` and `maxVersion` according to your need.
6. Press Enter in the console which will show the Router Firmware versions along with their URLs.

```js
/*
1. Goto http://fota.slv.fxd.jiophone.net/
2. Replace router options and current and max versions accordingly
3. Run it in browser developer console, to scan for available firmware versions.
*/

function incrementVersion(version) {
    let [major, minor, patch] = version.split('.').map(num => parseInt(num, 10));
    
    patch += 1;
    if (patch > 9) {
        patch = 0;
        minor += 1;
    }
    
    if (minor > 99) {
        minor = 0;
        major += 1;
    }

    return `${major}.${minor}.${patch}`;
}

function formatVersionForUrl(version) {
    let [major, minor, patch] = version.split('.');
    if (parseInt(patch, 10) === 0) {
        return `${major}.${minor}`;
    }
    return `${major}.${minor}.${patch}`;
}

function checkFirmwareExists(version, url) {
    return new Promise((resolve) => {
        const http = new XMLHttpRequest();
        http.open('HEAD', url, true);
        http.onreadystatechange = function() {
            if (this.readyState === this.DONE) {
                if (this.status !== 404) {
                    console.log(`${version} : ${url}`);
                }
                resolve();
            }
        };
        http.onerror = function() {
            console.error(`Error checking URL: ${url}`);
            resolve();
        };
        http.send();
    });
}

async function loadFirmwares() {
    const router = {
        manufacturer: "Sercomm", // Replace this with your Router Manufacturer
        model: "JCOW414",  // Replace this with your Router Model Name
        firmwarePrefix: "SRCMTF1_JCOW414_R", // Replace this with your Router Firmware Version Prefix
    };

    let currentVersion = "2.30.0";
    const maxVersion = "3.0.0"; // Set the maximum version here

    const versionChecks = [];

    while (currentVersion !== maxVersion) {
        const formattedVersion = formatVersionForUrl(currentVersion);
        const url = `http://fota.slv.fxd.jiophone.net/ONT/${router.manufacturer}/${router.model}/${router.firmwarePrefix}${formattedVersion}.img`;
        versionChecks.push(checkFirmwareExists(currentVersion, url));

        currentVersion = incrementVersion(currentVersion);
        
        if (parseInt(currentVersion.split('.')[0], 10) > parseInt(maxVersion.split('.')[0], 10)) break;
    }

    await Promise.all(versionChecks);
}

loadFirmwares();
```
