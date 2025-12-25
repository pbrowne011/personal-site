# Claude Code Task: Bill Belichick Library Catalogue for USNA

## Project Overview

Create a searchable, interactive catalogue page for Bill Belichick's book collection housed at the US Naval Academy. This will be integrated into an existing Hugo-based personal website using the ezhil theme.

**Repository**: The site uses Hugo with a modified ezhil theme. Key paths:
- Content: `content/belichick.md` (current page)
- Theme: `themes/ezhil/`
- Static assets: `static/` and `docs/`
- CSS: `themes/ezhil/static/css/main.css`, `dark.css`, `normalize.css`
- Output: `docs/belichick/index.html`

**Goal**: Replace the current simple HTML table with an interactive, searchable catalogue that:
1. Loads book data from a JSON file
2. Provides client-side search and filtering
3. Includes external links to WorldCat, Internet Archive, Anna's Archive, and BookFinder
4. Maintains the existing site's visual style
5. Works well on desktop, acceptably on mobile

---

## Phase 1: Data Cleaning and Consolidation

### Input Files

You have two CSV files to merge:

1. **belichick-collection-with-publisher.csv** - Contains:
   - TITLE, AUTHOR, PUBLISHING COMPANY, DATE
   - Location codes (e.g., C3S5 = Case 3, Shelf 5)
   - Category codes (BIO, FH, INST, NAVY, GS)
   - Some duplicate entries marked with (2), (3), (4) indicating multiple copies

2. **belichick-collection-atd.csv** - Contains:
   - Author, Title, Year
   - Location codes and category codes
   - Different set of books with some overlap

### Data Cleaning Specification

Write a Python script (`clean_belichick_data.py`) that performs the following:

#### 1. CSV Parsing
- Handle UTF-8 BOM if present (the files have `﻿` at start)
- Skip completely empty rows (rows where all fields are empty or just commas)
- Handle quoted fields with embedded commas properly

#### 2. Field Normalization

**Title Cleaning:**
- Expand common abbreviations:
  - "FB" → "Football"
  - "ND" → "Notre Dame"
  - "QB" or "Qback" → "Quarterback"
  - "RB" → "Running Back"
  - "HS" → "High School"
  - "Yrs" → "Years"
  - "Bk" → "Book"
  - "Bldg" → "Building"
  - "Com." → "Complete"
  - "Bal" → "Ball"
  - "Off." → "Offensive"
  - "Def." → "Defensive"
  - "vs." → "vs"
  - "U of" → "University of"
  - "Nat'l" → "National"
- Remove trailing periods
- Preserve original capitalization
- Strip leading/trailing whitespace
- Handle entries like "(2)" or "(3)" at end of title - these indicate duplicate copies; extract the copy count and store separately, remove from title

**Author Cleaning:**
- Normalize "Last, First" format to "First Last" for display
- Handle multiple authors separated by:
  - " and " 
  - " & "
  - "/" (e.g., "Beach/Moore")
  - "with" (e.g., "Ken Stabler with Berry Stainback")
- For "with" authors, format as "Primary Author with Secondary Author"
- Strip leading/trailing whitespace
- Replace "na" or "NA" with empty string
- Handle parenthetical co-authors like "Steve Wright (William Gildea, Kenneth Turan)"

**Year Cleaning:**
- Convert "na", "NA", "", or non-numeric values to null
- Handle ranges like "1967,69" → use first year (1967)
- Handle partial years like "192?" → store as "1920s" or just "192?"
- Ensure years are stored as integers where possible, strings where ambiguous

**Publisher Cleaning:**
- Strip leading/trailing whitespace
- Standardize common variations:
  - "Prentice-Hall INC" → "Prentice-Hall"
  - "& Company" / "and Company" → "& Co."
- Replace empty/na with null

**Category Mapping:**
Create human-readable categories from codes:
- "BIO" → "Biography"
- "FH" → "Football History"
- "INST" → "Instructional"
- "NAVY" → "Navy/Military"
- "GS" → "General Sports"
- Empty/missing → null

