# Project 3: Real Estate Scraping → Cleansing → Alert

Automated workflow to scrape property listings from multiple Thai real estate websites, cleanse and normalize data using AI, detect new listings, and send real-time alerts via LINE Notify.

## Features

- **Multi-source Scraping**: DDProperty, LivingInsider, Kaidee Property, and other Thai real estate portals
- **AI-powered Data Cleansing**: Automatically normalize prices, classify property types, detect owner-direct listings
- **Duplicate Detection**: Prevents duplicate alerts for same property
- **Smart Alerts**: LINE Notify integration for new listing notifications
- **Data Enrichment**: Location normalization, area standardization
- **Database Integration**: Stores all listings for historical analysis

## Architecture

```
Scheduled Trigger → Scrape Multiple Sites → Merge Data 
→ AI Cleanse & Normalize → Duplicate Check → Alert (LINE Notify) → Save Database
```

## Prerequisites

1. **n8n Instance**: Self-hosted or cloud
2. **OpenAI API Key**: For AI data cleansing
3. **LINE Notify Token**: For alert notifications
4. **Database**: PostgreSQL, MongoDB, or Supabase for storing listings
5. **Web Scraping**: HTTPClient nodes (websites allow scraping) or consider legal/ToS compliance

## Setup Instructions

### 1. Environment Variables

Create `.env` file:

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxx
LINE_NOTIFY_TOKEN=xxxxxxxxxxxxx
DATABASE_URL=postgresql://user:password@localhost/properties
DB_TYPE=postgres
SCRAPE_INTERVAL_MINUTES=30
ALERT_CHANNELS=all
FILTERS_MIN_PRICE=500000
FILTERS_MAX_PRICE=50000000
FILTERS_PROPERTY_TYPES=house,condo,land
```

### 2. Database Schema

```sql
CREATE TABLE property_listings (
  id UUID PRIMARY KEY,
  listing_id VARCHAR(255) UNIQUE,
  source VARCHAR(50),
  title VARCHAR(500),
  price DECIMAL(15,2),
  price_currency VARCHAR(10),
  area_sqm DECIMAL(10,2),
  bedrooms INT,
  bathrooms INT,
  property_type VARCHAR(50),
  owner_type VARCHAR(50),
  province VARCHAR(100),
  district VARCHAR(100),
  area_name VARCHAR(200),
  description TEXT,
  url VARCHAR(500),
  is_owner_direct BOOLEAN,
  agent_name VARCHAR(255),
  contact_phone VARCHAR(20),
  images_count INT,
  raw_data JSONB,
  cleansed BOOLEAN,
  alert_sent BOOLEAN,
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  scraped_at TIMESTAMP
);

CREATE INDEX idx_listing_id ON property_listings(listing_id);
CREATE INDEX idx_source ON property_listings(source);
CREATE INDEX idx_province ON property_listings(province);
CREATE INDEX idx_property_type ON property_listings(property_type);
CREATE INDEX idx_price ON property_listings(price);
CREATE INDEX idx_is_owner_direct ON property_listings(is_owner_direct);
```

### 3. Configure Scraping Sources

Update n8n nodes to point to actual real estate portals:

**DDProperty**:
```
https://www.ddproperty.com/search/sale/house
```

**LivingInsider**:
```
https://www.livinginsider.com/for-sale
```

**Kaidee Property**:
```
https://www.kaidee.com/property/?c=266
```

Add more sources as needed (Hipster, Propertywise, etc.)

### 4. Deploy Workflow

1. Export workflow to n8n
2. Configure LINE Notify token
3. Set database connection
4. Activate workflow (runs every 30 minutes by default)

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
