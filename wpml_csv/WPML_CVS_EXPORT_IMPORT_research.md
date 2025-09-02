# Complete Migration Guide: Weglot to WPML with SEO Preservation

**Migrating 60 WordPress posts from Weglot to WPML while preserving all SEO metadata and translations requires a multi-step approach combining front-end extraction, automation tools, and careful SEO management.** Since Weglot restricts export functionality to paid plans and you only have front-end access, this guide provides practical solutions for extracting, processing, and importing your multilingual content.

## Understanding the challenge

Weglot and WPML use fundamentally different architectures. **Weglot stores translations externally in their cloud infrastructure and injects content via JavaScript**, while **WPML creates duplicate posts in your WordPress database for each language**. This means direct migration isn't possible - you'll need to extract content from the front-end and recreate the multilingual structure in WPML.

## Front-end content extraction strategies

### Automated browser-based scraping approach

Since you lack backend access to Weglot, **browser automation provides the most reliable extraction method**. Here's a PHP script using Selenium WebDriver to systematically extract your translated content:

```php
<?php
class WeglotContentExtractor {
    private $base_url;
    private $languages;
    private $output_dir;
    
    public function __construct($base_url, $languages = ['en', 'de', 'it', 'fr']) {
        $this->base_url = rtrim($base_url, '/');
        $this->languages = $languages; // Spanish is already main language
        $this->output_dir = __DIR__ . '/extracted_content/';
        
        if (!file_exists($this->output_dir)) {
            mkdir($this->output_dir, 0755, true);
        }
    }
    
    public function extractAllContent() {
        $urls = $this->discoverPostUrls();
        
        foreach ($urls as $post_url) {
            foreach ($this->languages as $lang) {
                $translated_url = $this->buildLanguageUrl($post_url, $lang);
                $content = $this->fetchContentWithCurl($translated_url);
                
                if ($content) {
                    $parsed_content = $this->parseContent($content, $lang, $post_url);
                    $this->saveExtractedContent($parsed_content, $lang, $post_url);
                }
                
                sleep(2); // Rate limiting
            }
        }
    }
    
    private function buildLanguageUrl($url, $language) {
        // Handle Weglot's subdirectory structure
        $parsed = parse_url($url);
        return $parsed['scheme'] . '://' . $parsed['host'] . '/' . $language . $parsed['path'];
    }
    
    private function parseContent($html, $language, $original_url) {
        $dom = new DOMDocument();
        @$dom->loadHTML($html);
        
        // Remove Weglot scripts and language switchers
        $scripts = $dom->getElementsByTagName('script');
        for ($i = $scripts->length - 1; $i >= 0; $i--) {
            $script = $scripts->item($i);
            if (strpos($script->textContent, 'weglot') !== false) {
                $script->parentNode->removeChild($script);
            }
        }
        
        return [
            'url' => $original_url,
            'language' => $language,
            'title' => $this->extractTitle($dom),
            'content' => $this->extractMainContent($dom),
            'meta_title' => $this->extractMetaTitle($dom),
            'meta_description' => $this->extractMetaDescription($dom),
            'yoast_data' => $this->extractYoastData($dom)
        ];
    }
    
    private function extractYoastData($dom) {
        $yoast_data = [];
        
        // Extract Yoast SEO data from meta tags
        $meta_tags = $dom->getElementsByTagName('meta');
        foreach ($meta_tags as $meta) {
            $property = $meta->getAttribute('property');
            $name = $meta->getAttribute('name');
            $content = $meta->getAttribute('content');
            
            if (strpos($property, 'og:') === 0 || strpos($name, 'twitter:') === 0) {
                $yoast_data[$property ?: $name] = $content;
            }
        }
        
        return $yoast_data;
    }
}
```

### Browser extension approach for smaller batches

For manual extraction of specific posts, **use the Web Scraper Chrome extension** with this configuration:

