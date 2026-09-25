# EthTrust Security Levels WG — group page

**This repo is the live site.** GitHub Pages serves `docs/` at
<https://entethalliance.github.io/wg-ethtrust-site/>, and since 2026-08-24 the
historical entethalliance.org URLs **301-redirect here** (rules in the
WP Engine portal, applied by Redwan):

| Old URL | Redirects to |
|---|---|
| `entethalliance.org/groups/EthTrust/` | the site root |
| `…/specs/ethtrust-sl/` (latest release) | `spec/v3/` |
| `…/specs/ethtrust-sl/vN/` (v1–v3) | `spec/vN/` |
| `…/specs/ethtrust-sl/vN/checklist.html` | `spec/vN/checklist.html` |

## Layout

- `docs/index.html` — the WG page, built on the
  [EEA design system](https://github.com/EntEthAlliance/eea-design-system)
  **editorial family** (`editorial.css` loaded from the design system's Pages
  URL; canonical `.eea-site-bar` / `.eea-colophon` markup; all page-local CSS
  on `--eea-ed-*` tokens).
- `docs/spec/v1..v3/` — content-identical copies of every published spec
  version (+ checklists where one was published), restyled via the
  `docs/spec/ethtrust-editorial-spec.css` overlay. Self-contained: relative
  logo assets are vendored and Cloudflare-obfuscated emails are decoded to
  plain `mailto:` links. All pages carry the shared EEA Pages Google tag and
  self-canonicals.
- `docs/spec/v4/`: **EEA-hosted exploratory technical review**, using v3 as
  the baseline and considering execution changes through Fusaka. The EthTrust
  Working Group is currently inactive. This AI-assisted review does not
  reactivate the Working Group, create an EEA specification, establish a
  certification baseline, or represent EEA approval. Its [workspace index](docs/spec/v4/README.md),
  [working draft](docs/spec/v4/WORKING-DRAFT.md),
  [candidate checklist](docs/spec/v4/CHECKLIST-DRAFT.md) and
  [HTML review page](docs/spec/v4/index.html) are kept consistent. The GBBC RMF
  section is non-normative and implies no endorsement.
- `docs/og-card.png`, `robots.txt`, `sitemap.xml` — SEO layer.

## Deploying

Merge a PR to `main` — GitHub Pages redeploys automatically. There is no
WordPress-side step anymore; the redirect rules make this repo authoritative.
Rollback of the redirects themselves = deleting the rules in the WP Engine
portal (the old files still exist untouched on the WP host).

## History

The page's previous home was `wg-eta-registry/docs/` (now archived); its GH
Pages `index.html` redirected to the WordPress copy from Feb 2025. The
WordPress copy (a 2022 static Bootstrap template uploaded to the server) was
never under version control until this repo. Modernized 2026-08-24 per the
[WG Modernization Playbook](https://github.com/EntEthAlliance/eea-design-system/blob/main/WG_MODERNIZATION_PLAYBOOK.md);
per-WG record in [issue #4](https://github.com/EntEthAlliance/wg-ethtrust-site/issues/4).
Public comment on the spec: [EthTrust-public](https://github.com/EntEthAlliance/EthTrust-public).
