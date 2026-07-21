# Jellyfin Plugin Telegram Notifier

## Fork patch: ItemUpdated notifications / 本 fork 补丁说明

This fork branch adds dedicated Jellyfin media refresh/update notifications. Use branch `notify-item-updated` from `https://github.com/jiemo9527/jellyfin-plugin-TelegramNotifier`.

本 fork 分支新增了独立的媒体刷新/更新通知。请使用 `jiemo9527/jellyfin-plugin-TelegramNotifier` 的 `notify-item-updated` 分支。

This branch is based on upstream `12.3.0.0`, including `ServerDisplayUrl` and HTTP proxy configuration.

本分支已合并上游 `12.3.0.0`，包含 `ServerDisplayUrl / 展示用服务器地址` 和 HTTP 代理配置。

### What is fixed / 已修复功能

- Adds independent `ItemUpdated / 媒体刷新（更新）` notifications.
- Adds separate `ItemUpdated` switches, subtype switches, and templates for Movies, Series, Seasons, Episodes, Albums, Songs, and Books.
- Uses default Chinese update text like `{item.Name} 已刷（更）新`.
- Prevents parent TV refresh spam: refreshing Series only sends Series, refreshing Season only sends Season, and Episode notifications are sent only when the Episode itself is updated.
- Suppresses `ItemUpdated` for the same item if `ItemAdded` was sent successfully within the previous 5 minutes.
- Falls back to a text message if sending a photo fails.
- Normalizes `ServerUrl` so image URLs work whether `http://` or `https://` is included or not.
- Keeps upstream `ServerDisplayUrl` and HTTP proxy support from `12.3.0.0`.
- Makes the configuration UI cleaner by hiding disabled/non-primary notification types by default.
- Adds Chinese labels beside the existing UI text.

### Install this fork build / 安装此 fork 版本

1. Install or update the original `Telegram Notifier 12.3.0.0` plugin in Jellyfin first, then stop Jellyfin.

```bash
docker stop jellyfin
```

2. Build this fork branch with the .NET 9 SDK container.

```bash
git clone -b notify-item-updated https://github.com/jiemo9527/jellyfin-plugin-TelegramNotifier.git
docker run --rm -v "$PWD/jellyfin-plugin-TelegramNotifier:/src" -w /src mcr.microsoft.com/dotnet/sdk:9.0 dotnet build -c Release
```

3. Replace the installed plugin DLL.

```bash
cp jellyfin-plugin-TelegramNotifier/Jellyfin.Plugin.TelegramNotifier/bin/Release/net9.0/Jellyfin.Plugin.TelegramNotifier.dll "/srv/jellyfin/config/plugins/Telegram Notifier_12.3.0.0/Jellyfin.Plugin.TelegramNotifier.dll"
```

4. Start Jellyfin again.

```bash
docker start jellyfin
```

5. Open the Telegram Notifier settings in Jellyfin and enable `Item Updated / 媒体刷新（更新）` plus the needed subtypes.

### Reproduce the original issue / 原版问题复现

1. Install the original `Telegram Notifier 12.2.0.0` or `12.3.0.0` on Jellyfin `10.11.x`.
2. Configure a valid Telegram bot and enable `Item Added` notifications.
3. Add a movie or episode and wait for the Telegram Notifier scheduled task.
4. If the item primary image is not ready or returns `404`, the original plugin may fail the notification because it only tries `sendPhoto`.
5. Refresh metadata for an existing movie, series, season, or episode.
6. The original plugin does not send a refresh/update notification because it only subscribes to `ItemAdded`, not `ItemUpdated`.
7. If add and update events happen close together, a naive update implementation can produce duplicate/noisy notifications; this fork suppresses update notifications for 5 minutes after a successful add for the same item.

<a href='https://ko-fi.com/B0B8112Y0Y' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://storage.ko-fi.com/cdn/kofi1.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>

