# Zyte SmartProxy Puppeteer-Core
[![made-with-javascript](https://img.shields.io/badge/Made%20with-JavaScript-1f425f.svg)](https://www.javascript.com)
[![npm](https://img.shields.io/npm/v/zyte-smartproxy-puppeteer-core)](https://www.npmjs.com/package/zyte-smartproxy-puppeteer-core)

Use [puppeteer-core](https://github.com/puppeteer/puppeteer/) with
[Smart Proxy Manager](https://www.zyte.com/smart-proxy-manager/) easily!

A wrapper over puppeteer-core to provide Zyte Smart Proxy Manager specific functionalities.

## QuickStart

1. **Install Zyte SmartProxy puppeteer-core**

```
npm install zyte-smartproxy-puppeteer-core chrome-launcher axios
```

2. **Create a file `sample.js` with following content and replace `<SPM_APIKEY>` with your SPM Apikey**

``` javascript
const puppeteer = require('zyte-smartproxy-puppeteer-core');
const chromeLauncher = require('chrome-launcher');
const axios = require('axios');

(async () => {
    // Initializing a Chrome instance manually
    const chrome = await chromeLauncher.launch(
         {chromeFlags: ['--proxy-server=http://proxy.zyte.com:8011', '--disable-site-isolation-trials']}
    );
    const response = await axios.get(`http://localhost:${chrome.port}/json/version`);
    const { webSocketDebuggerUrl } = response.data;

    // Connect to the Chrome instance
    const browser = await puppeteer.connect({
        spm_apikey: '<SPM_APIKEY>',
        spm_host: 'http://proxy.zyte.com:8011',
        ignoreHTTPSErrors: true,
        browserWSEndpoint: webSocketDebuggerUrl,
    });
    console.log('Before new page');
    const page = await browser.newPage();

    console.log('Opening page ...');
    try {
        await page.goto('https://toscrape.com/', {timeout: 180000});
    } catch(err) {
        console.log(err);
    }

    console.log('Taking a screenshot ...');
    await page.screenshot({path: 'screenshot.png'});
    await browser.close();
})();
```

Make sure that you're able to make `https` requests using Smart Proxy Manager by following this guide [Fetching HTTPS pages with Zyte Smart Proxy Manager](https://docs.zyte.com/smart-proxy-manager/next-steps/fetching-https-pages-with-smart-proxy.html)

3. **Run `sample.js` using Node**

``` bash
node sample.js
```

## API

`ZyteProxyPuppeteer.connect` accepts all the arguments accepted by `Puppeteer.connect` and some
additional arguments defined below:

| Argument | Default Value | Description |
|----------|---------------|-------------|
| `spm_apikey` | `undefined` | Zyte Smart Proxy Manager API key that can be found on your zyte.com account. |
| `spm_host` | `http://proxy.zyte.com:8011` | Zyte Smart Proxy Manager proxy host. |
| `static_bypass` | `true` | When `true` ZyteProxyPuppeteer will skip proxy use for static assets defined by `static_bypass_regex` or pass `false` to use proxy. |
| `static_bypass_regex` | `/.*?\.(?:txt\|json\|css\|less\|gif\|ico\|jpe?g\|svg\|png\|webp\|mkv\|mp4\|mpe?g\|webm\|eot\|ttf\|woff2?)$/` | Regex to use filtering URLs for `static_bypass`. |
| `block_ads` | `true` | When `true` ZyteProxyPuppeteer will block ads defined by `block_list` using `@cliqz/adblocker-puppeteer` package. |
| `block_list` | `['https://easylist.to/easylist/easylist.txt', 'https://easylist.to/easylist/easyprivacy.txt']` | Block list to be used by ZyteProxyPuppeteer in order to initiate blocker enginer using `@cliqz/adblocker-puppeteer` and block ads |
| `headers` | `{'X-Crawlera-No-Bancheck': '1', 'X-Crawlera-Profile': 'pass', 'X-Crawlera-Cookies': 'disable'}` | List of headers to be appended to requests |

### Notes
Some websites may not work with `block_ads` and `static_bypass` enabled (default). Try to disable them if you encounter any issues.