**Location Codes:**
- Parse codes like "C3S5" into structured format: `{"case": 3, "shelf": 5}`
- Handle multiple locations (some books appear in multiple places)
- Store as array of location objects

#### 3. Deduplication Strategy

- Create a normalized key from: `lowercase(title) + lowercase(first_author_last_name)`
- When duplicates found:
  - Merge metadata (prefer non-null values)
  - Combine location arrays
  - Keep highest copy count
  - Prefer the more complete title
  - Prefer the record with publisher information

#### 4. Keyword Generation

For each book, generate keywords from:
- All words from title (split on spaces, remove punctuation)
- All author name components (first names, last names)
- Exclude common stop words: "the", "a", "an", "of", "and", "in", "to", "for", "with", "on", "at", "by", "from", "how"

Store keywords as lowercase array, deduplicated.

#### 5. External Link Generation

Generate URL-encoded search links for each book:

```python
import urllib.parse

def generate_links(title: str, author: str) -> dict:
    """Generate external resource links for a book."""
    
    # Clean inputs for URL encoding
    title_clean = title.strip()
    author_clean = author.split(" with ")[0].strip() if " with " in author else author.strip()
    
    # For search queries, use title + primary author
    search_query = f"{title_clean} {author_clean}"
    
    return {
        "worldcat": f"https://search.worldcat.org/search?q={urllib.parse.quote(search_query)}",
        "internet_archive": f"https://archive.org/search?query={urllib.parse.quote(search_query)}",
        "annas_archive": f"https://annas-archive.org/search?q={urllib.parse.quote(search_query)}",
        "bookfinder": f"https://www.bookfinder.com/search/?keywords={urllib.parse.quote(search_query)}&currency=USD&destination=us&mode=basic&classic=off&ps=tp&order=pricedesc&st=sr&ac=qr"
    }
```

**Important Link Notes:**
- WorldCat: Use `search.worldcat.org` (the newer interface)
- Internet Archive: Direct search URL
- Anna's Archive: Simple search query
- BookFinder: Include basic search parameters for used books

#### 6. Output Format

Generate `books.json` with this structure:

```json
{
  "metadata": {
    "generated": "2025-01-15T12:00:00Z",
    "total_books": 450,
    "source": "USNA Belichick Collection"
  },
  "books": [
    {
      "id": "snake-stabler-1986",
      "title": "Snake",
      "title_display": "Snake",
      "author": "Ken Stabler with Berry Stainback",
      "author_sort": "Stabler, Ken",
      "publisher": "Doubleday & Co.",
      "year": 1986,
      "year_display": "1986",
      "category": "Biography",
      "category_code": "BIO",
      "locations": [
        {"case": 3, "shelf": 5}
      ],
      "copies": 1,
      "keywords": ["snake", "ken", "stabler", "berry", "stainback"],
      "links": {
        "worldcat": "https://search.worldcat.org/search?q=Snake%20Ken%20Stabler",
        "internet_archive": "https://archive.org/search?query=Snake%20Ken%20Stabler",
        "annas_archive": "https://annas-archive.org/search?q=Snake%20Ken%20Stabler",
        "bookfinder": "https://www.bookfinder.com/search/?keywords=Snake%20Ken%20Stabler&..."
      }
    }
  ]
}
```

**ID Generation:**
- Create URL-safe slug from: `slugify(title)-slugify(first_author_last_name)-year`
- Example: "Football Scouting Methods" by Belichick, 1962 → `football-scouting-methods-belichick-1962`
- Handle missing years: `football-scouting-methods-belichick`

#### 7. Validation

After processing, output statistics:
- Total books after deduplication
- Books with missing years
- Books with missing authors
- Books with missing publishers
- Category distribution
- Decade distribution
- Any parsing warnings or errors

---

## Phase 2: Interactive HTML Page

### Design Requirements

Create a single HTML file that will be the content of `content/belichick.md` (using Hugo raw HTML) or a standalone HTML file at `static/belichick/index.html`.

**Recommended approach**: Create a standalone HTML file that Hugo will copy to output, referencing the existing CSS files.

### Visual Design Specification

#### Overall Layout

