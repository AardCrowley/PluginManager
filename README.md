# Plugin Manager

One MUSHclient plugin (`PluginManager.xml`, name `Plugin_Manager`) to install, update, reload, enable, disable, remove and roll back plugins, from commands or a themed window.
Crowley's plugins come from a **manifest** (a JSON file). Players can add **repositories** (GitHub repos, other people's manifests, single plugin files): what the manager finds in them is listed in the **Store** tab. Other people's packages can also be listed in the manifest as links.

Window tabs: **Crowley** (the plugins in the main manifest, plus any plugin whose author is Crowley) / **Stock** (the AardMUSHclient package's own) / **Other** (everything else that is loaded, and the manifest's link-only entries) / **Store** (offered by repositories you added) / **All**.
**Check** reads every source; a row with a newer version then gets an **Update** button (and **Update all** appears). The list has Plugin / Version / Author columns. Clicking a row shows its details at the bottom: name, version, author, purpose (always), file or source, and buttons for the repository and, when the plugin has one, its saved-state file (opened with the edit action your system has for `.xml` files, else the open action). Buttons per row: Update, On/Off, Reload, Remove (Install for plugins that are offered but not loaded). **Add** adds a plugin file you downloaded yourself.
The title bar menu has Font..., and `pm font NAME SIZE` sets the list font.

