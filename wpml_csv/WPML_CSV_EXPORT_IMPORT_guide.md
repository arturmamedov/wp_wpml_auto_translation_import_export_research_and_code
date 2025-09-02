# WordPress CSV Translation Import for WPML Migration

## Project Overview

I need to migrate a WordPress website from Weglot to WPML using the **CSV import approach**. This involves creating properly formatted CSV files with WPML metadata, translating content via AI APIs, and importing through WordPress import tools. 

**⚠️ CRITICAL CONSTRAINT:** This approach requires paid plugins for optimal implementation. WP All Import Pro ($99) + WPML Export and Import are the standard solution. Free alternatives require significant custom development.

## Technical Specifications

**Current Setup:**
- **Source Site:** https://nestshostels.com (Weglot-powered, Spanish main language)
- **Target Site:** https://wpml.nestshostels.com (WPML-powered, Spanish posts already imported)
- **Content Volume:** 60 WordPress posts with complete Yoast SEO metadata
- **Source Language:** Spanish (es) - already imported in WPML site
- **Target Languages:** English (en), German (de), Italian (it), French (fr)
- **WordPress Access:** Full admin and hosting access to target site

## Required CSV Structure for WPML Import

WPML Export and Import requires specific column structure for multilingual content:

```csv
post_title,post_content,post_excerpt,post_name,_yoast_wpseo_title,_yoast_wpseo_metadesc,_yoast_wpseo_focuskw,_yoast_wpseo_opengraph-title,_yoast_wpseo_opengraph-description,_yoast_wpseo_twitter-title,_yoast_wpseo_twitter-description,_wpml_import_language_code,_wpml_import_source_language_code,_wpml_import_translation_group

"Perfect Rural House","Complete English post content with HTML...","English excerpt","perfect-rural-house","Perfect Rural House - Best Price","Discover our perfect rural house with amazing amenities...","rural house","Perfect Rural House - Social","Discover our perfect rural house...","Perfect Rural House - Twitter","Discover our perfect rural house...","en","es","post_1"

"Perfektes Landhaus","Complete German post content with HTML...","German excerpt","perfektes-landhaus","Perfektes Landhaus - Bester Preis","Entdecken Sie unser perfektes Landhaus...","landhaus","Perfektes Landhaus - Social","Entdecken Sie unser perfektes...","Perfektes Landhaus - Twitter","Entdecken Sie unser perfektes...","de","es","post_1"
```

### Essential WPML Columns:
- **`_wpml_import_language_code`** - Target language (en, de, it, fr)
- **`_wpml_import_source_language_code`** - Original language (es for all translations)
- **`_wpml_import_translation_group`** - Unique identifier linking all translations (use original post ID or custom identifier)

### Yoast SEO Columns Required:
- **`_yoast_wpseo_title`** - SEO page title
- **`_yoast_wpseo_metadesc`** - Meta description  
- **`_yoast_wpseo_focuskw`** - Focus keyword (localized)
- **`_yoast_wpseo_opengraph-title`** - Social media title
- **`_yoast_wpseo_opengraph-description`** - Social media description
- **`_yoast_wpseo_twitter-title`** - Twitter card title
- **`_yoast_wpseo_twitter-description`** - Twitter card description

## Implementation Options

### Option A: Paid Plugin Solution (Recommended)

**Required Plugins:**
- **WP All Import Pro** ($99) - Professional CSV import with advanced mapping
- **WPML Export and Import** (free add-on) - Connects imported translations
- **WooCommerce Import Add-On Pro** (if needed for enhanced features)

**Implementation Process:**
1. Export Spanish posts to understand data structure
2. Create translation CSV files with proper WPML metadata
3. Import via WP All Import Pro with field mapping
4. Run WPML Export and Import to connect translations

### Option B: Free Custom Solution (Complex Development)