![GitHub Downloads (all assets, all releases)](https://img.shields.io/github/downloads/RomainPierre7/jellyfin-plugin-TelegramNotifier/total)

![GitHub Downloads (all assets, latest release)](https://img.shields.io/github/downloads/RomainPierre7/jellyfin-plugin-TelegramNotifier/latest/total)

This [Jellyfin](https://github.com/jellyfin) plugin provides notification functionalities via Telegram for various events occurring within your Jellyfin server. Stay informed about playback activities, user management, plugin operations, and more right from your Telegram account. You can personalize the configuration for each user.

<p align="center">
<img src="assets/logo.png" alt="Logo" width="300" height="300">

🧰 A troubleshooting guide is available [here](https://github.com/RomainPierre7/jellyfin-plugin-TelegramNotifier/blob/main/docs/Troubleshooting.md).

> [!IMPORTANT]
> If you encounter a problem, a bug or have an idea for a new feature, submit an issue [here](https://github.com/RomainPierre7/jellyfin-plugin-TelegramNotifier/issues). Please, submit only one feature or bug per issue.

> [!TIP]
> If you're finding value in this project and it's been helpful to you, consider giving it a star ⭐️ on GitHub ! Your support means a lot and helps others discover the project too.

## Table of contents

- [Install the plugin](#install-the-plugin)
- [Use the plugin](#use-the-plugin)
- [Install the project (for developers)](#install-the-project-for-developers)

## Install the plugin

To install the plugin on your Jellyfin server, you need to follow these steps:

1. Go to the Jellyfin dashboard
2. Go to the **Plugins** section
3. Click on the **Manage Repositories** button
4. Click on **New Repository**
5. Add the depot URL: 

Repository Name: ```Telegram Notifier```

Repository URL:
```
https://raw.githubusercontent.com/RomainPierre7/jellyfin-plugin-TelegramNotifier/main/manifest.json
```

6. Go back to the **plugins** page
7. Search for the plugin: ```Telegram Notifier```

> Make sure to enable the **All** or **Available** filter

<img src="assets/banner.png" alt="Logo" width="267" height="150">

8. Click on the **Install** button
9. Restart the Jellyfin server

> **Note:** You can also install the plugin manually by downloading the latest release from the [releases page](https://github.com/RomainPierre7/jellyfin-plugin-TelegramNotifier/releases).

## Use the plugin

To use the plugin, you need to follow these steps:

1. Go to the Jellyfin dashboard
2. Go to the **Plugins** section
3. Click on the **Telegram Notifier** plugin
4. Click on **settings**
5. Configure the plugin

### Configuration

1. Select the user you want to configure
2. Enter the Telegram bot token
> **Note:** You can create a Telegram bot by sending the command ```/newbot``` to the [BotFather](https://t.me/botfather) and collect the token.
3. Enter the chat ID
> **Note:** You can get the collect the chat ID by sending a message to the bot and then go to the URL: ```https://api.telegram.org/bot<YourBOTToken>/getUpdates```
4. Click on the **Test** button to test the configuration
5. Enable the notifications for the user
6. Select the events you want to be notified about

Available events:
- Item added
- Item deleted
- Playback start
- Playback progress
- Playback stop
- Subtitle download failure *(soon...)*
- Authentication failure *(soon...)*
- Authentication success *(soon...)*
- Session start
- Pending restart *(soon...)*
- Task completed *(soon...)*
- Plugin installation cancelled
- Plugin installation failed
- Plugin installed
- Plugin installing
- Plugin uninstalled
- Plugin updated
- User created
- User deleted
- User locked out
- User password changed
- User updated *(soon...)*
- User data saved *(soon...)*

7. Click on the **Save** button

**Example of the configuration page:**

![Configuration page exemple](assets/config.png)

## Install the project (for developers)

To install the project, you need to follow these steps:

> **Note:** You need to have the [.NET 9.0 SDK](https://dotnet.microsoft.com/download) installed on your machine.

1. Clone the repository
2. Install the dependencies
3. Compile the plugin
4. Install the plugin

### 1. Clone the repository

```bash
git clone https://github.com/RomainPierre7/jellyfin_telegram_notifier.git
```

### 2. Install the dependencies

```bash
dotnet add package Jellyfin.Controller
```

### 3. Compile the plugin

```bash
dotnet build
```

or use the **Makefile**:

This command will only build the dll file in the ```plugin``` directory.

```bash
make dev
```

Other commands are available in the **Makefile**:

- ```make build``` - Build the project
- ```make publish``` - Publish the project
- ```make clean``` - Clean the project
- ```make dev``` - Build the project and copy the `dll` file in the ```plugin``` directory
- ```make plugin``` - Publish the project and copy the `dll` file in the ```plugin``` directory

### 4. Install the plugin

To install the plugin, you have to find the Jellyfin plugin directory. It depends on your installation but the most common paths are:

- **Linux**: ```/var/lib/jellyfin/plugins```
- **Windows**: ```C:\ProgramData\Jellyfin\Server\plugins```
- **Docker**: ```/config/plugins``` or ```/config/data/plugins```

Then, you have to copy the ```Jellyfin.Plugin.TelegramNotifier.dll``` file in a folder in the plugin directory.

Example:

```
Plugin
│
└── TelegramNotifier
    │
    └── Jellyfin.Plugin.TelegramNotifier.dll
```

Finally, you have to restart the Jellyfin server.

### 5. Tips

If you want to easily test the plugin, you can use the following command to download a video file directly on your server:

```bash
curl https://cdn.pixabay.com/video/2025/03/28/268290_large.mp4 --output titanic.mp4
```
