# Tempus mobile test
An automated smoke suite for Tempus, an open-source Android music client for Navidrome, written with Maestro

## Why Tempus
- Tempus is an actively developed app, has real users and a public issue tracker
- It is connected to Navidrome, a music server I already test (webUI with Playwright https://github.com/gperdikas/navidrome_test_automation). Same backend with a new layer, the mobile client- Open-source, so anyone can intall the same APK and run this suit

## Status
- v1 (done): The full smoke test set. 9 cases designed (Jira + AIO tests), run manualy first (virtual device on Android Studio), then automated
- v2 (planned): Appium + WebdriverIO (TypeScript, Page Object Model), API checks against Navidrome, GitHub Actions CI with Allure report

## What is tested
The smoke suite covers the core user journeys of Tempus:
- Search: Search suggestions
- Library and favorites: Marking and unmarking a song as a favorite
- Playback: Starting playback from a playlist and from Shuffle all
- Player controls: Play, pause and next, in both mini player and full player
- Authentication: Login and logout with a saved server, user data kept across sessions

Each flow automates one case of the manual Smoke set and is named after what it checks.

Project structure
`smoke_test/` -> test flows, run together as one suite
`helper_flows/` -> shared setup steps called by the flows (not running as tests)

## Test design principles
- Independent flows: Each flow sets up its own starting state through conditional helpers. No flow depends on execution order or on teardown.
- Pre-action assertions: The starting state is asserted right before the action under test, so a skipped setup step can't produce a false pass.
- Negative validation: Every flow was verified to fail when its key action is removed.
- Selector strategy: UI controls are located by resource id. Data-driven list items (songs, playlists, servers) are located by text. Text is used for controls only when no stable id exists.
- State-based playback checks: Playback is verified through the state of the play/pause control, not only through the presence of a player.
- One flow per test case: Each flow asserts only what its manual test case defines.

## How tests are built
- Every flow starts clean on it own. Helpers put the app in the right state at the start. Flows do not count on clean-up at the end, because steps after a possible failure never run.
- A check before the action. Helpers skip silently when there is nothing to do. SO each flow checks the starting state itself right before the action. A helper that did nothing can't make a test pass by accident
- Every flow was failed on purpose. The main action was commented out and the flow had to go red

## Bugs found
Found while designing and running the manual cases, then reported to Tempus with logs and recordings
#1307 App crashes on roattion when the add/edit radio or add podcast dialog is open
#1310 Add/edit radio dialog can't scroll in landscape, last field cut off
#1313 Full-page Settings screens lose their title on rotation or theme change
#1314 Theme dropdown doesn;t close after choosing a theme

## Limitations
- Smoke only: 9 core cases. Most of the app is not covered
- One device: Android emulator, Pixel 10, Android 17 (API 37). No real phones, other screen sizes or Android versions
- English UI: A few elements have no useful id, so they are found by their text. In another language those flows fail
- No first-time login: Suite  starts from a Tempus already having a server. Logging from first install is not covered
- Manual test data: Navidrome, user, songs, playlists are set up by hand
- No CI yet

## Getting started
### Prerequisites
1. Java 17 or newer. Tested with 25
2. Maestro CLI 2.12.1
3. Android Studio + Android Emulator: Emulator 37.2.12, Platform-Tools 37.0.1
4. Emulator: Pixel 10, API 37 (Android 17), Google APIs, x86_64
5. Tempus APK: v4.31.0, app-tempus-release.apk 
6. Docker to run Navidrome

### Installation
1. Maestro: follow the Maestro install guide. On Windows: download the zip from Maestro's GitHub releases, unzip it (e.g. to C:\maestro), add its bin folder to your user PATH, then open a new terminal and check with `maestro --version`.
2. Emulator: in Android Studio → Device Manager, create the Pixel 10 / API 37 device above and start it. Check it's connected: `adb devices` must show `emulator-5554  device`.
3. Tempus: drag the APK onto the emulator window.
4. Navidrome: run it in Docker on port `4533` (see the Navidrome install guide).

### Test data and app state
The flows expect this setup:
Navidrome:
- A normal user `tempusUser1`
- Songs with unique titles. One called `global1`
- A playlist called `playlist_tempus` for `tempusUser1`
Tempus:
- A server named tempusUser1_server, with URL `http://10.0.2.2:4533`, user `tempusUser1`, low security off
- Log in once with that server. After that, the flows handle logging in and out themselves

### Usage
Run the whole suite: `maestro test smoke_test`
Run one flow: `maestro test smoke_test/file_name` eg: `maestro test smoke_test/search.yaml`

If a flow fails, Maestro saves logs and screenshot in `~/.maestro/tests/<timestamp>`.

## License
MIT