**Required Development:**
- Custom CSV parser and WordPress import functions
- Manual WPML database table manipulation
- Custom field mapping and relationship management
- Extensive error handling and validation

**Technical Challenges:**
- WordPress native CSV importer lacks advanced field mapping
- Manual WPML `icl_translations` table management
- Complex relationship handling for translation groups
- No built-in validation or rollback capabilities

## Required Solution Architecture

### Phase 1: Spanish Content Extraction
Create system to extract existing Spanish posts:
```sql
-- Extract Spanish posts with all metadata
SELECT 
    p.ID,
    p.post_title,
    p.post_content,
    p.post_excerpt,
    p.post_name,
    (SELECT meta_value FROM wp_postmeta WHERE post_id = p.ID AND meta_key = '_yoast_wpseo_title') as seo_title,
    (SELECT meta_value FROM wp_postmeta WHERE post_id = p.ID AND meta_key = '_yoast_wpseo_metadesc') as seo_desc,
    (SELECT meta_value FROM wp_postmeta WHERE post_id = p.ID AND meta_key = '_yoast_wpseo_focuskw') as focus_kw
FROM wp_posts p 
WHERE p.post_type = 'post' 
AND p.post_status = 'publish'
ORDER BY p.ID;
```

### Phase 2: AI Translation Integration
Implement translation service with CSV-safe output:

```php
class CSVTranslationProcessor {
    private $translation_api; // DeepL, OpenAI, etc.
    
    public function translatePostsToCSV($spanish_posts, $target_language) {
        $csv_rows = [];
        
        foreach ($spanish_posts as $post) {
            $translated_row = [
                'post_title' => $this->translateText($post['post_title'], $target_language),
                'post_content' => $this->translateHTML($post['post_content'], $target_language),
                'post_excerpt' => $this->translateText($post['post_excerpt'], $target_language),
                'post_name' => $this->generateSlug($post['post_title'], $target_language),
                '_yoast_wpseo_title' => $this->translateText($post['seo_title'], $target_language),
                '_yoast_wpseo_metadesc' => $this->translateText($post['seo_desc'], $target_language),
                '_yoast_wpseo_focuskw' => $this->translateKeyword($post['focus_kw'], $target_language),
                '_wpml_import_language_code' => $target_language,
                '_wpml_import_source_language_code' => 'es',
                '_wpml_import_translation_group' => 'post_' . $post['ID']
            ];
            
            $csv_rows[] = $translated_row;
        }
        
        return $this->generateCSV($csv_rows, $target_language);
    }
    
    private function translateHTML($content, $target_lang) {
        // Handle HTML/WordPress blocks safely
        // Preserve shortcodes and formatting
        // Use translation API with HTML support
    }
}
```

### Phase 3: CSV Generation and Validation
Create properly formatted CSV files for each language:

```php
class WPMLCSVGenerator {
    public function generateLanguageCSV($posts_data, $language) {
        $csv_file = "wpml_import_{$language}.csv";
        $handle = fopen($csv_file, 'w');
        
        // Write CSV headers
        $headers = [
            'post_title', 'post_content', 'post_excerpt', 'post_name',
            '_yoast_wpseo_title', '_yoast_wpseo_metadesc', '_yoast_wpseo_focuskw',
            '_yoast_wpseo_opengraph-title', '_yoast_wpseo_opengraph-description',
            '_yoast_wpseo_twitter-title', '_yoast_wpseo_twitter-description',
            '_wpml_import_language_code', '_wpml_import_source_language_code',
            '_wpml_import_translation_group'
        ];
        fputcsv($handle, $headers);
        
        // Write data rows with proper CSV escaping
        foreach ($posts_data as $post) {
            fputcsv($handle, $this->prepareCSVRow($post));
        }
        
        fclose($handle);
        return $csv_file;
    }
    
    private function prepareCSVRow($post_data) {
        // Handle HTML content in CSV
        // Escape special characters properly
        // Ensure WPML metadata integrity
    }
}
```