```
┌─────────────────────────────────────────────────────────────┐
│  [Site Header - from existing theme]                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Bill Belichick's Library at USNA                          │
│  ─────────────────────────────────────────                 │
│  [Brief intro paragraph - 2-3 sentences max]               │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🔍 Search books by title, author, or keyword...     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  Showing 342 of 450 books                                   │
│                                                             │
│  Filters: [Category ▼] [Decade ▼]   Sort: [Title ▼]        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Title ▲         │ Author        │ Year │ Links      │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ Snake           │ Ken Stabler   │ 1986 │ 📚 📖 🔍 🛒 │   │
│  │ Biography       │               │      │            │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ Football        │ Walter Camp   │ 1896 │ 📚 📖 🔍 🛒 │   │
│  │ Football History│               │      │            │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │ ...             │               │      │            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  [Site Footer - from existing theme]                        │
└─────────────────────────────────────────────────────────────┘
```

#### Desktop Layout (>768px)

- Full table view with all columns visible
- Search bar: full width, prominent, with placeholder text
- Filter dropdowns: inline, horizontal
- Table: 
  - Title column: ~45% width
  - Author column: ~25% width
  - Year column: ~10% width (right-aligned)
  - Links column: ~20% width (icon buttons)
- Category shown as subtle badge under title
- Clickable column headers for sorting (with sort direction indicator)

#### Mobile Layout (≤768px)

- Search bar: full width
- Filters: stack vertically or collapse into single "Filter" button
- Table transforms to card layout:

```
┌─────────────────────────────────┐
│ Snake                           │
│ Ken Stabler with Berry Stainback│
│ 1986 · Biography                │
│ [WorldCat] [Archive] [Buy]      │
└─────────────────────────────────┘
```

#### Color Scheme

Use existing ezhil theme colors. Reference `themes/ezhil/static/css/main.css` and `dark.css`:

- Background: inherit from theme (likely white/dark gray)
- Text: inherit from theme
- Table borders: subtle, light gray (`#e0e0e0` light / `#333` dark)
- Hover state: subtle background change
- Links: theme's link color
- Search bar: match theme input styles
- Badges (category): muted background, small text

**Do NOT introduce new colors.** Use CSS custom properties if the theme has them, otherwise use the same color values from the existing CSS.

#### Typography

- Use existing theme fonts (likely system fonts or Google Fonts already loaded)
- Title: bold, normal size
- Author: normal weight
- Year: tabular numerals if available, monospace fallback
- Category badge: small, uppercase, letter-spacing

### HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Belichick's Library | Pat Browne</title>
    
    <!-- Use existing theme CSS -->
    <link rel="stylesheet" href="/css/normalize.css">
    <link rel="stylesheet" href="/css/main.css">
    <link rel="stylesheet" href="/css/dark.css" media="(prefers-color-scheme: dark)">
    
    <!-- Page-specific styles - inline to avoid extra request -->
    <style>
        /* Minimal additional styles - see CSS section below */
    </style>
</head>
<body>
    <!-- Include site header/nav if standalone, or let Hugo handle it -->
    
    <main class="container">
        <article class="book-catalogue">
            <header>
                <h1>Bill Belichick's Library at USNA</h1>
                <p class="intro">A collection of [X] football books donated to the United States Naval Academy. 
                Search, browse, and find copies through library systems or booksellers.</p>
            </header>
            
            <section class="catalogue-controls">
                <div class="search-wrapper">
                    <input type="search" 
                           id="book-search" 
                           placeholder="Search by title, author, or keyword..."
                           aria-label="Search books">
                </div>
                
                <div class="filter-sort-row">
                    <div class="filters">
                        <select id="filter-category" aria-label="Filter by category">
                            <option value="">All Categories</option>
                            <option value="Biography">Biography</option>
                            <option value="Football History">Football History</option>
                            <option value="Instructional">Instructional</option>
                            <option value="Navy/Military">Navy/Military</option>
                            <option value="General Sports">General Sports</option>
                        </select>
                        
                        <select id="filter-decade" aria-label="Filter by decade">
                            <option value="">All Decades</option>
                            <!-- Dynamically populated -->
                        </select>
                    </div>
                    
                    <div class="results-count">
                        <span id="showing-count">450</span> of <span id="total-count">450</span> books
                    </div>
                </div>
            </section>
            
            <section class="catalogue-table-wrapper">
                <table class="catalogue-table" id="books-table">
                    <thead>
                        <tr>
                            <th class="sortable" data-sort="title">
                                Title <span class="sort-indicator"></span>
                            </th>
                            <th class="sortable" data-sort="author">
                                Author <span class="sort-indicator"></span>
                            </th>
                            <th class="sortable" data-sort="year">
                                Year <span class="sort-indicator"></span>
                            </th>
                            <th>Find</th>
                        </tr>
                    </thead>
                    <tbody id="books-tbody">
                        <!-- Populated by JavaScript -->
                    </tbody>
                </table>
            </section>
            
            <footer class="catalogue-footer">
                <p>Data sourced from the Belichick Collection at USNA. 
                Links open external sites for finding or purchasing books.</p>
            </footer>
        </article>
    </main>
    
    <!-- Inline JavaScript - no external dependencies -->
    <script>
        // See JavaScript section below
    </script>
