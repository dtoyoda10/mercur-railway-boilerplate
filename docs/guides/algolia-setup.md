# Algolia Search Setup

Add lightning-fast product search to your marketplace.

## Why Algolia?

- Super fast search (results in <10ms)
- Typo tolerance (finds "iphone" even if user types "ifone")
- Faceted search (filter by price, brand, category)
- Better than basic SQL LIKE queries
- Free tier covers ~10k searches/month

## Step 1: Create Algolia Account

1. Sign up at [algolia.com](https://www.algolia.com/)
2. Create a new application
3. Choose a region close to your users (US, EU, etc.)

## Step 2: Get Your API Keys

1. Go to **Settings** → **API Keys**
2. You'll see three keys:
   - **Application ID** - Your app identifier
   - **Search-Only API Key** - Safe for frontend
   - **Admin API Key** - For backend indexing (keep secret!)

### Add to Backend

Backend needs Admin API Key to index products:

```env
# apps/backend/.env
ALGOLIA_APP_ID=ABC123XYZ
ALGOLIA_API_KEY=abc123...xyz  # Admin API Key
```

### Add to Storefront

Storefront only needs Search-Only key:

```env
# apps/storefront/.env
NEXT_PUBLIC_ALGOLIA_ID=ABC123XYZ
NEXT_PUBLIC_ALGOLIA_SEARCH_KEY=def456...uvw  # Search-Only API Key
```

## Step 3: Create Indices

Indices are like database tables for search. This boilerplate uses:

- `products` - Main product search
- `products_price_asc` - Products sorted by price (low to high)
- `products_price_desc` - Products sorted by price (high to low)

### Auto-Creation

Good news - the Medusa Algolia module creates these automatically when you first start the backend with Algolia configured.

### Manual Creation (Optional)

If you want to create them manually:

1. Go to Algolia Dashboard → **Search**
2. Click **Create Index**
3. Name it `products`
4. Repeat for other indices

## Step 4: Index Your Products

### Automatic Indexing

The boilerplate auto-indexes products when:

- New product is created
- Product is updated
- Product is deleted

It's all handled by the Algolia module in `medusa-config.ts`.

### Manual Reindexing

If you need to reindex all products (e.g., after changing index settings):

```bash
cd apps/backend

# Option 1: Via Medusa CLI (if available)
npx medusa algolia reindex

# Option 2: Via API call
curl -X POST http://localhost:9000/admin/algolia/reindex \
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

## Step 5: Configure Search Settings

Make search work better by tweaking Algolia settings:

### Searchable Attributes

Tell Algolia which fields to search:

1. Go to **Configuration** → **Searchable attributes**
2. Add these in order (order = importance):
   - `title`
   - `description`
   - `metadata.brand`
   - `collection.title`

### Facets for Filtering

Enable filtering by attributes:

1. Go to **Configuration** → **Facets**
2. Add:
   - `collection.title`
   - `metadata.brand`
   - `variants.prices.amount`
   - `tags.value`

### Ranking and Sorting

Set custom ranking criteria:

1. Go to **Configuration** → **Ranking and Sorting**
2. Add custom ranking attributes:
   - `desc(popularity)` (if you track popularity)
   - `asc(variants.prices.amount)` (cheaper products rank higher)

## Step 6: Test Search

### Test from Dashboard

1. Go to Algolia Dashboard → **Search**
2. Select your `products` index
3. Type a search query
4. See results in real-time

### Test from Your Storefront

1. Go to your storefront
2. Use the search bar
3. Type product names, brands, or categories
4. Results should appear instantly

**If no results appear:**
- Check products are indexed (Algolia Dashboard → Browse)
- Check API keys are correct in `.env`
- Check console for errors
- Verify backend is sending events to Algolia

## Common Issues

### Products not showing in search

**Fixes:**
1. Check if products are in Algolia index (Dashboard → Browse)
2. Check `ALGOLIA_API_KEY` is the Admin API Key (not Search-Only)
3. Restart backend after adding env vars
4. Manually reindex products

### Search returns no results even though products exist

**Fixes:**
1. Check searchable attributes are configured
2. Verify index name matches in code (`products`)
3. Check for typos in search query (Algolia has typo tolerance but it's not magic)

### Getting rate limited

You're hitting the free tier limits:

- Free: 10,000 search requests/month
- Solution: Upgrade to a paid plan or optimize search usage

## Going Live Checklist

- [ ] Configure searchable attributes
- [ ] Set up facets for filtering
- [ ] Configure custom ranking
- [ ] Test search on production data
- [ ] Set up monitoring (Algolia Dashboard → Analytics)
- [ ] Consider upgrading if you expect high traffic

## Advanced: InstantSearch UI

This boilerplate uses Algolia's InstantSearch widgets for the search UI.

**Customization:**

Search components are in `apps/storefront/src/components/search/`

You can customize:
- Search box styling
- Result display
- Filters (facets)
- Pagination

Check [InstantSearch React docs](https://www.algolia.com/doc/guides/building-search-ui/what-is-instantsearch/react/) for more widgets.

## Resources

- [Algolia Documentation](https://www.algolia.com/doc/)
- [InstantSearch React](https://www.algolia.com/doc/guides/building-search-ui/what-is-instantsearch/react/)
- [Algolia + MedusaJS Guide](https://docs.medusajs.com/plugins/search/algolia)
- [Search Best Practices](https://www.algolia.com/doc/guides/managing-results/relevance-overview/)

## Need Help?

- [Algolia Support](https://www.algolia.com/support/)
- [Algolia Community](https://discourse.algolia.com/)
