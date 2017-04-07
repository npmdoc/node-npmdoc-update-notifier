# api documentation for  [update-notifier (v2.1.0)](https://github.com/yeoman/update-notifier#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-update-notifier.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-update-notifier) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-update-notifier.svg)](https://travis-ci.org/npmdoc/node-npmdoc-update-notifier)
#### Update notifications for your CLI app

[![NPM](https://nodei.co/npm/update-notifier.png?downloads=true)](https://www.npmjs.com/package/update-notifier)

[![apidoc](https://npmdoc.github.io/node-npmdoc-update-notifier/build/screenCapture.buildNpmdoc.browser.%2Fhome%2Ftravis%2Fbuild%2Fnpmdoc%2Fnode-npmdoc-update-notifier%2Ftmp%2Fbuild%2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-update-notifier/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-update-notifier/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-update-notifier/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Sindre Sorhus",
        "email": "sindresorhus@gmail.com",
        "url": "sindresorhus.com"
    },
    "bugs": {
        "url": "https://github.com/yeoman/update-notifier/issues"
    },
    "dependencies": {
        "boxen": "^1.0.0",
        "chalk": "^1.0.0",
        "configstore": "^3.0.0",
        "is-npm": "^1.0.0",
        "latest-version": "^3.0.0",
        "lazy-req": "^2.0.0",
        "semver-diff": "^2.0.0",
        "xdg-basedir": "^3.0.0"
    },
    "description": "Update notifications for your CLI app",
    "devDependencies": {
        "clear-require": "^2.0.0",
        "fixture-stdout": "^0.2.1",
        "mocha": "*",
        "strip-ansi": "^3.0.1",
        "xo": "^0.17.0"
    },
    "directories": {},
    "dist": {
        "shasum": "ec0c1e53536b76647a24b77cb83966d9315123d9",
        "tarball": "https://registry.npmjs.org/update-notifier/-/update-notifier-2.1.0.tgz"
    },
    "engines": {
        "node": ">=4"
    },
    "files": [
        "index.js",
        "check.js"
    ],
    "gitHead": "86ca238959d8176929499f69a31cb0ddb0c74e96",
    "homepage": "https://github.com/yeoman/update-notifier#readme",
    "keywords": [
        "npm",
        "update",
        "updater",
        "notify",
        "notifier",
        "check",
        "checker",
        "cli",
        "module",
        "package",
        "version"
    ],
    "license": "BSD-2-Clause",
    "maintainers": [
        {
            "name": "sindresorhus",
            "email": "sindresorhus@gmail.com"
        },
        {
            "name": "addyosmani",
            "email": "addyosmani@gmail.com"
        },
        {
            "name": "passy",
            "email": "phartig@rdrei.net"
        },
        {
            "name": "sboudrias",
            "email": "admin@simonboudrias.com"
        },
        {
            "name": "eddiemonge",
            "email": "eddie+npm@eddiemonge.com"
        },
        {
            "name": "arthurvr",
            "email": "contact@arthurverschaeve.be"
        }
    ],
    "name": "update-notifier",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/yeoman/update-notifier.git"
    },
    "scripts": {
        "test": "xo && mocha --timeout 20000"
    },
    "version": "2.1.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module update-notifier](#apidoc.module.update-notifier)
1.  [function <span class="apidocSignatureSpan">update-notifier.</span>UpdateNotifier (options)](#apidoc.element.update-notifier.UpdateNotifier)



# <a name="apidoc.module.update-notifier"></a>[module update-notifier](#apidoc.module.update-notifier)

#### <a name="apidoc.element.update-notifier.UpdateNotifier"></a>[function <span class="apidocSignatureSpan">update-notifier.</span>UpdateNotifier (options)](#apidoc.element.update-notifier.UpdateNotifier)
- description and source-code
```javascript
class UpdateNotifier {
	constructor(options) {
		this.options = options = options || {};
		options.pkg = options.pkg || {};

		// Reduce pkg to the essential keys. with fallback to deprecated options
		// TODO: Remove deprecated options at some point far into the future
		options.pkg = {
			name: options.pkg.name || options.packageName,
			version: options.pkg.version || options.packageVersion
		};

		if (!options.pkg.name || !options.pkg.version) {
			throw new Error('pkg.name and pkg.version required');
		}

		this.packageName = options.pkg.name;
		this.packageVersion = options.pkg.version;
		this.updateCheckInterval = typeof options.updateCheckInterval === 'number' ? options.updateCheckInterval : ONE_DAY;
		this.hasCallback = typeof options.callback === 'function';
		this.callback = options.callback || (() => {});

		if (!this.hasCallback) {
			try {
				const ConfigStore = configstore();
				this.config = new ConfigStore('update-notifier-${this.packageName}', {
					optOut: false,
					// Init with the current time so the first check is only
					// after the set interval, so not to bother users right away
					lastUpdateCheck: Date.now()
				});
			} catch (err) {
				// Expecting error code EACCES or EPERM
				const msg =
					chalk().yellow(format(' %s update check failed ', options.pkg.name)) +
					format('\n Try running with %s or get access ', chalk().cyan('sudo')) +
					'\n to the local update config store via \n' +
					chalk().cyan(format(' sudo chown -R $USER:$(id -gn $USER) %s ', xdgBasedir().config));

				process.on('exit', () => {
					console.error('\n' + boxen()(msg, {align: 'center'}));
				});
			}
		}
	}
	check() {
		if (this.hasCallback) {
			this.checkNpm()
				.then(update => this.callback(null, update))
				.catch(err => this.callback(err));
			return;
		}

		if (
			!this.config ||
			this.config.get('optOut') ||
			'NO_UPDATE_NOTIFIER' in process.env ||
			process.argv.indexOf('--no-update-notifier') !== -1
		) {
			return;
		}

		this.update = this.config.get('update');

		if (this.update) {
			this.config.delete('update');
		}

		// Only check for updates on a set interval
		if (Date.now() - this.config.get('lastUpdateCheck') < this.updateCheckInterval) {
			return;
		}

		// Spawn a detached process, passing the options as an environment property
		spawn(process.execPath, [path.join(__dirname, 'check.js'), JSON.stringify(this.options)], {
			detached: true,
			stdio: 'ignore'
		}).unref();
	}
	checkNpm() {
		return latestVersion()(this.packageName).then(latestVersion => {
			return {
				latest: latestVersion,
				current: this.packageVersion,
				type: semverDiff()(this.packageVersion, latestVersion) || 'latest',
				name: this.packageName
			};
		});
	}
	notify(opts) {
		if (!process.stdout.isTTY || isNpm() || !this.update) {
			return this;
		}

		opts = Object.assign({isGlobal: true}, opts);

		opts.message = opts.message || 'Update available ' + chalk().dim(this.update.current) + chalk().reset(' → ') +
			chalk().green(this.update.latest) + ' \nRun ' + chalk().cyan('npm i ' + (opts.isGlobal ? '-g ' : '') + this.packageName) + '
to update';

		opts.boxenOpts = opts.boxenOpts || {
			padding: 1,
			margin: 1,
			align: 'center',
			borderColor: 'yellow',
			borderStyle: 'round'
		};

		const message = '\n' + boxen()(opts.message, opts.boxenOpts);

		if (opts.defer === false) {
			console.error(message);
		} else {
			process.on('exit', () => {
				console.error(message);
			});

			process.on('SIGINT', () => {
				console.error('');
				process.exit();
			});
		}

		return this;
	}
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
