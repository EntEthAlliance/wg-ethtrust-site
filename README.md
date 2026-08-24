# EthTrust Security Levels WG — group page

Source for the working-group page published at
**https://entethalliance.org/groups/EthTrust/** (WordPress/WP Engine host).

- `docs/` — the static site, built on the
  [EEA design system](https://github.com/EntEthAlliance/eea-design-system)
  **editorial family** (`editorial.css` loaded from the design system's Pages
  URL; canonical `.eea-site-bar` / `.eea-colophon` markup; all page-local CSS
  on `--eea-ed-*` tokens).
- GitHub Pages serves `docs/` as the **staging preview**:
  https://entethalliance.github.io/wg-ethtrust-site/
- The live copy on WordPress is deployed by uploading the contents of `docs/`
  to `/groups/EthTrust/` on the WP Engine host — see *Deploying* below.

History: the page's previous home was `wg-eta-registry/docs/` (now archived);
its GH Pages `index.html` has redirected to the WordPress copy since Feb 2025,
and the WordPress copy itself was never under version control until this repo.

## Deploying to entethalliance.org

1. Get the change merged here and check the Pages preview.
2. Via WP Engine SFTP/file manager (chaals or IT hold access), back up the
   current `/groups/EthTrust/` directory, then replace its contents with
   `docs/` (keep the path `index.html` at `/groups/EthTrust/index.html`).
3. Purge WP Engine + Cloudflare cache for the path and smoke-check the live
   URL against the Pages preview.

The page is self-contained (one HTML file + logo; CSS comes from the design
system's stable URL), so a deploy is a two-file copy.
