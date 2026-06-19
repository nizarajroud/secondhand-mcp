[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/jlsookiki-secondhand-mcp-badge.png)](https://mseep.ai/app/jlsookiki-secondhand-mcp)

# Secondhand MCP (Fork — Enhanced)

> Fork of [jlsookiki/secondhand-mcp](https://github.com/jlsookiki/secondhand-mcp) with enhanced Facebook Marketplace features.

A [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server that lets AI assistants search secondhand marketplaces. Search Facebook Marketplace, eBay, Depop, and Poshmark for used and secondhand items — filter by price, category, condition, size, and color, then get full listing details with photos, descriptions, and seller info.

## Enhancements (this fork)

| Feature | Description |
|---------|-------------|
| **Publication date** | `postedAt` (ISO) + `postedAtRelative` ("il y a 18 minutes") in listing details |
| **Search radius** | Configurable `radius` in miles (converted to km internally) |
| **Days since listed** | `daysSinceListed` filter (1, 7, or 30 days) for Facebook |
| **Sort by newest** | `sort: "newest"` support for Facebook (CREATION_TIME_DESCEND) |
| **Wildcard browse** | Empty query or `"*"` → automatically uses `"e"` to browse all listings |
| **Commercial detection** | `isCommercial` field in details — identifies "En stock" sellers vs personal sales |

## Quick Start

```bash
git clone https://github.com/nizarajroud/secondhand-mcp.git
cd secondhand-mcp
npm install
npm run build
```

### MCP Configuration (Kiro / Claude Desktop)

```json
{
  "secondhand-mcp": {
    "command": "node",
    "args": ["/path/to/secondhand-mcp/dist/index.js"],
    "env": {
      "MARKETPLACES": "facebook"
    }
  }
}
```

## Prompt Samples

### Browse all recent listings in a location

> "Montre-moi tous les articles publiés aujourd'hui à Longueuil sous 100$."

### Search with all filters (price, radius, age, sort)

> "Cherche une table à manger sur Facebook Marketplace à Montréal, dans un rayon de 15 miles, entre 50$ et 300$, publiée dans les 7 derniers jours, triée par plus récent."

### Get publication time for each result

> "Cherche 3 tables à Longueuil entre 20$ et 200$, triées par plus récent. Pour chaque résultat, récupère les détails et affiche le titre, prix, localisation, et depuis quand c'est publié."

### Exclude commercial sellers (personal sales only)

> "Cherche 5 écouteurs bluetooth à Montréal sous 50$. Pour chaque résultat, récupère les détails et affiche uniquement les annonces de particuliers (exclut les commerciaux). Montre le titre, prix, et depuis quand c'est publié."

## Supported Marketplaces

| Marketplace | Auth Required | Notes |
|-------------|---------------|-------|
| Facebook Marketplace | No | Location-based search |
| eBay | Yes (API keys) | Official Browse API |
| Depop | No | Requires Chrome installed |
| Poshmark | No | Requires Chrome installed |

## Setup

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "secondhand": {
      "command": "npx",
      "args": ["-y", "secondhand-mcp"],
      "env": {
        "EBAY_CLIENT_ID": "your-ebay-client-id",
        "EBAY_CLIENT_SECRET": "your-ebay-client-secret"
      }
    }
  }
}
```

### Claude Code

Add to `~/.claude/.mcp.json`:

```json
{
  "mcpServers": {
    "secondhand": {
      "command": "npx",
      "args": ["-y", "secondhand-mcp"],
      "env": {
        "EBAY_CLIENT_ID": "your-ebay-client-id",
        "EBAY_CLIENT_SECRET": "your-ebay-client-secret"
      }
    }
  }
}
```

eBay, Depop, and Poshmark are all optional — if eBay API keys are missing or Chrome isn't installed, those marketplaces are automatically disabled and the rest still work.

### Depop & Poshmark / Chrome Requirement

Depop and Poshmark require a headless browser. If **Google Chrome or Chromium** is installed on your system, both are automatically enabled — no config needed. If Chrome isn't found, they are silently skipped.

On macOS, the first time you search Depop or Poshmark, you may see a system prompt asking to allow Node.js to control Chrome. This is expected — puppeteer needs to launch Chrome in headless mode. Allow it once and it won't ask again.

The browser runs invisibly in the background and only launches when you actually search Depop or Poshmark.

## Configuration

### Choosing Marketplaces

By default all marketplaces are enabled. To limit which are active, set the `MARKETPLACES` env var (comma-separated):

```json
{
  "env": {
    "MARKETPLACES": "facebook,ebay"
  }
}
```

Valid values: `facebook`, `ebay`, `depop`, `poshmark`

### eBay API Keys

eBay uses the official [Browse API](https://developer.ebay.com/api-docs/buy/browse/overview.html). You need a free eBay developer account:

1. Create an account at [developer.ebay.com](https://developer.ebay.com)
2. Create an application to get a Client ID and Client Secret
3. Add them to your MCP config as `EBAY_CLIENT_ID` and `EBAY_CLIENT_SECRET`

## Tools

### `search_marketplace`

Search for items across marketplaces.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `query` | No | | Search terms (empty or `"*"` browses all listings) |
| `marketplace` | No | `facebook` | `facebook`, `ebay`, `depop`, `poshmark`, or `all` |
| `location` | No | `san francisco` | City to search in (Facebook only) |
| `maxPrice` | No | | Maximum price |
| `minPrice` | No | | Minimum price |
| `radius` | No | 20km | Search radius in miles (Facebook only) |
| `daysSinceListed` | No | | Filter by listing age: 1, 7, or 30 days (Facebook only) |
| `limit` | No | `20` | Max results |
| `showSold` | No | `false` | Include sold items (Facebook only) |
| `includeImages` | No | `false` | Include image URLs in output |
| `sort` | No | `relevance` | Sort order. Facebook: `relevance`, `newest`. Depop/Poshmark: `relevance`, `newest`, `most_popular`, `price_low_to_high`, `price_high_to_low` |
| `condition` | No | | Item condition. eBay: `new`, `like_new`, `good`, `fair`. Depop: `new`, `like_new`, `excellent`, `good`, `fair`, `used`. Poshmark: `new` (NWT), `like_new` (NWOT), `good`, `fair` |
| `category` | No | | Product category. Depop: `tops`, `bottoms`, `dresses`, `coats-jackets`, `footwear`, `accessories`, `bags`, `jewellery`, `activewear`, `swimwear`. Poshmark: `Jackets_&_Coats`, `Dresses`, `Shoes`, `Accessories`, etc. |
| `brand` | No | | Brand filter (Poshmark only): e.g. `"Nike"`, `"Levi's"`, `"Gucci"` |
| `department` | No | | Department filter (Poshmark only): `Women`, `Men`, `Kids` |
| `sizes` | No | | Size filter (Depop, Poshmark): e.g. `["S", "M", "L"]` or `["US 9", "US 10"]` |
| `colors` | No | | Color filter (Depop, Poshmark): `black`, `white`, `red`, `blue`, `green`, `yellow`, `orange`, `pink`, `purple`, `brown`, `grey`, `cream`, `multi`, `silver`, `gold` |

**Data returned per marketplace:**

| Field | Facebook | eBay | Depop | Poshmark |
|-------|----------|------|-------|----------|
| Title | Yes | Yes | Yes | Yes |
| Price | Yes | Yes | Yes | Yes |
| Location | City | City, State | — | — |
| Condition | — | Yes | — | — |
| Photo count | 1 thumbnail | 1 thumbnail | 1 thumbnail | 1 thumbnail |
| Seller | Yes | Yes | — | — |

### `get_listing_details`

Get full details for a specific listing using an ID from search results.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `listingId` | Yes | | Listing ID from search results |
| `marketplace` | No | `facebook` | `facebook`, `ebay`, `depop`, or `poshmark` |

**Data returned per marketplace:**

| Field | Facebook | eBay | Depop | Poshmark |
|-------|----------|------|-------|----------|
| Description | Yes | Yes | Yes | Yes |
| All photos | Yes | Yes | Yes | Yes |
| Location | City | City, State, Country | — | — |
| Seller | Name | Username | Username | Username |
| Delivery types | Yes | — | — | — |
| Shipping | Yes/No | Service codes | Yes/No | Always included |
| Publication date | Yes (`postedAt` + `postedAtRelative`) | — | — | — |
| Commercial detection | Yes (`isCommercial`) | — | — | — |

> **Note**: `postedAt`, `postedAtRelative`, and `isCommercial` are only available via `get_listing_details` (not in search results). The agent must call details for each listing to access these fields.

### `list_marketplaces`

List all enabled marketplaces and their status.

## How It Works

**Facebook Marketplace** — Searches listings by location, price, and query. Resolves city names to coordinates. No login or browser needed.

**eBay** — Uses the official eBay Browse API with OAuth 2.0 client credentials. Tokens are cached and auto-refreshed.

**Depop** — Uses a headless browser to search listings with support for category, condition, size, and color filters. The browser instance is shared across requests.

**Poshmark** — Uses a headless browser to search listings with support for condition, size, color, sort, and price filters. Poshmark is not location-based — all items ship nationally.

## Development

```bash
git clone https://github.com/jlsookiki/secondhand-mcp.git
cd secondhand-mcp
npm install
npm run build
```

### Adding a Marketplace

1. Create a new file in `src/marketplaces/`
2. Extend `BaseMarketplace` and implement `search()` and optionally `getListingDetails()`
3. Add the constructor to `allMarketplaces` in `src/marketplaces/index.ts`

## Limitations

- **Facebook**: May break if Facebook changes their frontend
- **eBay**: Requires developer API keys (free tier available)
- **Depop**: Requires Chrome/Chromium installed; slower than Facebook/eBay (~5s per search)
- **Poshmark**: Requires Chrome/Chromium installed; no official API so relies on page scraping
- **Rate limiting**: Don't make too many requests too quickly

## License

MIT
