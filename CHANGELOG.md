# Release Notes

## Latest changes

## 0.5.0.0

### Major update

This update contains many backwards-incompatible changes

This update is the biggest of all the time, with over 5000 line additions and deletions, more than 121 files changed.

It has finished most of the parts of our Backend Improvements in our roadmap.

There are lots of internal improvements, not all changes are user-facing, but there are lots of critical bug fixes and quality improvements.

This update also fixes some not very critical, but security issues, upgrade is recommended to everyone.

This update starts the era of refreshed, lighter and better BitcartCC.
New features should now be added way faster due to amount of work done on improving maintenance work

There should be no action required to upgrade, everything will be performed automatically.
If it has failed, please let us know.

This changelog contains quite a bit of additional technical information than usual due to the nature of this update.

### Major refactor of all our backend code

We have re-organized all our code into their corresponding directories, so it is now less cluttered, even more readable and easier to add new features (i.e. `crud.py` into `crud` directory, `utils.py` into `utils` directory, utils are now split into files by their category)

All the endpoints were re-organized into logical sub-urls.
See breaking changes below for more information on how to migrate.

Improved texts handling in the database

All imports now use absolute names, code is more readable

Added new utilities for database management and made all the code use it, it will allow easier database access from plugins (to be added later) and scripts.

This allowed us to greatly reduce code duplication, and in the process we found and fixed many bugs.

For example, loading related invoice's products, or store's wallets and notifications is now done automatically, as well as updating references in the database. Before it was duplicated and had some bugs.

As all database access is now done by those functions, in the future we may add additional events or logs for those queries to be used by scripts

In the future those refactors will allow us to batch insert needed data, greatly improving performance for stores with many wallets connected, for example

### Test suite rewriten from scratch, new regtest tests added

Our test suite was rewritten from scratch, now each test doesn't depend on each other, which means that our tests are fully correct and can't pass while some issue still exists.

So now if one test fails the whole test suite doesn't fail with it together.

Improved tests helped us indentify many bugs.

We now have added regtest functional tests which help us test the pay flow, here's how it is done:

We run tests in an isolated environment (fresh database) in regtest network

The test creates a store, then sets transaction_speed (we test all variants), creates an invoice, pays it from the full node

Beforehand it starts a simple http server, sets notification_url to that server's url. Server on POST request just sends it's message later on, and then we check that two IPNs were sent in each case: `paid` status, and `complete` status

That way the whole payment processing workflow is now tested

We now test invoice response structure (i.e. payment methods) better

We now test different validation we have better

Improved CI testing process, it is now possible to publicly view test reports by anyone

Our SDK is now tested in 3 python versions instead of one, with regtest tests running on the base BitcartCC version (currently python 3.7)

### Breaking: Unique string ID

Now all objects IDs are strings.

All previous IDs will remain the same.

Newly generated IDs will be generated from a cryptographically secure source.

This change is breaking, but has been requested for a while, because it greatly improves privacy. It is no longer possible to sequentically scan for all wallet's addresses by opening checkout pages with IDs from 1 to N. It is also not possible to know the exact number of invoices on an instance.

Object id length is 22 for invoices and products, and 32 for other products.

As IDs are now longer than few characters, in the admin panel, for object ids with length > 22 they will be truncated and it will be possible to copy them on click

Payment methods order is now fully guaranteed by having a created date, existing data should be migrated automatically (but the created date of existing methods will not match the actual date)

As All object IDs are now strings, not integers, you should remove integer checks and conversions in your calls to API.

Existing ids will be returned as strings, so instead of id 42 you will receive id "42"

All upgrades should be performed automatically

As unique id is now in place, existing deployments will be unaffected, but on new deployments, the first created store by server admin becomes the default store at the store POS. Before that, store POS displays no store.

### Important security fixes

Before it was possible to create a store with wallets current user doesn't own from API. It didn't leak privacy, but that was allowed. This issue is now fixed.

There was added a check that ensures that all related objects (i.e. wallets connected to store, products connected to invoice, etc) are owned by the current user.

