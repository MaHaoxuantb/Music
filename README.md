<p align="center">
  <a href="https://music.qier222.com" target="blank">
    <img src="images/logo.png" alt="Logo" width="156" height="156">
  </a>
  <h2 align="center" style="font-weight: 600">YesPlayMusic</h2>

  <p align="center">
    A Netease Music Player
    <br />
    Refined version by ThomasB
    <br />
    <a href="https://music.qier222.com" target="blank"><strong>🌎 DEMO</strong></a>&nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="#%EF%B8%8F-安装" target="blank"><strong>📦️ Download</strong></a>&nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://t.me/yesplaymusic" target="blank"><strong>💬 Group</strong></a>
    <br />
    <br />
  </p>
</p>

[![Library][library-screenshot]](https://music.qier222.com)

## What's different

This version focus on more fancy visual effects and better UI. Also extra functions could be added.

This version of YesPlayMusic is less likely to be pushed to the main respository due to some reasons.

## ✨ Features

- ✅ Built with the full Vue.js ecosystem
- 🔴 Netease Cloud (NetEase) account login (scan QR / phone / email login)
- 📺 MV (music video) playback support
- 📃 Lyrics display
- 📻 Supports Private FM / Daily recommended tracks
- 🚫🤝 No social features
- 🌎️ Overseas users can play music directly (requires a Netease account)
- 🔐 Supports [UnblockNeteaseMusic](https://github.com/UnblockNeteaseMusic/server#音源清单) and will automatically use alternative audio sources to replace unavailable tracks (not supported on the web version)
  - “Alternative sources” refers to the sources enabled by default.
  - YouTube source requires `yt-dlp` to be installed separately.
- ~~✔️ Automatic daily sign-in (mobile + desktop simultaneous sign-in)~~
- 🌚 Automatic Light/Dark Mode switching
- 👆 Touch Bar support
- 🖥️ PWA support — installable in Chrome/Edge via the address bar ➕
- 🟥 Last.fm scrobbling support
- ☁️ Music cloud storage support
- ⌨️ Custom and global keyboard shortcuts
- 🎧 MPRIS support (Linux media player integration)
- 🛠 More features in development

## 📦️ Installation

The Electron builds are adapted and maintained by [@hawtim](https://github.com/hawtim) and [@qier222](https://github.com/qier222), and support macOS, Windows, and Linux.

Download installers from the project's [Releases](https://github.com/qier222/YesPlayMusic/releases) page.

- macOS users can install via Homebrew: `brew install --cask yesplaymusic`

- Windows users can install via Scoop: `scoop install extras/yesplaymusic`

## Similar projects (unordered)

Contributions and PRs to list more projects are welcome!

- [algerkong/AlgerMusicPlayer](https://github.com/algerkong/AlgerMusicPlayer)
- [asxez/MusicBox](https://github.com/asxez/MusicBox)
- [lianchengwu/wmplayer](https://github.com/lianchengwu/wmplayer)

## ⚙️ Deploy to Vercel

[IMPORTANT] This is for the mother resp, YesplayMusic

Besides downloading and running the desktop app, you can deploy this project to Vercel or your own server. The project's demo (https://music.qier222.com) is deployed on Vercel.

1. Deploy the Netease API — see [Binaryify/NeteaseCloudMusicApi](https://neteasecloudmusicapi.vercel.app/#/?id=%e5%ae%89%e8%a3%85). You can also deploy that API to Vercel.

2. Fork this repository into your GitHub account.

3. In your fork, add a `vercel.json` file and replace `https://your-netease-api.example.com` with your deployed Netease API URL:

```json
{
  "rewrites": [
    {
      "source": "/api/:match*",
      "destination": "https://your-netease-api.example.com/:match*"
    }
  ]
}
```

4. Open https://vercel.com and sign in with GitHub.

5. Import your forked repository into Vercel.

6. Under Project Settings > Environment Variables, add `VUE_APP_NETEASE_API_URL` with the value `/api` and click Add. Then click Deploy.

## ⚙️ Deploy to your own server

[IMPORTANT] This is for the mother resp, YesplayMusic

You can also host the web build on your own server.

1. Deploy the Netease API — see [Binaryify/NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)
2. Clone this repository:

```sh
git clone --recursive https://github.com/qier222/YesPlayMusic.git
```

3. Install dependencies:

```sh
yarn install
```

4. (Optional) Use Nginx to reverse-proxy the API to `/api`. If the API and the web app are served from different domains (cross-origin), some issues may occur without a proxy.

5. Copy `/.env.example` to `/.env` and set `VUE_APP_NETEASE_API_URL` to your Netease API address. For local development, you can set the API address to `http://localhost:3000` and YesPlayMusic to `http://localhost:8080`. If you used a proxy you can set the API address to `/api`.

```
VUE_APP_NETEASE_API_URL=http://localhost:3000
```

6. Build the project:

```sh
yarn run build
```

7. Upload the files in the `/dist` directory to your web server.

## ⚙️ Deploy via BT Panel (BaoTa) Docker App Store

[IMPORTANT] This is for the mother resp, YesplayMusic

1. Install BaoTa (BT Panel). Visit the official site: https://www.bt.cn/new/download.html and follow the installation script for the stable edition.

2. Log in to the panel and click Docker in the left sidebar. If Docker is not installed, the panel will prompt you to install it — follow the prompts to install.

3. In the App Store, find YesPlayMusic and install it. Configure domain, ports, and other basic settings.

4. After installation, visit the domain you configured to access the app.

## ⚙️ Docker deployment

[IMPORTANT] This is for the mother resp, YesplayMusic

1. Build the Docker image:

```sh
docker build -t yesplaymusic .
```

2. Run a container:

```sh
docker run -d --name YesPlayMusic -p 80:80 yesplaymusic
```

3. Or using Docker Compose:

```sh
docker-compose up -d
```

Then open: `http://localhost`

## 👷‍♂️ Build desktop clients (packaging)

If you can't find an installer for your device on the Releases page, you can build your own client.

1. Packaging Electron requires Node.js and Yarn. Download Node.js from https://nodejs.org. After installing Node.js, install Yarn with `npm install -g yarn`.

2. Clone the repository:

```sh
git clone --recursive https://github.com/qier222/YesPlayMusic.git
```

3. Install dependencies:

```sh
yarn install
```

4. Copy `/.env.example` to `/.env`.

5. Choose one of the commands below to build an installer for your target platform. The output is saved under `/dist_electron`. See the [electron-builder docs](https://www.electron.build/cli) for details.

| Command                                    | Description               |
| ----------------------------------------- | ------------------------- |
| `yarn electron:build --windows nsis:ia32` | Windows 32-bit            |
| `yarn electron:build --windows nsis:arm64`| Windows ARM               |
| `yarn electron:build --linux deb:armv7l`  | Debian armv7l (e.g. Raspberry Pi) |
| `yarn electron:build --macos dir:arm64`   | macOS ARM                 |

## :computer: Configure development environment

This project uses [NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi) as its API provider.

Run the project locally:

```shell
# Install dependencies
yarn install

# Create local environment variables
cp .env.example .env

# Run (web)
yarn serve

# Run (electron)
yarn electron:serve
```

Run a local NeteaseCloudMusicApi instance, or deploy it to Vercel and point the app to it:

```shell
# Run the API (default port 3000)
yarn netease_api:run
```

## ☑️ Todo

See the project board at [Projects](https://github.com/qier222/YesPlayMusic/projects/1)

Issues and Pull Requests are welcome.

## 📜 License

This project is intended for personal learning and research only. Commercial or illegal use is prohibited.

It is open-sourced under the [MIT license](https://opensource.org/licenses/MIT).

## Inspiration

The API source code comes from [Binaryify/NeteaseCloudMusicApi](https://github.com/Binaryify/NeteaseCloudMusicApi)

- [Apple Music](https://music.apple.com)
- [YouTube Music](https://music.youtube.com)
- [Spotify](https://www.spotify.com)
- [Netease Cloud Music](https://music.163.com)

## 🖼️ Screenshots

![lyrics][lyrics-screenshot]
![library-dark][library-dark-screenshot]
![album][album-screenshot]
![home-2][home-2-screenshot]
![artist][artist-screenshot]
![search][search-screenshot]
![home][home-screenshot]
![explore][explore-screenshot]

<!-- MARKDOWN LINKS & IMAGES -->

[album-screenshot]: images/album.png
[artist-screenshot]: images/artist.png
[explore-screenshot]: images/explore.png
[home-screenshot]: images/home.png
[home-2-screenshot]: images/home-2.png
[lyrics-screenshot]: images/lyrics.png
[library-screenshot]: images/library.png
[library-dark-screenshot]: images/library-dark.png
[search-screenshot]: images/search.png
