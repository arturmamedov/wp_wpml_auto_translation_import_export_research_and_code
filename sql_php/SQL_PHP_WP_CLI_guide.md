# WordPress Weglot to WPML Migration - Complete Implementation Guide

## Project Overview

I need to migrate a WordPress website from Weglot translation system to WPML, implementing a **completely free automated solution** using SQL export + PHP scripts + WPML API. The goal is to create translations for existing Spanish posts without using any paid plugins.

## Technical Specifications

**Current Setup:**
- **Source Site:** https://nestshostels.com (Weglot-powered, Spanish main language)
- **Target Site:** https://wpml.nestshostels.com (WPML-powered, Spanish posts already imported)
- **Content to Migrate:** 60 WordPress posts 
- **Source Language:** Spanish (es)
- **Target Languages:** English (en), German (de), Italian (it), French (fr)
- **SEO Plugin:** Yoast SEO (metadata must be preserved)

**Current Status:**
- WPML site has all 60 Spanish posts already imported as source language
- Need to create 240 translations (60 posts × 4 languages) and connect them via WPML
- Full WordPress admin and hosting access available on target site
- Only front-end access to Weglot translations (no export capability)

## Required Solution: SQL + PHP + WPML API Automation

Create a completely automated, free solution that:
1. **Extracts content** from the Weglot site via SQL/API scraping
2. **Translates content** using AI APIs (DeepL, OpenAI, or similar)
3. **Creates WordPress posts** programmatically using wp_insert_post()
4. **Preserves all Yoast SEO metadata** in translations
5. **Connects translations** using WPML API functions
6. **Finalizes connections** using WP-CLI WPML commands

## Step-by-Step Implementation Required

### Phase 1: Content Extraction Script
Create a PHP script that extracts Spanish post data including:
- Post title, content, excerpt, slug
- All Yoast SEO metadata (_yoast_wpseo_title, _yoast_wpseo_metadesc, _yoast_wpseo_focuskw, etc.)
- Post ID for WPML connection reference
- Featured image and media attachments

### Phase 2: Translation Integration
Implement translation functionality using:
- **DeepL API** (recommended for XML/HTML preservation)
- **OpenAI API** (alternative with good context understanding)
- Content structure preservation for WordPress blocks/shortcodes
- SEO metadata translation while maintaining optimization

### Phase 3: WPML Post Creation
Develop WordPress functions that:
- Create translated posts using wp_insert_post()
- Set all translated Yoast SEO metadata
- Use WPML API hooks for language assignment
- Connect translations using wpml_set_element_language_details

### Phase 4: Automation and Finalization  
- Batch processing for all 60 posts
- WP-CLI integration for final WPML connections
- Error handling and progress tracking
- Verification and quality assurance

## Required Code Architecture

### Main Classes Needed:
1. **ContentExtractor** - Extracts post data from source
2. **TranslationService** - Handles AI translation API calls
3. **WPMLTranslationCreator** - Creates posts and WPML connections
4. **BatchProcessor** - Orchestrates the full migration

### Key WPML Functions to Use:
- `wpml_element_trid` - Get translation group ID
- `wpml_set_element_language_details` - Connect translations
- `wp_insert_post()` - Create translated posts
- `update_post_meta()` - Set SEO metadata
- `wp wpml import process` - Final WP-CLI command

## Critical Technical Requirements

**WPML Database Structure:**
- Posts connected via `icl_translations` table using `trid` (translation group ID)
- Each translation needs proper `element_type`, `language_code`, `source_language_code`
- Original Spanish posts already have WPML language assignments

**Yoast SEO Fields to Preserve:**
- `_yoast_wpseo_title` (SEO title)
- `_yoast_wpseo_metadesc` (meta description)  
- `_yoast_wpseo_focuskw` (focus keyword)
- `_yoast_wpseo_opengraph-title` (social media title)
- `_yoast_wpseo_opengraph-description` (social media description)
- `_yoast_wpseo_twitter-title` and `_yoast_wpseo_twitter-description`

**Translation Style Requirements:**
Based on Nests Hostels brand guide (casual, Gen-Z/Millennial friendly):
- Use "tú" form in Spanish/Italian/French, "du" in German
- Casual, conversational tone
- Travel/hospitality context awareness
- Preserve brand personality and local expressions

## Expected Deliverables

1. **Complete PHP script** (can be multiple files/classes)
2. **WP-CLI integration** for execution and finalization
3. **Error handling** and logging functionality
4. **Progress tracking** for batch processing
5. **Verification queries** to confirm successful migration
6. **Documentation** for execution and troubleshooting

## Implementation Timeline

- **Phase 1:** Content extraction (1 day)
- **Phase 2:** Translation integration (1 day)
- **Phase 3:** WPML post creation (1 day)  
- **Phase 4:** Testing and batch execution (1 day)
- **Phase 5:** Verification and optimization (1 day)

## Technical Constraints

- **No paid plugins** - Must use free WordPress/WPML functionality only
- **API rate limits** - Handle translation API throttling appropriately
- **Memory management** - Process 60 posts efficiently without timeouts
- **SEO preservation** - All metadata must transfer accurately
- **WPML compliance** - Follow official WPML API best practices

## Success Criteria

After migration completion:
- 60 Spanish posts remain as source language in WPML
- 240 new translated posts created (4 languages × 60 posts)
- All translations properly connected in WPML translation groups
- All Yoast SEO metadata preserved and translated appropriately
- Language switcher shows all 5 languages for each post
- No duplicate content or broken internal links
- Site performance maintained (no significant slowdown)

Please provide a complete, working solution that automates this entire migration process using the technical approach outlined above. Include all necessary code, detailed implementation steps, and execution instructions.