Otherwise, a HTTP 403 Forbidden code is returned and operation is cancelled.

Also, it was possible to create products on store current user does not own, and other similar issues. Unfortunately there were quite a lot of such.

All access control issues were fixed, upgrade is recommended.

We try our best to build the most secure software possible, but we need your help. The bigger our community becomes, the more developers will be available to audit our code and help us identify some issues quickly.

As sometimes it's hard to find unpredictable behaviour in your own code.

Tests were added to proove that no such issues will arise in the future.

### Quality of life improvements

- Improved PATCH methods: to update objects from API no fields are required, there is no need to pass unnecessary fields (`user_id`, `wallets`) to just change store's name for example! Before it was impossible to update objects from API without additional API call to fetch those requires fields. Some fields required weren't required even on object creation. This shouldn't be an issue now.
- Added IPN sending logs: on success it will say that sending IPN succeeded, otherwise it will log an error message to help diagnose issues
- Added better pagination in admin panel to easily navigate between pages. The pagination buttons allow you to jump to the first, last page and some pages inbetween instead of constantly clicking "next" button.

### Breaking: removed PUT http method

We have Removed PUT http methods because they weren't used and not tested enough, therefore they could lead to unexpected issues if used.

Also see the PATCH method improvements

### Breaking: renamed endpoints

Due to our refactors and re-organization, here's a list of renamed endpoints:

`/rate` -> `/cryptos/rate`

`/fiatlist` -> `/cryptos/fiatlist`

`/categories` -> `/products/categories`

`/services` -> `/tor/services`

`/updatecheck` -> `/update/check`

`/crud/stats` -> `/users/stats`

`/wallet_history` -> `/wallets/history`

Note: it is no longer possible to open `/wallets/history/0` to get wallet history for all wallets on your account, added a new endpoint for that:

`/wallets/history/all`

### Misc changes

- Fixed search in admin panel in pages other than invoices page
- Fixed objects template editing
- Better logo rendering in dark mode
- New SDK method: get_invoice to get lightning invoice data
- Fixed rare bug with error on editing store checkout settings
- Fixed an issue with wallet's xpub validation not always running when creating from API
- PATCH: fixed issues with modifying store checkout settings from API, changes only changed settings, others remain unaffected (before they were reset to defaults)
- Disallowed modifying invoice products after creation
- When creating notification providers, we now validate that notification provider selected is supported, otherwise the operation is cancelled (to avoid issues with not-existing providers on invoice completion)
- Fixed policies update endpoints' returning incorrect response data (fields not updated) sometimes
- Fixed an issue where /wallet_history returned wallet history of all wallets on a server, not of current user
- Fixed bitcart-cli builds

## 0.4.0.0

### Major update

This update contains backwards-incompatible changes

### Changing current instance's settings via admin panel

It is now possible to change current instance's settings via admin panel's configurator.

Editing current instance settings (and pre-loading them via configurator) is only available to server admins.

By clicking on current instance mode, if you are server admin, your current settings will be loaded and filled in the form fields.

The app will automatically connect to your server via SSH and apply new settings.

Note: it is not possible to view deployment log when updating your current instance, as the process performing the update gets restarted.

Note: for that to work you should have working SSH support, see below.

Note that even though configurator should fit most use cases, if you are not using one-domain mode, or if you
have some completely custom and complex use-case, possibly involving multiple deployments on the same server, you should better
use setup scripts from CLI.

### Breaking: SSH support in setup scripts

Old method of executing commands on the host (for maintenance purposes, like updating the server) is no longer used.

It means that there will be no listener process started anymore.

Instead, both maintenance commands and configurator's current instance mode use SSH support.

When running `./setup.sh`, BitcartCC configures itself to use system host keys.

On first startup, it generates an SSH key, and adds it to the list of trusted keys in the host (usually `~/.ssh/authorized_keys`)

That way, it can connect to the host via ssh, which is a way better way of executing commands, which opens doors to new possibilities.

Note: SSH support is only enabled when `BITCART_ENABLE_SSH` is set to `true`, by default it is so.