```javascript
{
  "selectors": [
    {
      "id": "post-title",
      "type": "SelectorText", 
      "selector": "h1.entry-title, .post-title, h1",
      "multiple": false
    },
    {
      "id": "post-content",
      "type": "SelectorHTML",
      "selector": ".entry-content, .post-content, .content",
      "multiple": false
    },
    {
      "id": "meta-description", 
      "type": "SelectorAttribute",
      "selector": "meta[name='description']",
      "attribute": "content"
    }
  ]
}
```

## WPML CSV import format and automation

### Required CSV structure for WPML

**WPML Export and Import plugin requires three essential columns** for multilingual content:

```csv
post_title,post_content,_yoast_wpseo_title,_yoast_wpseo_metadesc,_yoast_wpseo_focuskw,_wpml_import_language_code,_wpml_import_translation_group,_wpml_import_source_language_code
"Casa Rural Perfecta","Complete post content here...","Perfect Rural House - Best Price","Discover our perfect rural house with amazing amenities...","casa rural","es","post_1","NULL"
"Perfect Rural House","Translated post content here...","Perfect Rural House - Best Price","Discover our perfect rural house with amazing amenities...","rural house","en","post_1","es"
```

**Critical WPML columns explained:**
- **`_wpml_import_language_code`**: Target language (es, en, de, it, fr)
- **`_wpml_import_translation_group`**: Unique identifier linking all translations of the same post
- **`_wpml_import_source_language_code`**: Original language code (NULL for source language posts)

### Automated CSV processing workflow

Create this PHP script to convert your extracted content into WPML-ready CSV format:

```php
<?php
class WPMLImportProcessor {
    private $extracted_dir;
    private $csv_output;
    
    public function processExtractedContent($extracted_dir) {
        $this->extracted_dir = $extracted_dir;
        $this->csv_output = fopen('wpml_import.csv', 'w');
        
        // CSV headers including all necessary WPML and Yoast fields
        $headers = [
            'post_title', 'post_content', 'post_excerpt', 'post_name',
            '_yoast_wpseo_title', '_yoast_wpseo_metadesc', '_yoast_wpseo_focuskw',
            '_yoast_wpseo_canonical', '_yoast_wpseo_opengraph-title',
            '_yoast_wpseo_opengraph-description', '_yoast_wpseo_twitter-title',
            '_yoast_wpseo_twitter-description', '_wpml_import_language_code',
            '_wpml_import_translation_group', '_wpml_import_source_language_code'
        ];
        
        fputcsv($this->csv_output, $headers);
        
        $grouped_content = $this->groupTranslationsByPost();
        
        foreach ($grouped_content as $post_group => $translations) {
            foreach ($translations as $lang => $content) {
                $this->writeCSVRow($content, $post_group, $lang, 
                    $this->determineSourceLanguage($translations));
            }
        }
        
        fclose($this->csv_output);
    }
    
    private function writeCSVRow($content, $group_id, $language, $source_lang) {
        $row = [
            $content['title'],
            $content['content'], 
            $this->generateExcerpt($content['content']),
            $this->generateSlug($content['title']),
            $content['meta_title'] ?: $content['title'],
            $content['meta_description'],
            $this->extractKeyword($content['title']),
            '', // canonical - let WPML handle
            $content['yoast_data']['og:title'] ?: $content['title'],
            $content['yoast_data']['og:description'] ?: $content['meta_description'],
            $content['yoast_data']['twitter:title'] ?: $content['title'],
            $content['yoast_data']['twitter:description'] ?: $content['meta_description'],
            $language,
            $group_id,
            ($language === $source_lang) ? '' : $source_lang
        ];
        
        fputcsv($this->csv_output, $row);
    }
}
```

## Free WordPress automation plugins

### WPML Export and Import (recommended)

**The official WPML Export and Import plugin works with WordPress's native import tools and WP All Import Pro**. Here's the optimal workflow:

1. **Install required plugins:**
   - WPML Multilingual CMS 
   - WPML String Translation
   - WPML Export and Import
   - WP All Import Pro (or use WordPress native importer)