### Phase 4: Import Execution

**With WP All Import Pro:**
1. Upload CSV file through WP All Import interface
2. Map CSV columns to WordPress fields and custom fields
3. Configure WPML-specific field mappings
4. Execute import with validation
5. Run `wp wpml import process` to connect translations

**With Custom Solution:**
1. Parse CSV files with PHP
2. Create posts using `wp_insert_post()`
3. Set metadata using `update_post_meta()`
4. Manually insert WPML translation relationships
5. Validate and verify all connections

## Translation Style Requirements

**Brand Context:** Nests Hostels - Casual, Gen-Z/Millennial travel brand

**CSV-Specific Considerations:**
- Handle CSV special characters (commas, quotes, newlines)
- Preserve HTML formatting in content fields
- Maintain WordPress shortcode integrity
- Ensure SEO optimization in translated metadata

**Language Adaptations:**
- **English:** Conversational travel tone, "you" form
- **German:** Casual "du" form, German travel expressions  
- **Italian:** Friendly "tu" form, Italian hospitality warmth
- **French:** Casual "tu" form, French travel sophistication

## Critical Technical Challenges

### 1. CSV Content Formatting
Handle complex content safely:
```php
// Proper CSV content escaping
private function escapeCSVContent($content) {
    // Handle HTML content with embedded quotes/commas
    // Preserve WordPress blocks and shortcodes
    // Maintain line breaks and formatting
    return addcslashes($content, '"');
}
```

### 2. Translation Group Management
Ensure proper WPML connections:
```csv
# Original post remains unchanged
"Spanish Title","Spanish content","es","","post_123"

# All translations reference same group
"English Title","English content","en","es","post_123"
"German Title","German content","de","es","post_123"
"Italian Title","Italian content","it","es","post_123"
"French Title","French content","fr","es","post_123"
```

### 3. Import Sequence and Validation
Critical import order:
1. **Import taxonomies first** (categories, tags)
2. **Import posts by language** (or all together with proper mapping)
3. **Verify WPML metadata** in database
4. **Execute WPML connection process**
5. **Validate translation relationships**

## Implementation Comparison

### Paid Plugin Approach
**Pros:**
- Reliable, tested import process
- Advanced field mapping capabilities
- Built-in validation and error handling
- Official WPML integration
- Professional support available

**Cons:**
- Requires WP All Import Pro ($99)
- Ongoing licensing costs
- Dependency on third-party plugins

### Free Custom Approach  
**Pros:**
- No licensing costs
- Full control over import process
- Customizable to specific needs
- No plugin dependencies

**Cons:**
- Extensive development required
- Manual WPML database manipulation
- Complex error handling needs
- No professional support
- Higher risk of data corruption

## Expected Deliverables

**For Paid Plugin Solution:**
1. Spanish content extraction scripts
2. AI translation integration system
3. CSV generation with proper WPML formatting
4. WP All Import Pro configuration guides
5. Import validation and verification tools

**For Custom Free Solution:**
1. Complete CSV parser and import system
2. WPML database manipulation functions  
3. Translation API integration
4. Error handling and rollback capabilities
5. Comprehensive testing and validation suite

## Success Criteria

After migration completion:
- **60 Spanish posts** maintained as source language
- **240 translated posts** successfully imported (60 × 4 languages)
- **All translations properly connected** in WPML translation groups
- **Yoast SEO metadata** preserved and accurately translated
- **Language switcher functionality** working across all content
- **No data loss or corruption** during import process
- **Site performance maintained** with proper optimization

## Recommendation

**For Production Use:** The paid plugin approach (WP All Import Pro + WPML Export and Import) is strongly recommended for reliability, support, and reduced development time.

**For Budget-Constrained Projects:** The custom free solution is possible but requires significant PHP development expertise and thorough testing.

Please specify which approach you prefer, and I'll provide the complete implementation details for your chosen method.