Note: SSH support requires an ssh server (`openssh-server`/`sshd`) to be running on the host machine.

**IMPORTANT**: For existing deployments, after updating, for future updates to work, you will need to re-run `./setup.sh`

### Many improvements in the configurator

In Remote deployment mode, there is now a button "Load settings", which will connect to the server via SSH and fill in it's settings
in the form fields.

Configurator should now be responsive and look better on mobile devices.

Configurator now removes stale tasks data from redis (if it was created more than a day ago).

### Other improvements

- Fix cleanup command
- Fix bugs in configurator
- Improve wallet loading logic in BitcartCC Core daemons
- Added support for build-time environment variables to docker-compose generator
- Maintenance updates for dependencies

## 0.3.1.1

Fixed configurator long-running deployments

## 0.3.1.0

BitcartCC Configurator now supports installing to remote servers via ssh!

Added restart server management command

Added ability to make email optional on store POS

Improved additional components handling in the docker deployment, and added ability to preview settings

Fixed stats refresh in the admin panel

Made the tor services page more clear

## 0.3.0.1

Fixed underpaid amounts calculation

Fixed admin panel in one-domain mode

## 0.3.0.0

### Major update

This update contains backwards-incompatible changes

### Completely improved docker deployment

We now test if deployment works in our automated systems, so that it will always work

Set up formatting and proper linting

Major improvements to the setup scripts:

New scripts:

- `restart.sh`, to restart the server
- `changedomain.sh`, to change domain (usage: `./changedomain.sh newdomain.tld`)

Added more validation to `setup.sh` (i.e. it is not possible to enter an invalid host anymore)

Added ability to change the root path where service is running by setting `BITCART_SERVICE_ROOTPATH` (i.e. `BITCART_STORE_ROOTPATH`)

Added new settings to configure nginx reverse proxy:

- `REVERSEPROXY_HTTP_PORT` - the http port nginx is running on (default 80)
- `REVERSEPROXY_HTTPS_PORT` - the https port nginx is running on (443)
- `REVERSEPROXY_DEFAULT_HOST` - the host to be served by default from server ip (default: none)

Overall generator refactor

#### One domain support

Existing deployments will be unaffected.

If reverse proxy is enabled and `BITCART_ADMIN_HOST` and `BITCART_STORE_HOST` and `BITCART_ADMIN_API_URL` and `BITCART_STORE_API_URL` are all unset, one domain mode is enabled.

For one domain mode, only one setting is used: `BITCART_HOST`.

It will determine the only domain bitcartcc will run on.

The 3 main services will run under different routes.

There is a root service, running at domain root. The root service is selected in the following order (if available): store, admin, api

By default, assuming `BITCART_HOST` was `bitcart.local`:

- the store will run on `bitcart.local`
- admin on `bitcart.local/admin`
- api on `bitcart.local/api`

Everything will be configured to work on one domain.

To enable one domain mode for existing deployments:

```bash
unset BITCART_ADMIN_HOST
unset BITCART_STORE_HOST
unset BITCART_ADMIN_URL
unset BITCART_STORE_URL
./setup.sh
```

#### Breaking change to improve readability

The following environment variables were renamed to reduce confusion:

- `BITCART_ADMIN_URL` -> `BITCART_ADMIN_API_URL`
- `BITCART_STORE_URL` -> `BITCART_STORE_API_URL`
- `BITCART_ADMIN_ONION_URL` -> `BITCART_ADMIN_API_ONION_URL`
- `BITCART_STORE_ONION_URL` -> `BITCART_STORE_API_ONION_URL`

Please set them in order for your deployment to work

### BitcartCC Configurator

This release included an alpha version of BitcartCC Configurator.

For now only manual deployment is supported.

BitcartCC Configurator is an application in your admin panel, allowing to install new BitcartCC instances (via ssh or manual generated script) and to re-configure them with ease.

Just enter needed settings and you will get a copiable script.

By default it can be accessed by anonymous users.

Added a new server policy to make it available for authorized users only (default: False)

