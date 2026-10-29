# Torsher for Unraid

Unraid template for [Torsher](https://torsher.app): your home cinema in one container, with a TV showcase, your own library, playback while the file downloads, and series that follow themselves.

The app itself, its issues and releases live in [torsher-app/torsher](https://github.com/torsher-app/torsher). This repository holds only the template.

> [!NOTE]
> The template hasn't been verified on a live Unraid server yet. Report problems with the app in [torsher-app/torsher](https://github.com/torsher-app/torsher/issues) and problems with the template itself (fields, paths, install) in [torsher-app/unraid-templates](https://github.com/torsher-app/unraid-templates/issues).

## Install

Open the **Apps** tab (Community Applications), search for **Torsher** and select **Install**. Check the paths below and select **Apply**.

If Torsher doesn't show up in Apps yet, add the template by hand. Open the Unraid terminal (the `>_` icon in the header) and run:

```bash
curl -fsSL -o /boot/config/plugins/dockerMan/templates-user/my-torsher.xml \
  https://raw.githubusercontent.com/torsher-app/unraid-templates/main/templates/torsher.xml
```

Then open **Docker**, select **Add Container**, and pick **torsher** in the **Template** list.

When the container is running, select **WebUI** in its menu. The panel asks you to create the owner account and starts the setup wizard.

## Paths

| In the container | Default on Unraid | What's inside |
| --- | --- | --- |
| `/data` | `/mnt/user/appdata/torsher` | Database, keys, the log and backups |
| `/storage` | `/mnt/user/data/torsher` | Downloads in `torrents`, the library in `library` |
| `/media` | empty | Optional. Your existing media, read-only |

Downloads and the library share one path on purpose: kept files are hard-linked from downloads into the library and take no extra space. For hard links, make sure **Settings → Global Share Settings → Tunable (support Hard Links)** is set to **Yes**. The setup wizard shows whether hard links work.

Keep `/data` on a pool, not on a share mounted from another machine. A tip for the database: keep the appdata share on a pool only (an exclusive share), or point the path at `/mnt/cache/appdata/torsher`, so the database doesn't go through the user-share file system.

If you already keep media in one `data` share, as in the [TRaSH Guides](https://trash-guides.info/File-and-Folder-Structure/How-to-set-up/Unraid/), the default `/mnt/user/data/torsher` puts Torsher's folders next to it.

## Ports

| Port | What for |
| --- | --- |
| `8080` | The showcase at `/` and the control panel at `/panel/` |
| `51420` TCP and UDP | Optional. Incoming connections from torrent peers |

The peer port must be the same number in both mappings and in the **Peer port** variable (advanced view). A port changed in the panel under **Settings → Torrent** takes precedence over the variable, so after changing it there, set the same number in the container settings too. While the variable is set and differs from the panel, the panel shows a warning; it mentions `docker-compose.yml`, which on Unraid means the container settings. The panel can't see the mappings themselves: changing only the host side of the peer port gives no warning.

If port `8080` is already taken on your server, change only the host side of the mapping. The **WebUI** button follows it.

## GPU transcoding

The template has no GPU device: a default device would stop the container from starting on servers without a GPU, so the template leaves it out.

- **Intel or AMD:** in the Unraid terminal run `ls /dev/dri`. If it's empty or missing, the GPU driver isn't loaded: for Intel, load it, for example with the Intel GPU TOP plugin, and check again. Then select **Add another Path, Port, Variable, Label or Device**, choose **Device**, and enter `/dev/dri`.
- **NVIDIA:** install the Nvidia Driver plugin. In the advanced view add `--runtime=nvidia` to **Extra Parameters**. The image already sets the NVIDIA variables. To give the container only one card, add the variable `NVIDIA_VISIBLE_DEVICES` with the card's UUID from the plugin page. NVENC support in the image hasn't been tested on NVIDIA hardware yet.

Check the result in the panel under **Settings → Playback**. See [Hardware transcoding](https://torsher.app/docs/install/hardware-acceleration).

## Sources

Torsher connects to any Torznab indexer, for example Prowlarr or Jackett. Nothing is included: you choose the sources and are responsible for using them lawfully.

## License

The template is under the [MIT License](LICENSE). Torsher itself is under AGPL-3.0.
