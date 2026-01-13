# Project 3: Configurable Multi-Page Web Scraper

A flexible n8n workflow for scraping multi-page websites with configurable selectors and fields. Supports pagination and extracts structured data from web pages.

## Features

- **Configurable Scraping**: Define target URL, CSS selectors, and fields to extract
- **Multi-Page Support**: Automatically follows pagination links and scrapes all pages
- **Flexible Field Extraction**: Extract text, attributes, or structured data using CSS selectors
- **Dynamic Configuration**: Configure sources and fields via JSON input
- **Template-Based**: Easy to adapt for different websites

## Architecture

```
Manual Trigger → Set Input Config → Get Start URL → Parse HTML 
→ Extract Fields → Handle Pagination → Collect Results
```

## Prerequisites

1. **n8n Instance**: Self-hosted or cloud
2. **Target Website**: Any website with publicly available data
3. **Basic HTML Knowledge**: Understanding of CSS selectors
4. **Browser Developer Tools**: To identify CSS selectors on target website

## Setup Instructions

### 1. Configure Input

Edit the "Input" node in the workflow with your scraping configuration:

```json
{
  "startUrl": "https://quotes.toscrape.com/tag/humor/",
  "nextPageSelector": "li.next a[href]",
  "fields": [
    {
      "name": "author",
      "selector": "span > small.author",
      "value": "text"
    },
    {
      "name": "text",
      "selector": "span.text",
      "value": "text"
    }
  ]
}
```

**Parameters**:
- `startUrl`: The first page to scrape
- `nextPageSelector`: CSS selector for the "next page" link
- `fields`: Array of fields to extract
  - `name`: Field name in output
  - `selector`: CSS selector to find the element
  - `value`: What to extract ("text" for text content, or attribute name)

### 2. Test the Workflow

1. Click "Execute Workflow"
2. Review the extracted data in the results
3. Adjust selectors if needed

### 3. Customize for Your Website

1. Open browser DevTools (F12)
2. Identify the data you want to extract
3. Get the CSS selectors for those elements
4. Update the configuration in the "Input" node

## Example Output

See `sample-data/listings.json` for example scraped and cleansed data.

## Data Cleansing Rules

**Property Type Classification**:
- "บ้าน" / "house" / "บ้านเดี่ยว" → `house`
- "คอนโด" / "condo" / "condo" → `condo`
- "ที่ดิน" / "land" / "land" → `land`
- Others → `other`

**Owner Type Detection**:
- No agent name, direct contact info → `owner` (direct seller)
- Agent/company name present → `agent`
- Unclear → `unknown`

**Price Normalization**:
- Convert "1.5M" or "1.5 million" → `1500000`
- Handle different currencies (if present)
- Remove commas and formatting

**Area Normalization**:
- Accept "150 ตรม" or "150 sqm" → `150` sq.m
- Convert if needed to square meters

## Alert Examples

**LINE Notify Message**:
```
🏠 New Property Listed!

💰 Price: ฿1,500,000
📍 Location: Bangkok, Sukhumvit, Thonglor
🏠 Type: Condo (2BR, 1BA)
📏 Area: 75 sq.m
👤 Owner: Direct Seller

View: [Link to property]
```

## Query Examples

Find owner-direct properties under 2M in Bangkok:

```sql
SELECT * FROM property_listings
WHERE is_owner_direct = true
  AND price < 2000000
  AND province = 'Bangkok'
  AND created_at > NOW() - INTERVAL '7 days'
ORDER BY created_at DESC;
```

Find condos in Sukhumvit area:

```sql
SELECT * FROM property_listings
WHERE property_type = 'condo'
  AND area_name ILIKE '%Sukhumvit%'
  AND source = 'DDProperty'
ORDER BY price ASC;
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Scraping returns empty | Check if website blocks bots; add delays between requests |
| AI cleansing fails | Ensure OpenAI API key is valid; check token quota |
| Duplicates not detected | Database may have connection issues; verify schema |
| LINE alerts not sending | Verify LINE_NOTIFY_TOKEN is correct and not expired |
| High false positives | Tune AI prompt to improve property type detection |

## Advanced Features (Optional)

- **Price Prediction**: ML model to predict if property is good value
- **Image Analysis**: OpenAI Vision to verify listing images count/quality
- **Mortgage Calculator**: Calculate monthly payments based on price
- **Neighborhood Analysis**: Add nearby amenities, transportation info
- **Price History**: Track price changes over time
- **Custom Filters**: User-specific alerts based on preferences (location, price range, type)
- **Web Dashboard**: Display live listings with filtering and search

## Compliance Notes

⚠️ **Web Scraping Disclaimer**:
- Verify ToS of each website before scraping
- Use appropriate delays to avoid overloading servers
- Consider using official APIs if available
- Respect robots.txt files

## Sample Data Structure

See `sample-data/` for:
- `raw-listings.json` - Raw scraped data
- `cleansed-listings.json` - AI-cleansed and normalized data
- `alerts-sent.json` - Examples of alerts generated

## Files

- `workflow.json` - n8n workflow export
- `README.md` - This file
- `sample-data/` - Example datasets

## References

- DDProperty: https://www.ddproperty.com
- LivingInsider: https://www.livinginsider.com
- Kaidee: https://www.kaidee.com
- LINE Notify: https://notify-bot.line.me
- OpenAI API: https://platform.openai.com/docs