## 0.2.3.1

Fix invoice creation for fiat currencies without a symbol

Don't create duplicate make expired tasks on startup

## 0.2.3.0

Maintenance release. Use deterministic requirements, decrease docker image sizes, fix some fiat currencies not being displayed

## 0.2.2.0

This release is mostly a bugfix release, but with a few new features too.

### Full invoice tasks handling refactor

Now there is only one expired task created for an invoice, instead of N (where N is the number of payment methods).

It means that BitcartCC should use less RAM, and also there will be less edge cases and duplicate IPN sent.

Also, to handle some rare cases if gunicorn workers are restarting, we have changed the way background tasks are handled.

Technical details:
The idea is to make every background task, and every asyncio task run in background worker, and not gunicorn ones.

Background worker listens on a redis channel for events, and gunicorn workers publish messages to the channel. Worker parses messages and executes event handlers in separate asyncio tasks.

### Fixed local deployment

Now local deployment via .local domains works as before, and it now modifies /etc/hosts in a clever way, avoiding duplicate entries

### Other changes

- Admin panel now displays a helpful message when invoice has no payment methods connected
- Fixed recommended fee in case it's unavailable.
- Tor extension now doesn't log always, instead it logs only warnings if something was misconfigured.
- Fixed IPN sending
- Fixed logging in docker environment
- Fixed pagination for id 0
- Added ability to change fiat currency used in the /rate endpoint
- Fixed websockets' internal channel ids clash sometimes

## 0.2.1.1

Fix multiple store support on POS

## 0.2.1.0

Fix image uploading

Allow decimal values for `underpaid_percent`

Multiple stores on one store POS instance - new `/store/{id}` URLs added to serve any store

Default store served is still configured by server policies

## 0.2.0.2

Bugfixes in recommended fee calculation (GZRO)

## 0.2.0.1

Bugfixes in recommended fee calculation (BCH)

## 0.2.0.0

### Major update

This update contains numerous changes, some of which are not backwards-compatible

### New payment statuses

This update completely changes our status system, by adding new statuses: `paid` and `confirmed`.

`Pending` status was renamed to `pending`

New store setting, `transaction_speed` added, which controls when should invoice be marked as complete

New transaction flow:

Invoice created -> pending

Payment received -> paid

Payment has >= 1 confirmation -> confirmed

Payment number of confirmations is >= `transaction_speed` of the store OR it is a lightning invoice -> complete

Payment expired -> expired

If payment was detected within the invoice time frame, and the payment expired, it won't be set to expired but instead wait for confirmations.

For more details read this:
https://docs.bitcartcc.com/guides/transaction-speed

### A lot of new store checkout settings

