# Vite Helper for Kirby

Use the `vite` helper function to get the correct path to your versioned css and js files generated by [vite](https://github.com/vitejs/vite). This plugin is highly inspired by [Diverently](https://github.com/Diverently)'s [Laravel Mix Helper for Kirby](https://github.com/Diverently/laravel-mix-kirby) and [andrefelipe](https://github.com/andrefelipe)'s [vite-php-setup](https://github.com/andrefelipe/vite-php-setup).

## Installation

### Manual download
Download and copy this repository to `site/plugins/kirby-vite`.

### Git submodule
```
git submodule add https://github.com/arnoson/kirby-vite.git site/plugins/kirby-vite
```

### Composer
```
composer require arnoson/kirby-vite
```

## Usage
Make sure you have the right [setup](#setup).
Then inside your template files (or anywhere else) you can use the `vite` helper function.
```html
<html>
  <head>
    <?= vite('@style') ?> <!-- or vite('index.css') -->
    <?= vite('@client') ?>
  </head>
  <body>
    <?= vite('@entry') ?> <!-- or vite('index.js') -->
  </body>
</html>
```

## Setup

### Install Vite
Follow the instructions [here](https://vitejs.dev/guide/).

### Folder structure
With the default configuration, `kirby-vite` expects a folder structure like this:
```
.
├── public # Your complete kirby site.
│   ├── assets -> src/assets # A symlink to your assets folder.
│   ├── dist # Here goes everything that is automatically generated by vite for production.
│   │   ├── manifest.json # The manifest.json generated by vite.
│   │   └── assets # Your compiled assets.
│   ├── content
│   ├── kirby
│   ├── media
│   ├── site
│   └── ...
├── src # Your js and css source files.
│   ├── assets # Place your assets (images, fonts, ...) here.
│   ├── index.js
│   └── index.css
├── vite.config.js
├── node_modules
└── package.json

```

### Vite configuration
To match the folder structure described above, your `vite.config.js` should look something like this:
```js
// vite.config.js
export default {
  // `root` is important here if you want to use the name `public` for your
  // kirby folder. Vite treats the `public` folder special so in order to
  // prevent vite and kirby from clashing we have to make sure vite doesn't use
  // the project root. If you don't want to use the name `public` or put your
  // `vite.config.js` inside the `src` folder you don't have to do this.  
  root: 'src',

  // Make sure kirby uses the correct path in production.
  base: process.env.APP_ENV === 'development' ? '/' : '/dist/',

  server: {
    // Required to load scripts from custom host.
    cors: true,
    // Resolve paths correctly.
    hmr: { host: 'localhost' },
    // Make sure the ports are matching.
    port: 3000,
    strictPort: true,
  },

  build: {
    // Output directory for production build.
    outDir: '../public/dist',
    emptyOutDir: true,
    // Emit manifest so `kirby-vite` can find the hashed files.
    manifest: true,
    // Entry file.
    rollupOptions: {
      input: './src/index.js'
    }    
  }
}
```

### Development
The last thing we have to do is let `kirby-vite` know when we are in development mode, so it can serve the right files.
To do this, add a kirby config file for your development domain ([learn more](https://getkirby.com/docs/guide/configuration#multi-environment-setup) about multi-environment setup).
Assuming you're running your development server on `my-site.test`:
```
├── config # Your kirby config folder
│   └── config.my-site.test.php
└── ...
```
```php
<?php
// config.my-site.test.php
return [
  // ...
  'arnoson.kirby-vite' => [
    'dev' => true
  ]
];
```
Now run `npm dev` (see [NPM scripts](#NPM-scripts)) and visit your kirby site in the browser. Note: we only use vite's dev server to serve our assets. So you can visit your kirby site under your php development server (in this example `my-site.test`).

## NPM scripts
When you created your vite app via `npm init @vitejs/app` (see [install vite](#-install-vite)), `vite` adds the following NPM scripts to your `package.json`:
```json
"scripts": {
  "dev": "vite",
  "build": "vite build",
  "preview": "vite serve" 
}
```
We have to alter them like this to get our configuration to work:
```json
"scripts": {
  "dev": "APP_ENV=development vite",
  "build": "APP_ENV=production vite build"
  // The `preview` script can be deleted.
}
```

## Watch php files
If you also want to live reload your site in development mode whenever you change something in kirby, install [vite-plugin-live-reload](https://github.com/arnoson/vite-plugin-live-reload). Then adjust your `vite.config.js`:
```js
// vite.config.js
import liveReload from 'vite-plugin-live-reload'

export default {
  // ...
  plugins: [
    liveReload('../public/site/(templates|snippets|controllers|models)/**/*.php'),
  ]
}
```

## Known issues
### Vite's dev server port
Make sure `vite` and `kirby-vite` are using the same port. `vite` will use port
`3000` by default. But if you have another `vite` dev server running, `vite` will
increase the port (`3001` and so on). So either make sure no other `vite` dev
server is running or specify another port in `kirby-vite`:
```php
'arnoson.kirby-vite.devServer' => 'http://localhost:1234'
```

### Static assets during development
To get static assets (images, fonts, ...) working during development you have to
create a symlink on your php dev server to match your assets folder:
```
cd {root_of_your_project}
ln -s $PWD/src/assets ./public/assets
```
More information [here](https://vitejs.dev/guide/backend-integration.html).