<p>
<img src="https://apitoolkit.io/assets/img/logo-full.svg" alt="APIToolkit" width="250px" />
</p>

## APIToolkit expressjs integration.

The NODEJS SDK integration guide for APIToolkit. It monitors incoming traffic, gathers the requests and sends the request to the apitoolkit servers.

### Installation

Run the following command to install the package from your projects root:

```sh
npm install apitoolkit-express

```

### Project setup

Intialize apitoolkit into your project is as simple as :

```js
import APIToolkit from 'apitoolkit-express';

const apitoolkitClient = await APIToolkit.NewClient({ apiKey: '<API-KEY>' });
```

where `<API-KEY>` is the API key which can be generated from your [apitoolkit.io](apitoolkit.io) accoun

Next, you can use the apitoolkit middleware for your respective routing library.

Eg, for express JS, your final code would look like:

```js
app.use(apitoolkitClient.expressMiddleware);
```

where app is your express js instance.

Your final could might look something like this especially on typescript:

```js
import APIToolkit from 'apitoolkit-express';
import express from 'express';

const port = 3000;
const apitoolkit = await APIToolkit.NewClient({ apiKey: '<API-KEY>' });
app.use(apitoolkit.expressMiddleware);

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`);
});
```

If you're unable to use await at the top level, then you could wrap your apitoolkit and express initialization logic in a closure.
Also notice the `.default` at the end of the require, to access the default export of the SDK.

```js
const APIToolkit = require('apitoolkit-express').default;
const express = require('express');

(async function () {
  const port = 3000;
  const apitoolkit = await APIToolkit.NewClient({ apiKey: '<API-KEY>' });
  app.use(apitoolkit.expressMiddleware);

  app.get('/', (req, res) => {
    res.send('Hello World!');
  });

  app.listen(port, () => {
    console.log(`Example app listening on port ${port}`);
  });
})();
```

## Redacting Senstive Fields and Headers

While it's possible to mark a field as redacted from the apitoolkit dashboard, this client also supports redacting at the client side. Client side redacting means that those fields would never leave your servers at all. So you feel safer that your sensitive data only stays on your servers.

To mark fields that should be redacted, simply add them to the apitoolkit config object. Eg:

```js
const APIToolkit = require('apitoolkit-express').default;
const express = require('express');

(async function () {
  const app = express();
  const port = 3000;
  const apitoolkitClient = await APIToolkit.NewClient({
    apiKey: '<API-KEY>',
    redactHeaders: ['Content-Type', 'Authorization', 'Cookies'], // Specified headers will be redacted
    redactRequestBody: ['$.credit-card.cvv', '$.credit-card.name'], // Specified request bodies fields will be redacted
    redactResponseBody: ['$.message.error'], // Specified response body fields will be redacted
  });
  app.use(apitoolkitClient.expressMiddleware);

  app.get('/', (req, res) => {
    res.send('Hello World!');
  });

  app.listen(port, () => {
    console.log(`Example app listening on port ${port}`);
  });
})();
```

It is important to note that while the `redactHeaders` config field accepts a list of headers(case insensitive), the `redactRequestBody` and `redactResponseBody` expect a list of JSONPath strings as arguments.

The choice of JSONPath was selected to allow you have great flexibility in descibing which fields within your responses are sensitive. Also note that these list of items to be redacted will be aplied to all endpoint requests and responses on your server. To learn more about jsonpath to help form your queries, please take a look at this cheatsheet: https://lzone.de/cheat-sheet/JSONPath

## Handling File Uploads with Formidable

Working with file uploads using the `multer` package is quite straightforward and requires no manual intervention, making it seamless to send multipart/form-data requests to APIToolkit.

However, if you choose to employ `formidable` for managing file uploads, a more hands-on approach becomes necessary to ensure proper data transmission to APIToolkit. Without manual intervention, no data is dispatched, potentially hindering the accurate monitoring of the endpoint. To enable this functionality, developers must attach both the `fields` and `files` extracted from the `form.parse` method to the request object.

For instance:

```js
import express from 'express';
import APIToolkit from 'apitoolkit-express';
import formidable from 'formidable';

const app = express();
const client = await APIToolkit.NewClient({
  apiKey: '<API_KEY>',
});

app.use(client.expressMiddleware);

app.post('/upload-formidable', (req, res, next) => {
  const form = formidable({});
  form.parse(req, (err, fields, files) => {
    // Attach fields to request body
    req.body = fields;
    // Attach files
    req.files = files;

    res.json({ message: 'Uploaded successfully' });
  });
});

app.listen(3000, () => {
  console.log('Server is running on port 3000');
});
```

By executing this procedure, APIToolkit gains access to non-redacted fields and files, thereby enhancing the precision of monitoring and documentation processes. This method ensures that all necessary data is accessible and correctly relayed to APIToolkit for thorough analysis and documentation.
