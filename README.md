<div align="center">

# HIKARI ./PULSE

### Cyberpunk-inspired MyXL command-line toolkit

An unofficial, community-maintained CLI for exploring MyXL account information, packages, family features, store listings, and supported purchase flows — with an optional Telegram bot.

[![Platform](https://img.shields.io/badge/platform-Termux%20%7C%20Linux%20%7C%20OpenWrt-00D9FF?style=for-the-badge)](#-platform-support)
[![Python](https://img.shields.io/badge/Python-3.10%2B-00D9FF?style=for-the-badge&logo=python&logoColor=white)](#-requirements)
[![Unofficial](https://img.shields.io/badge/status-unofficial-9B5DE5?style=for-the-badge)](#-disclaimer)

**Repository:** [github.com/nexterade/hikari-pulse](https://github.com/nexterade/hikari-pulse)

</div>

---

## ◈ Overview

**HIKARI ./PULSE** is a fork of [taamarin/cli](https://github.com/taamarin/cli). It provides an interactive terminal interface for supported MyXL account and package operations.

The project can be used directly from a terminal. An optional Telegram bot is also included for a limited set of account, package, bookmark, and purchase workflows.

> This README documents the functionality currently present in the source tree. Planned interface redesigns and features are not described as implemented features.

## ◈ Features

### Interactive CLI

- Login and switch between saved accounts.
- View currently active packages.
- Browse package details using an option code.
- Browse packages by family code.
- Run the family-code purchase loop with configurable starting option, delay, and prompts.
- View transaction history.
- Access Family Plan / Akrab Organizer and Circle information.
- Browse Store Segments, Store Family List, Store Packages, and Redeemables.
- Register and validate an MSISDN.
- Manage package bookmarks.
- View notifications.

### Telegram bot

The optional bot uses the Telegram Bot API directly and is restricted to one configured Telegram user ID in a private chat.

Available commands:

| Command | Purpose |
|---|---|
| `/login` | Login using an XL number and SMS OTP |
| `/balance` | View balance and active period |
| `/packages` | List active packages |
| `/family <family_code>` | List package options for a family code |
| `/bookmark <family_code> <no>` | Save a package bookmark |
| `/bookmarks` | List saved bookmarks |
| `/unbookmark <no>` | Remove a bookmark |
| `/buy <family_code> <no>` | Start a package purchase |
| `/buybookmark <no>` | Start a purchase from a bookmark |
| `/confirm balance` | Confirm using balance |
| `/confirm qris` | Confirm using QRIS |
| `/confirm ewallet <provider> [number]` | Confirm using an e-wallet |
| `/menu` | Show the Telegram feature keyboard |
| `/cancel` | Cancel the current flow |
| `/help` | Show command help |

Supported e-wallet names in the command flow: `dana`, `shopeepay`, `gopay`, and `ovo`. DANA and OVO require a phone number. QRIS payment information is delivered as a QR image.

**Important:** the CLI and Telegram bot share local account state (`refresh-tokens.json`). Do not run both at the same time.

## ◈ Platform support

| Environment | Notes |
|---|---|
| Android / Termux | Supported installation path; use Termux packages for native dependencies such as Pillow |
| Linux | Requires Python 3.10+ and pip |
| OpenWrt | `setup.sh` has package-manager handling for `opkg` and `apk`; availability depends on the device's repositories and architecture |

The setup script checks for Python 3.10 or newer. This version check does not guarantee that every dependency has a prebuilt wheel for every Python/OS combination.

## ◈ Requirements

- Python 3.10 or newer
- pip
- Git
- A terminal
- Network access to the services used by the application

The project dependencies are listed in [`requirements.txt`](requirements.txt). Pillow is included as an unpinned dependency and may need a platform-native package on Termux.

## ◈ Installation

### Android / Termux

Update Termux and install the base packages:

```sh
pkg update && pkg upgrade -y
pkg install -y git python python-pillow
```

Clone this fork:

```sh
git clone https://github.com/nexterade/hikari-pulse.git
cd hikari-pulse
```

Run the setup script:

```sh
sh setup.sh
```

Start the CLI:

```sh
python3 main.py
```

### Linux

Install Python 3.10+ and pip using your distribution's package manager, then:

```sh
git clone https://github.com/nexterade/hikari-pulse.git
cd hikari-pulse
cp .env.template .env
sh setup.sh
python3 main.py
```

### OpenWrt

Connect to the router using SSH, install Git if needed, then clone the repository:

```sh
git clone https://github.com/nexterade/hikari-pulse.git
cd hikari-pulse
cp .env.template .env
sh setup.sh
python3 main.py
```

`setup.sh` attempts to install Python 3 and pip if they are missing and detects `opkg` or `apk`. Package names and Python availability vary by OpenWrt release and target architecture.

## ◈ First launch and local data

On first launch, use the account menu and follow the prompts to request and submit an SMS OTP.

The application stores account data locally in the working directory:

- `refresh-tokens.json` — saved refresh tokens and account metadata.
- `active.number` — selected account number.
- `.env` — local environment configuration, including Telegram bot settings when used.

Treat these files as private. Do not upload them to GitHub, send them to other people, or include them in bug reports. Never share OTPs, access tokens, refresh tokens, bot tokens, or personal identity information.

Create the environment file from the template when configuring the optional Telegram bot:

```sh
cp .env.template .env
```

## ◈ Telegram bot setup

1. Create a bot with [@BotFather](https://t.me/BotFather).
2. Obtain your numeric Telegram user ID from a trusted method.
3. Open `.env` and set:

   ```dotenv
   TELEGRAM_BOT_TOKEN="your_bot_token"
   TELEGRAM_ALLOWED_USER_ID="your_numeric_user_id"
   ```

4. Start the bot:

   ```sh
   python3 telegram_bot.py
   ```

The bot only processes messages and callback queries from the configured user in a private chat. Keep the bot token secret. The bot and CLI share `refresh-tokens.json`; do not run them simultaneously.

## ◈ Troubleshooting

### 1. Pillow fails to build — missing JPEG headers

**Typical error:**

```text
The headers or library files could not be found for jpeg,
a required dependency when compiling Pillow from source.
RequiredDependencyException: jpeg
```

**Why it happens:** pip cannot find a compatible prebuilt Pillow wheel for the current Termux/Python environment, so it attempts to compile Pillow from source. The build then fails because the native JPEG development files are unavailable.

**Recommended fix for Termux:**

Install Pillow from the Termux package repository instead of building it with pip:

```sh
pkg update
pkg install -y python-pillow
```

Verify that Python can import it:

```sh
python -c "from PIL import Image; print('Pillow OK:', Image.__version__)"
```

Then install the project requirements:

```sh
python -m pip install --retries 10 --timeout 120 -r requirements.txt
```

If the import test succeeds but pip still tries to build Pillow, check that `python` and `pip` refer to the same Termux Python installation:

```sh
which python
python -m pip --version
python -m pip show pillow
```

### 2. Build Pillow from source (alternative)

Use this only if the Termux `python-pillow` package is unavailable or unsuitable:

```sh
pkg update
pkg install -y clang make pkg-config \
  libjpeg-turbo libpng freetype \
  littlecms libwebp openjpeg
```

Then try:

```sh
python -m pip install --no-build-isolation --no-cache-dir pillow
```

After Pillow installs successfully, retry the project requirements:

```sh
python -m pip install --retries 10 --timeout 120 -r requirements.txt
```

Native library names and availability can vary by repository state and Termux version.

### 3. PyPI `ReadTimeoutError`

**Typical error:**

```text
ReadTimeoutError: HTTPSConnectionPool(host='pypi.org', port=443)
```

This indicates a package-index connection timeout. It is separate from a Pillow compilation error.

Increase pip's timeout and retry count:

```sh
python -m pip config set global.timeout 120
python -m pip config set global.retries 10
```

Retry installation:

```sh
python -m pip install -r requirements.txt
```

If the timeout continues, check the network connection and try again later.

### 4. `python` or `pip` command not found

On Termux:

```sh
pkg update
pkg install -y python
```

Use `python -m pip` rather than a standalone `pip` command to ensure pip belongs to the selected Python interpreter:

```sh
python --version
python -m pip --version
```

### 5. Setup reports an unsupported Python version

The setup script requires Python 3.10 or newer:

```sh
python3 --version
```

If multiple Python installations exist, check which executable is being used:

```sh
command -v python
command -v python3
python3 -m pip --version
```

The minimum-version check is not a guarantee that all third-party packages support every newer Python release or platform. Avoid changing or downgrading Python until the actual dependency error has been identified.

### 6. Telegram bot exits immediately

Check that both required values exist in `.env` and that the numeric user ID contains digits only:

```dotenv
TELEGRAM_BOT_TOKEN="..."
TELEGRAM_ALLOWED_USER_ID="123456789"
```

Run the bot from the repository directory:

```sh
python3 telegram_bot.py
```

If Telegram connectivity fails, verify the bot token and network connection.

## ◈ Updating

From the repository directory:

```sh
git pull
python -m pip install -r requirements.txt
```

Back up `refresh-tokens.json`, `active.number`, and `.env` before updating. Review upstream changes before replacing local modifications.

## ◈ Security and responsible use

- This is an unofficial third-party project and is not affiliated with or endorsed by XL Axiata / MyXL.
- Use only accounts you own or are explicitly authorized to manage.
- Follow applicable laws and the service provider's terms.
- Protect account tokens, OTPs, Telegram bot credentials, and personal information.
- Review code before running it, especially when using credentials or payment-related features.
- Purchase and payment operations may have real consequences. Verify package details and payment method before confirming.

## ◈ Credits

- Original project: [taamarin/cli](https://github.com/taamarin/cli)
- HIKARI ./PULSE fork: [nexterade/hikari-pulse](https://github.com/nexterade/hikari-pulse)

## ◈ Disclaimer

HIKARI ./PULSE is provided as-is, without warranty. APIs, package availability, and service behavior may change without notice. The maintainers are not responsible for account restrictions, failed transactions, unintended purchases, or losses resulting from use of this software.

---

<div align="center">

**HIKARI ./PULSE** · *Illuminate your terminal.*

</div>
