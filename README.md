# Steam Independent Games: Launch Intelligence (Power BI)

An interactive Power BI report that helps a small game studio decide what genre, price and release date give a new Steam game the best chance of success.

Data: [Steam Games Dataset](https://huggingface.co/datasets/FronkonGames/steam-games-dataset) by FronkonGames, collected from the Steam Store API and SteamSpy (snapshot: January 2026). The raw CSV is not included in this repository because of its size.

---

## Situation

Steam is the largest PC game store, and it gets more crowded every year. A lot of new games come from small, independent studios. For a small team with a limited budget, a single launch can decide whether the studio survives, yet many games launch quietly and never reach a meaningful audience.

Small studios usually make key launch decisions, such as genre, price and release date, based on intuition or on the stories of a few big hits, which are the exception rather than the rule.

## Task

The task was to build a report that helps a small independent studio plan its next Steam release with data instead of intuition. The report had to:

- show how the Steam market has changed and how hard it is for a typical game to get noticed,
- compare genres and tags by both competition and chance of success,
- show how launch price and release timing relate to a game's performance,
- let a studio check its own game concept against the rest of the market.

To compare games consistently, the project also needed a clear definition of a "successful" game that could be measured with publicly available data.

## Action

The report answers four business questions:

1. **Market:** How has the Steam market changed between 2015 and 2025, and how hard is it for an average game to get noticed?
2. **Genre:** Which tags combine relatively low competition with a high chance of success?
3. **Price:** How does the launch price relate to the number and quality of reviews?
4. **Timing:** Do the launch month and weekday affect a game's performance?

### Defining success
Steam does not publish sales data, and the "estimated owners" field is only available as broad ranges. Review count is a common industry proxy for sales, so a game is defined as successful when it has at least 500 reviews and at least 80% positive reviews.

### Data preparation (Power Query)
- Fixed a broken header in the source CSV: two column names (`Discount` and `DLC count`) were merged into one, which shifted every following column by one position (e.g. support e-mails appeared in the `Windows` column).
- Converted dates and prices using the en-US locale (source dates use the `Oct 21, 2008` format and prices use a decimal point).
- Removed unused columns (descriptions, URLs, screenshots), rows without a name or release date, and duplicate `AppID`s.
- Kept games released 2015-2025.
- Added calculated columns for review metrics, price bands, release dates and game flags, e.g. `Total Reviews`, `Positive %`, `Price Band`, `Is Successful` and `Is Independent`.

### Data model
- Star schema: `Games` (one row per game) + `Date` table + `GameTags` bridge table (one row per game-tag pair, created by splitting the comma-separated `Tags` column).
- `GameTags → Games` relationship with bidirectional filtering, so selecting a tag filters games.
- Grouped all adult-content tags into a single Adult Content tag, de-duplicated per game.
- Flagged non-genre tags (e.g. Singleplayer, 2D, 3D, Indie, Early Access) and tags that describe an outcome rather than a pre-launch decision (e.g. Classic, Beautiful, Moddable, Masterpiece). These are excluded from tag comparisons, but not from market-level metrics.

### Analytical decisions
- **Success is measured on releases from 2015-2023.** Games released in 2024-2025 have not had enough time to collect reviews, so including them would make recent years and fast-growing tags look worse than they are.
- **Competition is measured as the number of releases in 2024-2025**, which shows how crowded a tag is now.
- **Tags are compared with the average tag success rate, not the market average.** Popular games collect more player-added tags, so almost every tag beats the market average. Averaging over game-tag pairs removes this bias.
- **Minimum sample sizes:** only tags with at least 500 releases are compared (adjustable with a What-if parameter), and heatmap cells with fewer than 30 games are left blank.
- **Log scale** on the competition axis, because tag sizes range from about 100 to 10,000 releases.

### Report pages

#### 1. Market Overview
Answers question 1. Releases per year (independent vs other developers), success rate by release year, the most common tags and the distribution of Steam review scores.

<img width="1452" height="815" alt="image" src="https://github.com/user-attachments/assets/a899464e-9c0d-4c3c-a459-f6992ebb96b7" />

#### 2. Genres & Tags
Answers question 2. A scatter chart of competition (releases in 2024-2025, log scale) against success rate, split into four quadrants by the median competition and the average tag success rate, plus a sortable tag performance table.

<img width="1453" height="814" alt="image" src="https://github.com/user-attachments/assets/b553517d-420d-471e-ae33-0ffbd0dd9f76" />

#### 3. Pricing
Answers question 3. Success rate and median reviews by price band, the price band mix over time and a tag and price band heatmap.

<img width="1451" height="813" alt="image" src="https://github.com/user-attachments/assets/1d0cbdc9-6c55-408a-92c9-9d68512c636c" />

#### 4. Release Timing
Answers question 4. Releases and success rate by launch month, the share of releases and success rate by weekday, and a weekday and month heatmap.

<img width="1452" height="811" alt="image" src="https://github.com/user-attachments/assets/ff7632bd-da22-4d0e-87e3-e1fd17515c85" />

#### 5. Concept Check
An interactive "what-if" page: a studio selects a tag, price band, launch month and Early Access status, and the page compares that segment with the whole market (success rate, median reviews, % positive reviews, review score distribution and success rate by price band).

<img width="1453" height="814" alt="image" src="https://github.com/user-attachments/assets/8797414f-efa2-4afa-8875-c6f14dab62b6" />

## Result

### Market
- Steam releases grew 10 times in ten years (2.5K in 2015 → 25.0K in 2025, +24.7% in 2025 alone). That is about 480 new games every week.
- 83.5% of games released since 2015 come from independent developers.
- The median game released since 2015 has only 6 reviews, and 55% of games never reach 10 reviews.
- Only 4.8% of all games released in 2015-2025 meet the success definition, partly because recent releases haven't had time to collect reviews.

### Genre
- Low competition, high success: Dating Sim (23.3% success rate, 537 releases in 2024-2025), Turn-Based (21.3%, 244), Metroidvania (16.7%, 315).
- High success despite high competition: Psychological Horror (18.5%, 2,214 releases), Story Rich (17.8%, 3,554), Sandbox (17.8%, 1,281).
- Crowded and weak: Casual, Action, Arcade, Linear and Hidden Object: many releases, below-average success.

### Price
- Success rate grows with price up to $20-29.99 (18.6%) and then falls for $30+ (8.2%). Games under $5 have the lowest success rates (Free 4.6%, < $5 5.0%).
- The same pattern repeats in all 10 of the largest tags: $20-29.99 is the best price band in every one of them. Story Rich games in this band reach 41.1%.
- Free-to-play games are a growing share of releases (21.7% overall) but have a lower success rate (4.6%) than paid games (6.9%).

### Timing
- Thursday and Friday account for 44% of releases, weekends for less than 10%.
- Thursday (8.2%) and Tuesday (7.9%) have the highest success rates, while weekend releases have the lowest (2.3-2.7%).
- Differences between months are much smaller than between weekdays. October is the busiest month (9.9% of releases) and April has the highest success rate (7.1%).

### Example: Concept Check
A Mystery game priced $5-9.99, released in September, not in Early Access: 99 comparable games, 27.3% success rate (+20.8 percentage points vs market), a median of 108 reviews (market: 15) and 93% positive reviews (market: 85%).

### What this means for a small studio
1. Avoid the largest, most generic categories (Casual, Action) unless the game clearly stands out. Look for tags in the "niche, high success" quadrant.
2. Don't compete on price. Games that can justify $10-30 perform much better than those priced under $5.
3. Release mid-week (Tuesday-Thursday), as established publishers do, and avoid weekends.

### Limitations
- **Correlation, not causation.** Higher-priced games are usually bigger productions, and mid-week releases are typical for experienced publishers. Raising the price or moving the launch day alone will not make a game successful.
- **Reviews are a proxy for sales.** The ratio of reviews to sales differs between games.
- **Tags are user-generated.** They reflect how players describe a game after release, and popular games collect more tags.
- **Recent releases** (2024-2025) have not had time to collect reviews and are excluded from success metrics.
- **Early Access** status is taken from the game's current genres, so games that have already left Early Access may not be flagged.
- **Small segments:** combining several filters on the Concept Check page can leave few games, so results for small segments are indicative only.
- **Single snapshot** of the data (January 2026), not a time series of each game's performance.
