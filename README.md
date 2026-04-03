# buywhere-mcp

MCP server for the [BuyWhere](https://buywhere.ai) product catalog. Lets Claude Desktop, Cursor, Windsurf, and other MCP-compatible agents search and retrieve products without writing any HTTP code.

## Setup

### 1. Get your API key

Sign up at [buywhere.ai/developers](https://buywhere.ai/dashboard) and copy your API key.

### 2. Configure your client

#### Claude Desktop

Open `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows) and add:

```json
{
  "mcpServers": {
    "buywhere": {
      "command": "python",
      "args": ["mcp_server.py"],
      "env": {
        "BUYWHERE_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

#### Cursor

Open **Settings → MCP** and add a new server, or edit `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "buywhere": {
      "command": "python",
      "args": ["mcp_server.py"],
      "env": {
        "BUYWHERE_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

#### Windsurf

Edit `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "buywhere": {
      "command": "python",
      "args": ["mcp_server.py"],
      "env": {
        "BUYWHERE_API_KEY": "your_api_key_here"
      }
    }
  }
}
```

### 3. Ask your agent to search products

Restart your client, then try:

> *"Search BuyWhere for wireless earbuds under $50 in Singapore"*

> *"Use BuyWhere to compare the cheapest iPhone 15 cases"*

> *"Get me the affiliate link for product ID abc123 from BuyWhere"*

---

## Tools

### `search_products`

Search the BuyWhere catalog using keywords or natural language.

| Parameter  | Type    | Required | Description |
|------------|---------|----------|-------------|
| `query`    | string  | yes      | Keyword or natural-language query |
| `country`  | string  | no       | Country code hint (`sg`, `my`, `id`) |
| `category` | string  | no       | Category slug (e.g. `electronics/smartphones`) |
| `limit`    | integer | no       | Max results, 1–50 (default 10) |

### `get_product`

Fetch full details for a single product.

| Parameter    | Type   | Required | Description |
|--------------|--------|----------|-------------|
| `product_id` | string | yes      | BuyWhere product ID |

### `get_price`

Compare current prices for a product across all available merchants. Returns listings ranked by total price (including shipping), plus best-price and best-value recommendations.

| Parameter    | Type   | Required | Description |
|--------------|--------|----------|-------------|
| `product_id` | string | yes      | BuyWhere product ID |

### `compare_prices`

Side-by-side comparison of 2–5 products. Returns structured differentiators, price range, pros/cons, and a best-value recommendation — purpose-built for AI agent decision-making.

| Parameter     | Type     | Required | Description |
|---------------|----------|----------|-------------|
| `product_ids` | string[] | yes      | Array of 2–5 BuyWhere product IDs |

### `get_affiliate_link`

Get the click-tracked BuyWhere affiliate link for a product. Always use this instead of raw product URLs when sharing links with users — it logs the referral and redirects to the merchant page.

| Parameter    | Type   | Required | Description |
|--------------|--------|----------|-------------|
| `product_id` | string | yes      | BuyWhere product ID |

### `get_catalog`

List available product categories. Use this to discover what categories exist before searching or filtering.

| Parameter     | Type   | Required | Description |
|---------------|--------|----------|-------------|
| `parent_slug` | string | no       | Parent slug for subcategories (omit for top-level) |

## Resources

| URI | Description |
|-----|-------------|
| `buywhere://catalog/sg` | Available categories in Singapore |
| `buywhere://catalog/my` | Available categories in Malaysia |
| `buywhere://catalog/id` | Available categories in Indonesia |

## Environment variables

| Variable           | Required | Description |
|--------------------|----------|-------------|
| `BUYWHERE_API_KEY` | yes      | Your BuyWhere API key |
| `BUYWHERE_API_URL` | no       | Override API base URL (default: `https://api.buywhere.ai`) |

## Sample agent conversation

```
User: Find the cheapest mechanical keyboard on BuyWhere and compare the top 2

Claude: I'll search BuyWhere for mechanical keyboards and compare the top results.

[Calls search_products with query="mechanical keyboard", limit=5]

Found 12 products. Top results:
1. Keychron K2 — SGD 89.00 (prod_abc123)
2. Royal Kludge RK61 — SGD 65.00 (prod_def456)

[Calls compare_prices with product_ids=["prod_abc123", "prod_def456"]]

Comparison: Keychron K2 vs Royal Kludge RK61
Price range: SGD 65 – SGD 89
Key differences:
  • Keychron K2: full-size TKL layout, Mac-optimised keycaps
  • RK61: compact 60% layout, budget-friendly
Best value: Royal Kludge RK61 at SGD 65

[Calls get_affiliate_link with product_id="prod_def456"]

Here's your purchase link: https://api.buywhere.ai/r/prod_def456
```

## Development

```bash
npm install
npm run build
BUYWHERE_API_KEY=your_key python mcp_server.py
```