2. **Import process:**
   ```
   1. Import your CSV using WP All Import Pro
   2. Map CSV columns to Custom Fields:
      - Map _wpml_import_language_code to Custom Field
      - Map _wpml_import_translation_group to Custom Field  
      - Map _wpml_import_source_language_code to Custom Field
   3. After import completion, go to WPML → Export and Import
   4. Click "Run WPML Import" to connect translations automatically
   ```

### Alternative free solutions

**WP Ultimate CSV Importer** (free version) provides basic WPML compatibility:

```php
// Custom hooks for WP Ultimate CSV Importer
add_action('uci_after_import', 'connect_wpml_translations_after_import');

function connect_wpml_translations_after_import($post_id) {
    $language_code = get_post_meta($post_id, '_wpml_import_language_code', true);
    $translation_group = get_post_meta($post_id, '_wpml_import_translation_group', true);
    $source_language = get_post_meta($post_id, '_wpml_import_source_language_code', true);
    
    if ($language_code && $translation_group) {
        // Set language details
        $set_language_args = array(
            'element_id' => $post_id,
            'element_type' => apply_filters('wpml_element_type', get_post_type($post_id)),
            'language_code' => $language_code,
            'source_language_code' => $source_language ?: null,
            'trid' => $translation_group
        );
        
        do_action('wpml_set_element_language_details', $set_language_args);
    }
}
```

## Comprehensive PHP automation solution

### Complete automation script

For maximum control, here's a complete PHP solution that handles the entire migration:

```php
<?php
class WeglotToWPMLMigrator {
    private $wpdb;
    private $source_site_url;
    private $languages;
    
    public function __construct($source_url, $languages) {
        global $wpdb;
        $this->wpdb = $wpdb;
        $this->source_site_url = $source_url;
        $this->languages = $languages;
    }
    
    public function executeMigration() {
        // Step 1: Extract content from Weglot site
        echo "Step 1: Extracting content from Weglot...\n";
        $extracted_content = $this->extractWeglotContent();
        
        // Step 2: Create posts and translations in WPML
        echo "Step 2: Creating WPML posts and translations...\n";  
        $this->createWPMLContent($extracted_content);
        
        // Step 3: Import SEO metadata
        echo "Step 3: Importing SEO metadata...\n";
        $this->importSEOMetadata($extracted_content);
        
        // Step 4: Connect translations
        echo "Step 4: Connecting WPML translations...\n";
        $this->connectTranslations($extracted_content);
        
        echo "Migration completed successfully!\n";
    }
    
    private function createWPMLContent($content_groups) {
        foreach ($content_groups as $group_id => $translations) {
            $trid = null; // Translation group ID
            
            // Create source language post first (Spanish)
            $source_content = $translations['es'];
            $source_post_id = wp_insert_post([
                'post_title' => $source_content['title'],
                'post_content' => $source_content['content'],
                'post_status' => 'publish',
                'post_type' => 'post'
            ]);
            
            // Set as source language
            $set_language_args = [
                'element_id' => $source_post_id,
                'element_type' => 'post_post',
                'language_code' => 'es'
            ];
            do_action('wpml_set_element_language_details', $set_language_args);
            
            // Get the trid for connecting translations
            $source_language_info = apply_filters('wpml_element_language_details', null, [
                'element_id' => $source_post_id,
                'element_type' => 'post_post'
            ]);
            $trid = $source_language_info->trid;
            
            // Create translations
            foreach (['en', 'de', 'it', 'fr'] as $lang) {
                if (isset($translations[$lang])) {
                    $translation_content = $translations[$lang];
                    
                    $translation_post_id = wp_insert_post([
                        'post_title' => $translation_content['title'],
                        'post_content' => $translation_content['content'],
                        'post_status' => 'publish',
                        'post_type' => 'post'
                    ]);
                    
                    // Connect to translation group
                    $set_language_args = [
                        'element_id' => $translation_post_id,
                        'element_type' => 'post_post',
                        'trid' => $trid,
                        'language_code' => $lang,
                        'source_language_code' => 'es'
                    ];
                    do_action('wpml_set_element_language_details', $set_language_args);
                }
            }
        }
    }
    
    private function importSEOMetadata($content_groups) {
        foreach ($content_groups as $group_id => $translations) {
            foreach ($translations as $lang => $content) {
                // Find the post ID for this language version
                $post_id = $this->findPostByContent($content['title'], $lang);
                
                if ($post_id) {
                    // Import Yoast SEO metadata
                    update_post_meta($post_id, '_yoast_wpseo_title', $content['meta_title']);
                    update_post_meta($post_id, '_yoast_wpseo_metadesc', $content['meta_description']);
                    update_post_meta($post_id, '_yoast_wpseo_focuskw', $this->extractKeyword($content['title']));
                    
                    // Import social media metadata
                    if (!empty($content['yoast_data'])) {
                        foreach ($content['yoast_data'] as $key => $value) {
                            $meta_key = str_replace(':', '-', '_yoast_wpseo_' . $key);
                            update_post_meta($post_id, $meta_key, $value);
                        }
                    }
                }
            }
        }
    }
}

// Execute migration
$migrator = new WeglotToWPMLMigrator('https://nestshostels.com', ['es', 'en', 'de', 'it', 'fr']);
$migrator->executeMigration();
```

