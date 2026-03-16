# WAF Regex Builder

A zero-dependency, client-side regex pattern builder designed for WAF (Web Application Firewall) operators. Paste a value from your logs, select match options with toggle buttons, and get a ready-to-use regex for blocking rules.

**[Live Demo](https://aeon-7.github.io/regex-builder/)**

## Features

- **Instant generation** — regex updates in real-time as you type and toggle options
- **Toggle-based UI** — no dropdowns, no forms to submit. Click toggles, get regex.
- **Smart Suggestions** — auto-detects value type (IP, path, user-agent) and offers contextual patterns
- **URL encoding detection** — catches single-layer (`%XX`) and nested/double-layer (`%25XX`) encoding evasion
- **Zero dependencies** — single HTML file, no frameworks, no build step, no external requests
- **Dark theme** — designed for security operations dashboards
- **Mobile responsive** — works on any screen size
- **Privacy-first** — everything runs client-side. No data leaves your browser.

## Match Options

### Match Type (pick one)
| Option | What It Does | Example |
|--------|-------------|---------|
| **Contains** | Matches if value appears anywhere | `admin` matches `/wp-admin/setup.php` |
| **Exact Match** | Entire string must equal the value | `/wp-admin/` matches only `/wp-admin/` |
| **Starts With** | String must begin with the value | `/wp-` matches `/wp-admin/`, `/wp-content/` |
| **Ends With** | String must end with the value | `.php` matches `/setup.php`, `/xmlrpc.php` |

### Case Sensitivity (pick one)
| Option | When to Use |
|--------|-------------|
| **Case Insensitive** (default) | Most blocking rules — attackers randomize case |
| **Case Sensitive** | Precision matching for known-exact values |

### Encoding Detection (optional, multi-select)
| Option | What It Catches |
|--------|----------------|
| **URL Encoded (%XX)** | `../` also matches `%2e%2e%2f` |
| **Nested/Double Encoded (%25XX)** | `../` also matches `%252e%252e%252f` (evasion technique) |

### Additional Options (optional, multi-select)
| Option | What It Does |
|--------|-------------|
| **Negate** | Blocks everything EXCEPT the value (positive security) |
| **Wildcard Segments** | `/wp/admin/setup.php` becomes `/wp/.*?/admin/.*?/setup\.php` |
| **Word Boundaries** | `admin` won't match `administrator` or `sysadmin` |
| **IP Address Mode** | Auto-escapes dots for IP regex (`1.2.3.4` -> `1\.2\.3\.4`) |

## Deployment

### Option 1: GitHub Pages (easiest)

1. Fork this repository
2. Go to Settings > Pages > Source: Deploy from branch > `main` / `/ (root)`
3. Your regex builder is live at `https://yourusername.github.io/regex-builder/`

### Option 2: Static file server

The tool is a single `index.html` file with no dependencies. Serve it from any web server:

```bash
# Nginx
cp index.html /var/www/html/regex-builder/index.html

# Caddy
cp index.html /srv/tools/regex-builder.html

# Python (quick test)
python3 -m http.server 8080
```

### Option 3: Docker (one-liner)

```bash
docker run -d -p 8080:80 -v $(pwd)/index.html:/usr/share/nginx/html/index.html:ro nginx:alpine
```

### Option 4: Embed in Grafana

Add a **Text panel** in Grafana with an HTML link to the tool:

```html
<a href="https://your-domain/tools/regex-builder.html" target="_blank"
   style="display:inline-block; padding:0.6rem 1.5rem; background:#a78bda;
   color:#161618; border-radius:20px; text-decoration:none; font-weight:600;">
   Open Regex Builder
</a>
```

### Option 5: Integrate with a WAF stack

If you're running Caddy + Coraza WAF (like the [Cosmic Mind](https://github.com/AEON-7/cosmic-mind) project), mount the file and serve it behind your auth layer:

```yaml
# docker-compose.yml
services:
  tools-server:
    image: caddy:2-alpine
    volumes:
      - ./index.html:/srv/tools/regex-builder.html:ro
    command: caddy file-server --root /srv/tools --listen :8080
```

## Configuration

No configuration needed. The tool is entirely self-contained in `index.html`.

To customize the color scheme, edit the CSS variables at the top of the file:

```css
/* Purple theme (default) */
.toggle-btn.active { background: #a78bda; }     /* primary accent */
.output-box { color: #8CF594; }                   /* regex output color */
body { background: #161618; }                      /* background */

/* To switch to a blue theme: */
.toggle-btn.active { background: #3B82F6; }
```

## Use Cases

- **WAF rule authoring** — Generate Coraza/ModSecurity SecRule patterns from log values
- **CrowdSec pattern blocks** — Build regex for custom CrowdSec scenarios
- **Nginx/Caddy location blocks** — Create path-matching patterns
- **Log analysis** — Build grep/ripgrep patterns from suspicious log entries
- **Security training** — Teach regex concepts in a WAF context with real examples

## How It Works

The tool runs entirely in the browser using vanilla JavaScript. When you type a value:

1. Special regex characters are escaped (`.*+?^${}()|[]\` -> `\.\*\+\?...`)
2. Match type wrapping is applied (`^`, `$`, or both)
3. Case sensitivity flag is prepended (`(?i)` for case-insensitive)
4. Encoding alternations are generated (each character becomes `(?:literal|%HexCode)`)
5. Additional transforms are applied (negation, word boundaries, wildcards)

All operations compose — you can combine any match type + case sensitivity + encoding + extras.

## Contributing

Issues and pull requests welcome. The entire tool is a single HTML file — keep it that way.

## License

MIT