- Underpaid invoices support. For example, if customer sends from an exchange wallet, it might deduct the fees from amount sent. This way you can accept customer's invoice.

  More details [here](https://docs.bitcartcc.com/support-and-community/faq/stores-faq#what-is-underpaid-percentage)

- Custom logo support. [Details](https://docs.bitcartcc.com/support-and-community/faq/stores-faq#what-is-custom-logo-link)
- Dark mode support. [Details](https://docs.bitcartcc.com/support-and-community/faq/stores-faq#what-is-the-use-dark-mode-setting)
- Recommended fee support. Recommended fee will be displayed in all onchain payments methods. [Details](https://docs.bitcartcc.com/support-and-community/faq/stores-faq#recommended-fee)

### Multiple wallets of the same currency support

Before, it was disallowed to create an invoice with multiple wallets of the same currency (only one was picked).

Now it is allowed, and in the checkout, payment methods will be indexed.
For example, if you have 2 btc and 2 ltc wallets connected with lightning enabled, here's how it would look:

- BTC (1)
- BTC (⚡) (1)
- BTC (2)
- BTC (⚡) (2)
- LTC (1)
- LTC (⚡) (1)
- LTC (2)
- LTC (⚡) (2)

A new `name` attribute was added to PaymentMethod's structure, which contains pre-formatted payment method name ready for display

### Maintenance

All dependencies of packages has been upgraded, and two maintenance releases were made: of SDK, to make new release with a new license,
and of BitCCL, to fix it's use together with SDK 1.0

### Quality of life improvements

- By pressing enter in the edit dialog, it will be automatically saved (like in login page, enter to login)
- Added new mark complete batch action, to be able to mark some invoices complete manually. All notifications, emails and scripts will be executed.
- Added filters to admin panel, it is now possible to easily filter out paid or invalid invoices

### Misc changes

- Fixed payment methods order being inconsistent sometimes
- Fixed scrollbars in checkout page being shown on different screen resolutions
- Tor extension logs are now DEBUG level instead of INFO (less log spam)
- Fixed patch/put methods, it is now possible to completely disconnect notifications from store
- Fixed stale expired invoices occuring in rare cases

## 0.1.0.2

Fixed html template rendering and product price is now properly formatted when using it in templates
Fixed UI issues with the new checkout in Firefox.
Details are now displayed differently in admin panel:
The name of the field is on one line, and the actual data on the next line.
Template's text is now hidden into item details.

## 0.1.0.1

Small bugfixes

Various visual, performance improvements and fixes in the admin panel (now using vuetify 2.3.x)

## 0.1.0.0

### Major update

This update contains numerous changes, some of which are not backwards-compatible

### Lightning network checkout

Lightning has been supported in BitcartCC Core Daemons for a long time, but not in our payment methods.

Now BitcartCC fully supports Lightning Network as a checkout method too!

Lightning Network is supported via BitcartCC daemons's lightning mode.

It means that, there is still no need to install additional software, lightning is enabled just by one setting.

To enable lightning, just run:

```bash
export COIN_LIGHTNING=true
```

For each `COIN` (i.e. `BTC`, `LTC`) where you want to enable lightning support.

It will start a lighning gossip in the daemon, and enable the ability to perform lightning operations.

Lightning is currently only supported in native segwit wallets (electrum implementation limitation).

To enable lightning, visit wallets page, click on lightning icon, read all the warnings displayed.

Currently there are lots of limitations and issues in the Lightning Network itself, and the Electrum implementation, so proceed only if you understand that you may risk losing your funds, as Lightning Network is experimental.

But if you agree, press the green button, and lightning in the wallet will be enabled.

Lightning network is supported individually in each wallet, so even non-admin users can utilize it!

Each wallet can be considered an individual lightning node, or, better said, an account on the same lightning node.

So for each wallet you will have to open new lightning channels.

As lightning implementation is built-in in BitcartCC, it is not possible to use other implementations, and you will have to open channels from scratch.

Please read the warning in the admin panel for more details.

When enabled, and when creating an invoice, along with the regular currency checkout method, a lightning one will be created too.

On checkout, the customer will be able to scan/copy lightning invoice, and your node id, if they need to open a channel with you.

When paid, instantly you will see that invoice has been paid.

As every wallet is a fresh node, you will need a way to manage your lightning channels.

If you click on lightning icon near the wallet again, you will see a lightning management page.

It will display your lightning balance, node id and your lightning channels.

You are able to close or force-close channels from there.

Also you can open a new lightning channel or pay an lightning invoice from your lightning node in that same page.

When invoice has been paid, you will see the paid currency will have the lightning icon near it.

Also, all paid currencies are now upper-case, your existing data will be migrated.

To support lightning network, invoice data returned from API was changed.

`payments` key is now a list, and not a dictionary.

There are other changes to the API, please read the other release features below.

Also, new endpoints were added:

- `/wallets/{wallet_id}/checkln` endpoint to check if lightning is supported for a wallet.
  Returns False if not supported, node id otherwise

- `/wallets/{wallet_id}/channels` endpoint, returning wallet's lightning channels
- `/wallets/{wallet_id}/channels/open` endpoint, allowing to open a new lightning channel
- `/wallets/{wallet_id}/channels/close` endpoint, allowing to close a lightning channel
- `/wallets/{wallet_id}/lnpay`, allowing to pay a lightning invoice from your lightning node

To support lightning network, the daemon now has a new method: `get_invoice`, to get invoice by it's `rhash`.

### Improved checkout page

To support lightning, as there are now even more checkout methods, will have listened to your suggestions, so now, currency selection is not tab-like,
but it's a select list.

Changed the checkout modal layout, now it is more minimalistic and takes less space.

The exchange rate is now displayed in a pretty rate, and formatted correctly.

Added open in wallet button, which should launch user's application to automatically paste all the details to perform a payment.

Instead of displaying qr code and texts at once, and, considering that lightning invoices are big, we now have split the checkout into two tabs:
Scan and Copy.

Scan tab will show a qr code

and Copy tab will display the texts, but as lightning invoices are long, it will display only parts of it, and by clicking on the fields, the full text
will be copied

When on a lightning payment method, you will be able to switch between lightning invoice qr code and node id qr code.

The checkout page in the store was changed in a similar way.

### Added help texts to some fields

To help newcomers, some special fields now have a question mark icon near them, by clicking on which, user will be redirected to our docs.

It should help to understand some basics without asking, right from your instance.

### Added ID field everywhere

As requested, ID field was added to each datatable. It was possible to copy id before by clicking on copy icon, but now it's also displayed as a field

### Better money formatting

Do you remember weird amounts in the checkout before, like

1 BTC = 25000.424242 USD

When USD can have maximum of 2 decimal places?

It is finally fixed!

Now, every amount will be formatted as per it's currency settings.

Also, the rate string is now fancier than before, it can contain currency symbol and it's code.

There are some breaking changes to make it working

Now every amount in the payment methods is not a Decimal, but a string, which was already formatted for you.

Also, every payment method now has a `rate` field, which will contain a computer-readable formatted Decimal of the exchange rate at the moment of invoice creation.

Also every payment method now has a `rate_str` field, which is a human-readable pretty-formatted rate string, ready to be used in checkout

### Major task system and logging refactor

Now all background tasks are running only in the worker
process, and not in every worker ran by gunicorn

Instead of using local process state, fetched and parsed data is now
placed in redis, and fetched from there by workers serving requests.

Now, instead of each process logging concurrently, which lead to race
condition (like, having logs from yesterday contain today's logs), each worker uses sends request to logging
server, which is ran as part of the main background worker.

What does it mean?

It means that BitcartCC deployments should use way less RAM, should have less issues and work more stable.

Plus it means that logging system now works correctly

Logs in the admin panel are now sorted, and it is now possible to remove individual logs or clean them all.

Tor extension is now refreshed every 15 minutes instead of every 2 minutes to reduce logs spam.

### Added configuration samples and documented every setting

Every setting supported by BitcartCC that is configured by an environment variable is supported.

Now, the conf directory of your cloned BitcartCC repo contains more samples:

- .env.sample was cleaned up from unused settings, and now every setting is documented
- .env.dev.sample was added, which contains the configuration we use when developing BitcartCC

### Misc changes

- Fixed worker start on Mac OS
- Pending invoice processing will no longer stop processing on error in one of the invoices
- Added wallet balance endpoint, `/wallets/{wallet_id}/balance`, which returns a dictionary with 4 keys:
  `confirmed`, `unconfirmed`, `lightning` and `unmatured`.
- Various bugfixes and performance improvements
- Added an easter egg (:

## 0.0.0.3

### Logging system

We now have logging system set up for our merchants API.

It means that it'll be easier to debug issues, as it is now possible to view logs from admin panel.

You can view logs (collected each day), and download them if needed for others to help to fix your issue.

### Admin panel responsibility improvements

Admin panel should now work better on mobile, especially in server management page.

### Other changes

Search input in admin panel now no longer searches all related tables, but only the one you are currently viewing.

Fixed a bug where if you have passed products=`None` to API it would crash.

Update check now doesn't even start to run if update url is not set

Admin panel's wallets creation dialog currency field is now a dropdown-to avoid entering invalid fields.

## 0.0.0.2

Removed non-working settings page from admin panel

## 0.0.0.1

First BitcartCC release with a version!