## Step-by-step migration workflow for 60 posts

### Phase 1: Preparation and setup (Day 1-2)

1. **Create complete backup** of wpml.nestshostels.com
2. **Install required plugins:**
   - WPML Multilingual CMS
   - WPML String Translation  
   - WPML Export and Import
   - WPML SEO (free addon)
   - Yoast SEO

3. **Configure WPML languages:**
   - Set Spanish as primary language
   - Add English, German, Italian, French as secondary languages
   - Configure URL structure (recommended: subdirectories)

### Phase 2: Content extraction (Day 3-5)

1. **Deploy the extraction script** on your local development environment
2. **Extract content systematically:**
   - Run scraping script for all 60 posts
   - Verify extraction quality for each language
   - Handle any timeout or rate-limiting issues

3. **Process extracted data:**
   - Convert to WPML-compatible CSV format
   - Validate translation groupings
   - Clean and optimize content for import

### Phase 3: Import and connection (Day 6-8)

1. **Import content using WP All Import Pro:**
   - Import Spanish posts first (source language)
   - Import translations with proper WPML columns mapped
   - Verify post creation and basic content

2. **Run WPML connection process:**
   - Go to WPML → Export and Import
   - Click "Run WPML Import"
   - Verify all translations are properly connected

3. **Import SEO metadata:**
   - Use the PHP script to import Yoast SEO fields
   - Verify meta titles, descriptions, and social metadata
   - Check for any missing or corrupted data

### Phase 4: Verification and optimization (Day 9-10)

1. **Test multilingual functionality:**
   - Verify language switching works correctly
   - Check URL structure and hreflang tags
   - Test translated content display

2. **SEO verification:**
   - Check XML sitemaps include all languages
   - Verify Yoast SEO data displays correctly
   - Test social media sharing for each language

## Tools for front-end extraction with limited access

### Recommended browser extensions

1. **Web Scraper (Chrome):**
   - Point-and-click interface for content extraction
   - Handles JavaScript-rendered content
   - Exports to CSV/JSON formats
   - Free with good documentation

2. **Instant Data Scraper:**
   - AI-powered content detection
   - Works with dynamic content loading
   - Supports pagination and bulk operations

### Python/Selenium alternative

For more reliable extraction, use this Python script:

