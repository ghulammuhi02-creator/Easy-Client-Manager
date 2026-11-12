# AutoDeliver

Automatically deliver finished video exports to client-specific local folders and Google Drive folders.

AutoDeliver watches an export folder. When a video file is stable and follows the naming convention `CLIENTCODE__video-name.mp4`, it looks up that client in `config.yaml`, then copies the video to the configured local destination and/or uploads it to Google Drive.

## Features

- Watch a local export folder for new video files
- Route files using a client code in the filename
- Copy finished videos into client-specific local folders
- Upload finished videos to client-specific Google Drive folders
- Wait for files to finish exporting before delivery
- SQLite delivery history to avoid repeat deliveries
- Logs and a dry-run mode
- Source files are never deleted

## 1. Requirements

- Python 3.10 or newer
- A Google account (only if using Google Drive delivery)
- Google Drive API credentials (only if using Google Drive delivery)

## 2. Install

```bash
# Create and activate a virtual environment
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

## 3. Configure

Copy `config.example.yaml` to `config.yaml` and edit it.

```yaml
watch:
  folder: "D:/VideoExports"
  recursive: false
  stable_seconds: 8
  scan_interval_seconds: 3

clients:
  CLIENTA:
    local_folder: "D:/ClientVideos/ClientA"
    google_drive_folder_id: "YOUR_GOOGLE_DRIVE_FOLDER_ID"
```

Filename examples:

- `CLIENTA__Instagram_Reel_01.mp4`
- `CLIENTB__YouTube_Final.mov`

The part before the first double underscore (`__`) is the client code. Client codes are case-insensitive and must match a key under `clients`.

For local-only delivery, leave `google_drive_folder_id` empty. For Drive-only delivery, leave `local_folder` empty.

## 4. Google Drive setup (optional)

1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Create or select a Google Cloud project.
3. Enable **Google Drive API**.
4. Configure the OAuth consent screen for your account.
5. Create an OAuth client ID with application type **Desktop app**.
6. Download the OAuth client JSON file and save it as `credentials.json` in the AutoDeliver project root.
7. Set `google_drive.enabled: true` in `config.yaml`.

On first Drive delivery, a browser window opens for Google sign-in and permission approval. The app saves `token.json` locally for later sessions. Keep `credentials.json` and `token.json private; they are excluded by `.gitignore`.

Use the Google Drive folder ID in the folder URL. For example, in
`https://drive.google.com/drive/folders/1AbCdEf...`, the folder ID is `1AbCdEf...`.

## 5. Run

```bash
# Preview routing without copying or uploading
python -m autodeliver --config config.yaml --dry-run

# Start watching and delivering
python -m autodeliver --config config.yaml
```

Press `Ctrl+C` to stop.

## 6. Important behavior

- AutoDeliver copies; it does not move or delete your original export.
- Files are only processed after their size and modification time remain unchanged for the configured stable period.
- Unknown client codes are logged and skipped.
- A delivery is recorded in a local SQLite database. If you intentionally need to deliver the same source file again, remove its corresponding history entry from `data/deliveries.db` or export it under a new filename.
- Google Drive uploads use the Drive API and the signed-in Google account's access. The Google Drive API may require additional consent-screen configuration depending on your account and organization.

## Security

Never commit `credentials.json`, `token.json`, `config.yaml` if it contains private folder IDs, logs, database files, or client media. Review `.gitignore` before pushing to GitHub. Do not expose this local watcher directly to the public internet.

## Project status

This repository is an initial local automation MVP. It does not yet include a web dashboard, team accounts, notifications, or cloud-hosted processing.