![Screenshot 1](https://zipline.arcylix.net/u/NSBzLT.png)
![Screenshot 2](https://zipline.arcylix.net/u/LE1zcW.png)
![Screenshot 3](https://zipline.arcylix.net/u/mMKU8C.png)

## Commands (`pm help` shows them in the client)
| | |
|---|---|
| `pm` | open or close the window |
| `pm list [crowley\|stock\|other\|store\|links\|updates\|all]` | text list with clickable `[reload] [disable] [update] [remove] [repo]` links |
| `pm check` | read the manifest and the repositories, mark plugins that have a newer version |
| `pm update NAME\|NUMBER\|all` | download, check, swap in (old file kept as `name.xml.bak_YYYYMMDD`), reload |
| `pm install NAME\|NUMBER` | download a plugin from the manifest that is not loaded, put it in `pm dir`, load it |
| `pm reload / unload / enable / disable NAME\|NUMBER` | the plain actions |
| `pm remove NAME\|NUMBER` | uninstall the plugin from MUSHclient and nothing else: its file and its saved data are never deleted or renamed (`pm load FILE` / `pm add` bring it back) |
| `pm rollback NAME\|NUMBER` | put back the files the last manager update replaced |
| `pm aliases / triggers NAME\|NUMBER` | a window (same look as the manager, with tabs for Aliases and Triggers, a scroll bar and **Copy**; each entry shows one line of what it sends, cut to the width; clicking an entry opens a side panel to the right with the whole entry (wrapped), **Copy entry** and an x; clicking the same entry again closes it) listing what the loaded plugin defines: match text, what it sends, script, group, on/off, regular expression (also the **Aliases** / **Triggers** buttons in the details) |
| `pm info / open / state NAME\|NUMBER` | details (always with the purpose and author) / open the repository link in the browser / open the plugin's saved-state file in your editor |
| `pm add [FILE]` | add a plugin file you downloaded: checked, copied into the plugins folder (with the `.lua` files it names), loaded. No FILE opens a file picker |
| `pm load FILE` | load a plugin file where it is (a path, or a name inside the plugins folder) |
| `pm source` / `pm source add URL` / `pm source remove N` | the repositories: a GitHub repository, a manifest (`.json`) or one plugin file (`.xml`) |
| `pm font [NAME [SIZE]\|reset]` | the list font (saved in the plugin's state) |
| `pm save` | save the world file (`Save()`) so what is loaded now is loaded next time (also File > Save world) |
| `pm manifest [URL\|reset]`, `pm dir [FOLDER]`, `pm auto on\|off` | settings |

NAME is part of a name; NUMBER is the number in the last `pm list` (or the open window tab).

## Repositories (the Store tab)
**A GitHub user or organization** (`pm source add https://github.com/AardPlugins`, or `...?tab=repositories`, or Repos > Add a repository): the manager lists its public repositories (GitHub's API, 100 per request, up to 500) in a checklist window with check boxes, dates, descriptions and tags for forks, archived and empty repositories.
- **Add selected** adds the checked repositories as sources (asks first) and reads them.
- **Look for plugins in all** (or `pm source scan URL` from the command line) looks through every repository except forks, archived and empty ones, and adds only the repositories that contain MUSHclient plugins as sources. It asks first. GitHub allows about 60 API requests an hour without an account and each repository needs one, so at most 50 are looked at per run (repositories that are already listed are skipped, so running it again goes on with the rest); if GitHub says "too many requests" the run stops, keeps what it found, and says so.
- Nothing is installed by either; the plugins show up on the Store tab like any other repository's.

**Store tab layout:** two levels of collapsing. One header per **owner** (the GitHub user or organization; for other addresses, the host): only the picked owner is open. Inside an owner that has several repositories there is one header per repository, and only the picked one is open; an owner with a single source shows its plugins straight under its name. Clicking a header opens it and closes the others on its level (clicking the open one closes it). The choices are kept in the plugin's state (`store_owner`, `store_open`). Every owner and repository header has a **Remove repo** (**Remove repos** for an owner with several) button that stops listing it (all of an owner's repositories, or just that one) after a question; plugins already installed from it stay installed. The same by command: `pm source remove N|URL|OWNER`; the Repos menu lists "Remove all of OWNER (n)".

**Force install** (button in the details, or `pm force NAME`): for a plugin a repository offers that looks like one you have installed (same plugin id, same file name, or a very alike name), for example a fork. It asks first, downloads and checks the repository's files, copies/moves the current plugin file to `name.xml.bak_DATE` (same file name: the update machinery moves it aside; other file name: a copy is made and the old file stays), takes the current plugin out of MUSHclient, and installs the repository's version. If the new one does not load, the old plugin is put back. The manager itself is never replaced this way.

`pm source add https://github.com/OWNER/REPO` (or Store tab > Repos > Add a repository). Accepted addresses: a GitHub repository (optionally `.../tree/BRANCH/FOLDER`), a link to a file in one (`.../blob/...xml`), a manifest `.json`, or a plugin `.xml`.
- A repository with a `manifest.json` in its root (same format as below) is read as a manifest. Without one the manager lists the repository's files (GitHub's API, one request), looks at up to 40 `.xml` files (under 1.5 MB, not in hidden folders), keeps those that are valid MUSHclient plugins (plugin tag with a 24-character id, name and numeric version, complete, script compiles), and notes each one's checksum. A `.lua` file next to a plugin is taken along when the plugin's text names it. If a plugin id turns up twice, the higher version is listed.
- At install time the file is downloaded again and must match the checksum from the scan; if the repository changed in between, nothing is installed and `pm check` rescans.
- Sources are only added by the player (command, or a confirmation in the window). Installing or updating from one asks for confirmation in the window and names the source. Everything the manager lists from a source is that source's own content: the checksum guards against a changed file between the scan and the install, not against a source that is untrustworthy.
- A plugin the main manifest offers is always listed under Crowley, even if a repository lists it too.

## Files a plugin needs (dependencies)
Many plugins come with files of their own (Lua modules, data, sounds), often in sub-folders;the Consider window has two `.lua` files. When the manager scans a repository it gives each plugin **everything in the plugin's own folder and the folders below it** except:
- repository files: README*, LICENSE*, COPYING, NOTICE, CHANGELOG/CREDIT/AUTHORS (as `.md`, `.txt` or no extension; a `changelog.lua` is code and is kept), anything `.md`, `git.txt`, and hidden files and folders (`.gitignore`, `.github/`);
- programs and other plugin files: `.exe .dll .bat .cmd .com .scr .ps1 .vbs .js .msi .lnk .reg .jar ...`, and `.xml` / `.mcl` files;
- the folders of other plugins found in the same repository (each plugin owns its own folder).
Limits: 80 extra files per plugin, 1.5 MB per file, 4 folder levels, plain file names (no Windows device names like `nul` or `aux`). If one of the plugin's files cannot be downloaded or installed (too large, an unusable name, more than 80 files), that plugin is not offered at all, and `pm source` says which one and why: it would install broken. Programs next to it (`.exe`, `.bat` ...) are simply ignored.
A plugin that has extra files is installed into a **folder of its own** named after the plugin file (`plugins\Mushclient-Consider\Aardwolf_Consider_Miniwin.xml` with its `lua` files beside it), so its files cannot collide with other plugins' files. The folders are created with `lfs` when the client has it, else with a hidden `mkdir`. Updates go into the folder the plugin is already in, replacing changed files (the old ones are kept as `.bak_DATE`) and adding new ones. A new install never overwrites a file that is already there, and an update of a plugin that sits in the shared plugins folder never replaces a file of the same name that the manager did not put there for that plugin (it could be another plugin's module); a plugin in a folder of its own may replace its own files.
Manifest entries can say the same: `"extras": [ {"file": "Aardwolf_Consider_Miniwin.lua", "url": "...", "sha256": "..."} ]` (relative paths with `/`) and `"dir": "FolderName"`.

## Safety
- Nothing is downloaded or changed without a command or a click. `pm auto on` (default) only **checks** the manifest about once a day and says if updates exist.
- Addresses must be `https://` (the manifest and every download). The manifest must list a `sha256` for every file; a missing or wrong checksum means nothing is installed.
- A download is also checked to be a complete plugin file (closing tag present), with the same plugin id and the same version as the manifest entry, and with a script that compiles. File names from the manifest must be plain names (letters, digits, `_ - .` and spaces); extras cannot be `.xml`/`.mcl` files.
- Files are written next to their target first, then swapped in; the old file stays as `name.xml.bak_YYYYMMDD`. If a later file of the same update cannot be swapped in, the earlier ones are put back. If a reloaded plugin fails to load, the previous file is put back and loaded again. A plugin you had disabled stays disabled after a reload or update.
- The manager does not unload, disable or remove itself. `pm update` / `pm reload` / `pm rollback` of the manager reload it through the world's script engine (the way the Aardwolf plugins reload themselves), and `pm update all` does the manager last.
- While an update or check runs, other changing commands wait.
- `pm open` opens only `http(s)` links.
- Remove only takes the plugin out of MUSHclient (after a question in the window, which mentions it when the plugin is one of the AardMUSHclient package's). No file or saved data is deleted or renamed. Updates keep the replaced file as `name.xml.bak_YYYYMMDD`.
- `pm add` refuses files that are not plugins (or whose script does not compile), file names that are not plain, and a plugin whose id is already loaded; copies are taken back out if the plugin will not load.