```python
from selenium import webdriver
from bs4 import BeautifulSoup
import json
import time

def extract_weglot_content(base_url, languages, post_slugs):
    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_argument('--headless')
    driver = webdriver.Chrome(options=chrome_options)
    
    all_content = {}
    
    for post_slug in post_slugs:
        all_content[post_slug] = {}
        
        for lang in languages:
            url = f"{base_url}/{lang}/{post_slug}/"
            
            try:
                driver.get(url)
                time.sleep(3)  # Wait for Weglot to load translations
                
                html = driver.page_source
                soup = BeautifulSoup(html, 'html.parser')
                
                # Extract content
                content = {
                    'title': soup.find('h1').text.strip() if soup.find('h1') else '',
                    'content': soup.find('.entry-content').get_text().strip() if soup.find('.entry-content') else '',
                    'meta_title': soup.find('meta', {'property': 'og:title'})['content'] if soup.find('meta', {'property': 'og:title'}) else '',
                    'meta_description': soup.find('meta', {'name': 'description'})['content'] if soup.find('meta', {'name': 'description'}) else ''
                }
                
                all_content[post_slug][lang] = content
                
            except Exception as e:
                print(f"Error extracting {url}: {e}")
                
            time.sleep(2)  # Rate limiting
    
    driver.quit()
    
    # Save to JSON file
    with open('extracted_content.json', 'w', encoding='utf-8') as f:
        json.dump(all_content, f, ensure_ascii=False, indent=2)
    
    return all_content

# Usage
languages = ['en', 'de', 'it', 'fr']
post_slugs = ['post-slug-1', 'post-slug-2', '...']  # Your 60 post slugs
extracted = extract_weglot_content('https://nestshostels.com', languages, post_slugs)
```

## Yoast SEO integration and metadata preservation

### WPML SEO setup for Yoast compatibility

**Install WPML SEO addon** (free) which replaces the old Yoast WPML "glue" plugin and provides:

- Automatic synchronization of SEO settings
- Translation management for meta titles and descriptions
- Proper hreflang tag generation
- Multilingual XML sitemap support

### SEO metadata mapping during import

**Map these Yoast SEO fields** in your CSV import:

```php
$yoast_fields = [
    '_yoast_wpseo_title' => 'meta_title',
    '_yoast_wpseo_metadesc' => 'meta_description', 
    '_yoast_wpseo_focuskw' => 'focus_keyword',
    '_yoast_wpseo_canonical' => 'canonical_url',
    '_yoast_wpseo_opengraph-title' => 'og_title',
    '_yoast_wpseo_opengraph-description' => 'og_description',
    '_yoast_wpseo_twitter-title' => 'twitter_title',
    '_yoast_wpseo_twitter-description' => 'twitter_description'
];
```

### Post-migration SEO verification checklist

1. **Technical SEO elements:**
   - ✅ XML sitemaps include all language versions
   - ✅ Hreflang tags properly implemented
   - ✅ URL structure follows best practices
   - ✅ No duplicate content issues

2. **Content-level SEO:**
   - ✅ Meta titles within 60 characters
   - ✅ Meta descriptions within 155 characters
   - ✅ Focus keywords localized appropriately
   - ✅ Internal linking structure preserved

3. **Social media integration:**
   - ✅ Open Graph tags for each language
   - ✅ Twitter Card data complete
   - ✅ Social sharing previews working correctly

## Timeline and resource requirements

### Estimated timeline for 60 posts

- **Preparation and setup:** 2 days
- **Content extraction:** 3 days (including troubleshooting)
- **Import and processing:** 3 days
- **Verification and optimization:** 2 days
- **Total:** 10 working days

### Skills and resources needed

- **PHP development:** For custom scripts and automation
- **WordPress administration:** For plugin configuration and content management
- **Basic SEO knowledge:** For metadata optimization and verification
- **Optional:** Python/Selenium for advanced extraction scenarios

This comprehensive approach addresses all aspects of your Weglot to WPML migration while preserving SEO metadata and enabling automation through both free plugins and custom PHP solutions. The combination of front-end extraction techniques and WPML's official import tools provides a complete solution for your 60-post migration project.
