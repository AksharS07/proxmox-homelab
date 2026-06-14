# Home Media Server: Configuration & Automation Log

This document serves as the official documentation for the custom Media Server automation pipeline built on CasaOS. If anything ever breaks or needs to be re-installed, use this reference guide.

## 1. Directory Structure
Fixed massive duplicate file issues by completely restructuring the root folders in CasaOS to ensure Sonarr can perform atomic moves instead of copying files.

**The Golden Paths:**
- **Movies:** `/DATA/Media/Movies`
- **TV Shows:** `/DATA/Media/TV Shows`
- **Anime:** `/DATA/Media/Anime`
- **Downloads:** `/DATA/Downloads`

> **WARNING:** Never mix Downloads and Media folders! Sonarr relies on moving completed files from the `/DATA/Downloads` folder into the clean `/DATA/Media` folders to properly organize and rename them.

## 2. Network & AdGuard Home
Installed **AdGuard Home** to act as a network-wide ad-blocker and DNS server.
- **Port Conflict Resolved:** Disabled `systemd-resolved` on port `53` to allow AdGuard Home to listen for DNS requests.
- **DNS Bypass:** Configured the CasaOS host machine to bypass its own AdGuard DNS (using Google DNS `8.8.8.8` instead) so that internal server apps can still download updates and fetch metadata even if AdGuard is blocking trackers.

## 3. The Automation Stack (The 'Arrs)
Fully automated the media pipeline using the standard Arr stack.

### Prowlarr (The Indexer Manager)
- Serves as the master list of all torrent sites.
- **Nyaa (Anime Tracker):** Bypassed ISP blocks by using a proxy URL (`https://nyaa.iss.ink`).
- **AnimeTosho:** Added as a powerful, uncensored backup indexer that mirrors Nyaa without the strict bot-spam limits.

### Sonarr (The TV/Anime Manager)
- Connected to Prowlarr and qBittorrent.
- **Series Types:** Anime shows must be set to the `Anime` series type to utilize the Nyaa indexer.
- **Renaming:** Global episode renaming enabled. If files get stuck with ugly torrent names (e.g., `[Erai-raws]`), use the **Preview Rename** button to fix them.
- **The Anime Naming Curse:** If Sonarr gets confused by Season/Cour numbers, use **Wanted -> Manual Import** from the Downloads folder to forcefully tell Sonarr exactly which Season/Episode it is.

### Jellyseerr (The Request App)
- Connected to Sonarr and Jellyfin. Allows seamless requests for new shows/movies from a Netflix-style interface, automatically triggering Sonarr downloads.

## 4. Jellyfin (The Media Player)
Jellyfin is the core playback engine, configured to read directly from the `/DATA/Media` folders.

### Metadata & Databases
- **Anime Metadata:** 'The Open Movie Database' (OMDb) is terrible for Anime. Use the **Identify** tool (3 dots on the main show poster) to forcefully bind Anime shows to **TheMovieDb (TMDB)** or **TheTVDB**.
- **Missing Posters:** Manually upload images downloaded from Google directly into the Jellyfin UI if TMDB is missing season posters.

### Custom Plugins: Intro Skipper
Installed the **Jellyfin Intro Skipper** plugin to bring Netflix-style "Skip Intro" buttons to anime.
- **Repository:** `https://intro-skipper.org/manifest.json`
- **Configuration:** Set to automatically analyze new media for Introductions and Credits.
- **Web UI:** Relies on the **File Transformation Plugin** to inject the physical "Skip" button into the web browser player.