</body>
</html>
```

### CSS Specification

Add minimal CSS that extends the theme. Keep it under 100 lines.

```css
/* Book Catalogue Styles */

.book-catalogue {
    max-width: 900px;
    margin: 0 auto;
    padding: 1rem;
}

.intro {
    color: var(--text-secondary, #666);
    margin-bottom: 2rem;
}

/* Search */
.search-wrapper {
    margin-bottom: 1rem;
}

#book-search {
    width: 100%;
    padding: 0.75rem 1rem;
    font-size: 1rem;
    border: 1px solid var(--border-color, #ddd);
    border-radius: 4px;
    background: var(--bg-secondary, #fff);
    color: inherit;
}

#book-search:focus {
    outline: 2px solid var(--link-color, #0066cc);
    outline-offset: 1px;
}

/* Filters */
.filter-sort-row {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    margin-bottom: 1rem;
}

.filters {
    display: flex;
    gap: 0.5rem;
    flex-wrap: wrap;
}

.filters select {
    padding: 0.5rem;
    border: 1px solid var(--border-color, #ddd);
    border-radius: 4px;
    background: var(--bg-secondary, #fff);
    color: inherit;
    font-size: 0.875rem;
}

.results-count {
    font-size: 0.875rem;
    color: var(--text-secondary, #666);
}

/* Table */
.catalogue-table-wrapper {
    overflow-x: auto;
}

.catalogue-table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.9375rem;
}

.catalogue-table th,
.catalogue-table td {
    padding: 0.75rem 0.5rem;
    text-align: left;
    border-bottom: 1px solid var(--border-color, #e0e0e0);
}

.catalogue-table th {
    font-weight: 600;
    white-space: nowrap;
}

.catalogue-table th.sortable {
    cursor: pointer;
    user-select: none;
}

.catalogue-table th.sortable:hover {
    background: var(--bg-hover, rgba(0,0,0,0.03));
}

.sort-indicator::after {
    content: '';
    margin-left: 0.25rem;
}

.sort-asc .sort-indicator::after {
    content: '▲';
}

.sort-desc .sort-indicator::after {
    content: '▼';
}

/* Book row */
.book-title {
    font-weight: 500;
}

.book-category {
    display: inline-block;
    font-size: 0.75rem;
    color: var(--text-secondary, #666);
    text-transform: uppercase;
    letter-spacing: 0.03em;
    margin-top: 0.25rem;
}

.book-author {
    color: var(--text-secondary, #555);
}

.book-year {
    font-variant-numeric: tabular-nums;
    text-align: right;
}

/* Links */
.book-links {
    display: flex;
    gap: 0.5rem;
}

.book-links a {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 28px;
    height: 28px;
    border-radius: 4px;
    text-decoration: none;
    font-size: 0.75rem;
    transition: background 0.15s;
}

.book-links a:hover {
    background: var(--bg-hover, rgba(0,0,0,0.05));
}

.book-links a[title]::before {
    /* Use abbreviations instead of emoji for consistency */
}

/* Link type abbreviations */
.link-wc::after { content: 'WC'; }   /* WorldCat */
.link-ia::after { content: 'IA'; }   /* Internet Archive */
.link-aa::after { content: 'AA'; }   /* Anna's Archive */
.link-bf::after { content: 'BF'; }   /* BookFinder */

/* Alternative: Use small text labels */
.book-links a {
    font-size: 0.6875rem;
    font-weight: 500;
    padding: 0.25rem 0.5rem;
    width: auto;
    border: 1px solid var(--border-color, #ddd);
}

/* Mobile */
@media (max-width: 768px) {
    .catalogue-table thead {
        display: none;
    }
    
    .catalogue-table,
    .catalogue-table tbody,
    .catalogue-table tr,
    .catalogue-table td {
        display: block;
    }
    
    .catalogue-table tr {
        padding: 1rem 0;
        border-bottom: 1px solid var(--border-color, #e0e0e0);
    }
    
    .catalogue-table td {
        padding: 0.25rem 0;
        border: none;
    }
    
    .catalogue-table td:first-child {
        font-weight: 600;
    }
    
    .book-links {
        margin-top: 0.5rem;
    }
}

/* Dark mode adjustments if needed */
@media (prefers-color-scheme: dark) {
    .book-links a {
        border-color: #444;
    }
}

/* No results state */
.no-results {
    padding: 2rem;
    text-align: center;
    color: var(--text-secondary, #666);
}
```

### JavaScript Specification

**Requirements:**
- Zero external dependencies
- Vanilla JavaScript ES6+
- Under 200 lines
- Fast search (debounced, 150ms delay)
- Client-side only

**Search Implementation:**

Do NOT use Fuse.js or any library. Implement simple substring matching:

```javascript
(function() {
    'use strict';
    
    // Book data - will be embedded or fetched
    let books = [];
    let filteredBooks = [];
    let currentSort = { field: 'title', direction: 'asc' };
    
    // DOM elements
    const searchInput = document.getElementById('book-search');
    const categoryFilter = document.getElementById('filter-category');
    const decadeFilter = document.getElementById('filter-decade');
    const tbody = document.getElementById('books-tbody');
    const showingCount = document.getElementById('showing-count');
    const totalCount = document.getElementById('total-count');
    const headers = document.querySelectorAll('.catalogue-table th.sortable');
    
    // Initialize
    async function init() {
        // Load books data
        // Option 1: Embedded in page
        // books = window.BOOK_DATA;
        
        // Option 2: Fetch from JSON file
        try {
            const response = await fetch('/data/books.json');
            const data = await response.json();
            books = data.books;
            totalCount.textContent = books.length;
            
            // Populate decade filter
            populateDecadeFilter();
            
            // Initial render
            applyFiltersAndSort();
        } catch (err) {
            console.error('Failed to load books:', err);
            tbody.innerHTML = '<tr><td colspan="4" class="no-results">Failed to load book data.</td></tr>';
        }
    }
    
    function populateDecadeFilter() {
        const decades = new Set();
        books.forEach(book => {
            if (book.year) {
                const decade = Math.floor(book.year / 10) * 10;
                decades.add(decade);
            }
        });
        
        const sorted = Array.from(decades).sort((a, b) => a - b);
        sorted.forEach(decade => {
            const option = document.createElement('option');
            option.value = decade;
            option.textContent = `${decade}s`;
            decadeFilter.appendChild(option);
        });
    }
    
    function applyFiltersAndSort() {
        const searchTerm = searchInput.value.toLowerCase().trim();
        const categoryValue = categoryFilter.value;
        const decadeValue = decadeFilter.value;
        
        // Filter
        filteredBooks = books.filter(book => {
            // Search filter
            if (searchTerm) {
                const searchFields = [
                    book.title,
                    book.author,
                    ...(book.keywords || [])
                ].join(' ').toLowerCase();
                
                if (!searchFields.includes(searchTerm)) {
                    return false;
                }
            }
            
            // Category filter
            if (categoryValue && book.category !== categoryValue) {
                return false;
            }
            
            // Decade filter
            if (decadeValue && book.year) {
                const bookDecade = Math.floor(book.year / 10) * 10;
                if (bookDecade !== parseInt(decadeValue)) {
                    return false;
                }
            } else if (decadeValue && !book.year) {
                return false;
            }
            
            return true;
        });
        
        // Sort
        filteredBooks.sort((a, b) => {
            let aVal = a[currentSort.field] || '';
            let bVal = b[currentSort.field] || '';
            
            // Handle author_sort for author field
            if (currentSort.field === 'author') {
                aVal = a.author_sort || a.author || '';
                bVal = b.author_sort || b.author || '';
            }
            
            // Numeric sort for year
            if (currentSort.field === 'year') {
                aVal = a.year || 0;
                bVal = b.year || 0;
                return currentSort.direction === 'asc' ? aVal - bVal : bVal - aVal;
            }
            
            // String sort
            const comparison = String(aVal).localeCompare(String(bVal));
            return currentSort.direction === 'asc' ? comparison : -comparison;
        });
        
        renderBooks();
        updateCount();
        updateSortIndicators();
    }
    
    function renderBooks() {
        if (filteredBooks.length === 0) {
            tbody.innerHTML = '<tr><td colspan="4" class="no-results">No books found matching your search.</td></tr>';
            return;
        }
        
        tbody.innerHTML = filteredBooks.map(book => `
            <tr>
                <td>
                    <div class="book-title">${escapeHtml(book.title)}</div>
                    ${book.category ? `<div class="book-category">${escapeHtml(book.category)}</div>` : ''}
                </td>
                <td class="book-author">${escapeHtml(book.author || '—')}</td>
                <td class="book-year">${book.year_display || '—'}</td>
                <td class="book-links">
                    <a href="${book.links.worldcat}" target="_blank" rel="noopener" title="Search WorldCat">WC</a>
                    <a href="${book.links.internet_archive}" target="_blank" rel="noopener" title="Search Internet Archive">IA</a>
                    <a href="${book.links.annas_archive}" target="_blank" rel="noopener" title="Search Anna's Archive">AA</a>
                    <a href="${book.links.bookfinder}" target="_blank" rel="noopener" title="Find on BookFinder">BF</a>
                </td>
            </tr>
        `).join('');
    }
    
    function updateCount() {
        showingCount.textContent = filteredBooks.length;
    }
    
    function updateSortIndicators() {
        headers.forEach(th => {
            th.classList.remove('sort-asc', 'sort-desc');
            if (th.dataset.sort === currentSort.field) {
                th.classList.add(`sort-${currentSort.direction}`);
            }
        });
    }
    
    function escapeHtml(text) {
        const div = document.createElement('div');
        div.textContent = text;
        return div.innerHTML;
    }
    
    // Debounce helper
    function debounce(fn, delay) {
        let timeout;
        return function(...args) {
            clearTimeout(timeout);
            timeout = setTimeout(() => fn.apply(this, args), delay);
        };
    }
    
    // Event listeners
    searchInput.addEventListener('input', debounce(applyFiltersAndSort, 150));
    categoryFilter.addEventListener('change', applyFiltersAndSort);
    decadeFilter.addEventListener('change', applyFiltersAndSort);
    
    headers.forEach(th => {
        th.addEventListener('click', () => {
            const field = th.dataset.sort;
            if (currentSort.field === field) {
                currentSort.direction = currentSort.direction === 'asc' ? 'desc' : 'asc';
            } else {
                currentSort.field = field;
                currentSort.direction = 'asc';
            }
            applyFiltersAndSort();
        });
    });
    
    // Start
    init();
})();
```

---

## Phase 3: Integration with Hugo

### Option A: Standalone HTML (Recommended)

1. Create `static/belichick-library/index.html` with the full HTML
2. Create `static/data/books.json` with the book data
3. Update `content/belichick.md` to redirect or link to the new page

### Option B: Hugo Shortcode

1. Create a shortcode that embeds the catalogue
2. Store JavaScript inline or in `static/js/`
3. Reference from `content/belichick.md`

### File Placement

```
personal-site/
├── content/
│   └── belichick.md          # Update with link to new page
├── static/
│   ├── belichick-library/
│   │   └── index.html        # The catalogue page
│   └── data/
│       └── books.json        # Book data
└── scripts/
    └── clean_belichick_data.py  # Data processing script
```

---

## Phase 4: Link Quality and User Guidance

### How Links Are Generated

Explain to the user in the page footer or in documentation:

1. **WorldCat**: Searches global library holdings. Best for finding the book in a library near you.
2. **Internet Archive**: May have digitized copies for borrowing or reading online.
3. **Anna's Archive**: Aggregates multiple sources; may have PDFs available.
4. **BookFinder**: Aggregates used book sellers; useful for purchasing rare copies.

### User Verification Process

After generating links, the user should spot-check:

1. **Random sample of 20 books**: Click each link type and verify:
   - Does the search return the correct book?
   - For older books (pre-1950), are there any results?
   - Do abbreviated titles cause search failures?

2. **Edge cases to check**:
   - Books with very common titles ("Football")
   - Books with special characters in title
   - Books with no author
   - Books with very long titles

### Improving Link Quality

**Recommendations for the user:**

1. **Add ISBNs manually** for post-1970 books:
   - Look up on WorldCat or Open Library
   - Add to JSON as `isbn_10` and/or `isbn_13`
   - Update link generators to use ISBN when available:
     ```
     WorldCat: https://search.worldcat.org/search?q=bn:{isbn}
     ```

2. **Add OCLC numbers** for rare books:
   - WorldCat assigns unique identifiers
   - Direct link: `https://search.worldcat.org/title/{oclc_number}`

3. **Manual overrides**:
   - For books that don't search well, add a `links_override` field
   - Allows manually specifying the correct URL

4. **Fallback text search tuning**:
   - If title is very generic, include subtitle
   - If author name is common, include publisher

---

## Deliverables Checklist

1. **`scripts/clean_belichick_data.py`**
   - Parses both CSVs
   - Cleans and normalizes data
   - Generates `books.json`
   - Outputs validation report

2. **`static/data/books.json`**
   - Clean, structured book data
   - All fields populated where available
   - Generated links for each book

3. **`static/belichick-library/index.html`**
   - Self-contained HTML page
   - References existing theme CSS
   - Inline JavaScript (no dependencies)
   - Responsive design

4. **Updated `content/belichick.md`**
   - Link to new interactive catalogue
   - Brief description of the collection

5. **Documentation in README or comments**
   - How to regenerate book data
   - How to verify link quality
   - How to add ISBNs or improve metadata

---

## Design Principles Summary

1. **Minimal dependencies**: Zero JS libraries, use theme CSS
2. **Progressive enhancement**: Page shows data even if JS fails (via `<noscript>` message)
3. **Consistent with site**: Match ezhil theme exactly
4. **Accessible**: Proper ARIA labels, keyboard navigation, focus states
5. **Fast**: No external requests except data fetch, minimal CSS
6. **Maintainable**: Data in JSON, presentation in HTML/CSS/JS
7. **Practical links**: All four link types serve different purposes

---

## Example Final Appearance

When complete, a book entry should look like:

**Desktop:**
```
┌────────────────────────┬─────────────────────┬──────┬─────────────────┐
│ Snake                  │ Ken Stabler with    │ 1986 │ [WC][IA][AA][BF]│
│ Biography              │ Berry Stainback     │      │                 │
└────────────────────────┴─────────────────────┴──────┴─────────────────┘
```

**Mobile:**
```
┌─────────────────────────────────────┐
│ Snake                               │
│ Ken Stabler with Berry Stainback    │
│ 1986 · Biography                    │
│ [WC] [IA] [AA] [BF]                 │
└─────────────────────────────────────┘
```

---

## Execution Order

1. First, run the data cleaning script and review output
2. Spot-check 10-20 generated links manually
3. Build the HTML page
4. Test on desktop and mobile
5. Integrate with Hugo site
6. Deploy and verify

Good luck! This should result in a clean, functional, and maintainable book catalogue.
