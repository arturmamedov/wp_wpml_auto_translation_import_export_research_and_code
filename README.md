### WordPress Weglot to WPML Migration Repository

# WPML XLIFF Auto AI Translator with Brand Voice
https://github.com/arturmamedov/wp_wpml_ai_xliff_translator

After this research a plugin was developed with success and a medium/big website with blog and custom posts/ACF was translated automatically using OpenAI 4o mini model with realy minimal costs.

A comprehensive collection of automated solutions for migrating WordPress multilingual websites from Weglot to WPML, preserving SEO metadata and translation relationships.

## Problem Statement

Migrating from Weglot to WPML presents several technical challenges:

- **Weglot limitations**: No content export functionality without premium plans
- **Different architectures**: Weglot (cloud-based) vs WPML (database-native)
- **SEO preservation**: Maintaining Yoast SEO metadata across languages
- **Translation relationships**: Connecting content across 5 languages
- **Cost constraints**: Need for free/budget-friendly solutions

## Project Specifications

**Target Migration:**
- **Source**: Weglot-powered WordPress site (Spanish primary language)
- **Destination**: WPML-powered WordPress site  
- **Content Volume**: 60 WordPress posts
- **Languages**: Spanish (source) → English, German, Italian, French
- **SEO Plugin**: Yoast SEO (metadata preservation required)
- **Total Output**: 300 posts (60 × 5 languages)

## Solution Approaches

This repository contains four distinct migration methodologies, each with different technical requirements, costs, and complexity levels. The XLIFF approach specifically addresses brand voice preservation for marketing-focused content.

### 1. SQL/PHP + WPML API Approach 🆓

**Status**: Completely Free | **Complexity**: Medium | **Reliability**: High

Direct database manipulation combined with WordPress native functions and WPML API integration.

**Architecture:**
- SQL export from source site for content extraction
- PHP scripts using `wp_insert_post()` and WPML hooks
- AI translation API integration (DeepL/OpenAI)
- WP-CLI commands for finalizing connections

**Key Components:**
- `ContentExtractor` - SQL-based post data extraction
- `TranslationService` - AI API integration with rate limiting
- `WPMLTranslationCreator` - Post creation with translation relationships
- `BatchProcessor` - Orchestrates full migration workflow

**Advantages:**
- No paid plugins required
- Uses official WordPress and WPML APIs
- Full control over migration process
- Reliable translation relationship management

**Ideal For**: Projects with PHP development skills and budget constraints

### 2. WXR/XML Translation Approach 🆓

**Status**: Completely Free | **Complexity**: High | **Reliability**: Medium

WordPress WXR (eXtended RSS) file manipulation with AI-powered translation while preserving XML structure.

**Architecture:**
- WXR export from WordPress (Tools → Export)
- XML parsing with CDATA section handling
- AI translation with structure preservation
- Multi-language WXR generation with WPML metadata
- Sequential import using WordPress native importer

**Key Components:**
- `WXRParser` - Parse existing Spanish WXR files
- `AITranslator` - XML-aware translation processing
- `WXRGenerator` - Multi-language WXR file creation
- `WPMLMetadataManager` - Translation group assignment
- `ImportOrchestrator` - Import sequence coordination

**Advantages:**
- Uses WordPress standard export/import format
- No external plugin dependencies
- Maintains complete content structure
- Supports complex content with HTML/shortcodes

**Challenges:**
- Complex XML manipulation requirements
- CDATA section translation complexity
- Post ID management considerations
- Requires careful error handling

**Ideal For**: Projects requiring XML expertise and maximum control over content structure

### 3. CSV Import Approach 💰/🆓

**Status**: Paid Plugin Recommended | Free Alternative Available | **Complexity**: Low-Medium | **Reliability**: High (paid) / Medium (free)

Structured CSV creation with WPML import metadata for systematic content migration.

**Architecture:**
- Spanish content extraction to CSV format
- AI translation with CSV-safe formatting
- WPML metadata column management
- Import via WP All Import Pro (recommended) or custom solution

**Required CSV Structure:**
```csv
post_title,post_content,_yoast_wpseo_title,_yoast_wpseo_metadesc,_wpml_import_language_code,_wpml_import_source_language_code,_wpml_import_translation_group
```

**Implementation Options:**

**Option A: Paid Plugin Solution ($99)**
- WP All Import Pro + WPML Export and Import
- Professional field mapping and validation
- Reliable import process with error handling
- Official WPML integration support

**Option B: Free Custom Solution**
- Custom CSV parser with WordPress import functions
- Manual WPML database table manipulation
- Extensive custom error handling required

**Advantages (Paid):**
- User-friendly interface
- Advanced field mapping capabilities  
- Professional support and documentation
- Proven reliability for large migrations

**Advantages (Free):**
- No licensing costs
- Customizable import logic
- Full process control

**Ideal For**: Projects with budget for professional tools (paid) or extensive PHP development resources (free)

### 4. XLIFF Translation Workflow Approach 🆓

**Status**: Completely Free | **Complexity**: Medium-High | **Reliability**: High

Professional translation workflow using XLIFF (XML Localization Interchange File Format) files with AI-powered translation and brand voice preservation.

**Architecture:**
- WPML XLIFF export from existing translated content structure
- XLIFF parsing with translation unit extraction
- AI translation with brand voice consistency and tone preservation
- XLIFF reconstruction with translated content
- WPML XLIFF import with automatic translation relationship management

**Key Components:**
- `XLIFFParser` - Professional XLIFF file parsing and structure analysis
- `BrandVoiceTranslator` - AI translation with style guide compliance
- `TranslationMemoryManager` - Consistency across similar content segments
- `XLIFFRebuilder` - Reconstructed XLIFF file generation
- `WPMLXLIFFImporter` - Automated import with translation connection verification

**Advantages:**
- Industry-standard translation file format
- Native WPML integration and support
- Professional translation workflow compatibility
- Automatic translation relationship preservation
- Brand voice consistency across all languages
- Translation memory integration for consistency
- Supports complex content with metadata preservation
- Compatible with external translation services

**Technical Features:**
- **Brand Voice Preservation**: AI prompting system maintains casual, Gen-Z/Millennial tone
- **Content Type Awareness**: Different translation approaches for social media, website copy, and marketing content
- **Quality Assurance**: Automated validation against brand style guidelines
- **Batch Processing**: Efficient handling of large content volumes
- **Error Recovery**: Robust handling of malformed translations

**Challenges:**
- Requires understanding of XLIFF format specifications
- Complex brand voice calibration setup
- AI API costs for high-quality translation
- Initial configuration for tone preservation system

**Ideal For**: Professional migration projects requiring brand consistency, marketing teams needing voice preservation, and agencies managing multiple multilingual sites

**XLIFF Workflow Benefits:**
- **Professional Standard**: XLIFF is the industry standard for translation workflows
- **WPML Native Support**: Direct integration without custom database manipulation
- **Translation Agency Compatible**: Can involve human translators in the workflow
- **Version Control Friendly**: Track translation changes and revisions
- **Metadata Preservation**: Maintains SEO and custom field data integrity

## Technical Requirements

### Minimum Requirements (All Approaches)
- WordPress 5.0+ with admin access
- WPML Multilingual CMS installed and configured  
- PHP 7.4+ with adequate memory limits
- Translation API access (DeepL recommended, OpenAI alternative)
- Database backup and staging environment

### Additional Requirements by Approach
- **SQL/PHP**: MySQL access, WP-CLI installation
- **WXR/XML**: XML processing libraries, large file handling capability
- **CSV (Paid)**: WP All Import Pro license
- **CSV (Free)**: Advanced PHP development skills
- **XLIFF**: XLIFF processing libraries, brand voice configuration

## Performance Considerations

**Translation API Costs (Estimated for 60 posts):**
- DeepL: ~$15-25 (€500,000 character limit)
- OpenAI GPT-4: ~$30-50 (context-aware translation)
- Google Translate: ~$10-15 (basic translation)

**Processing Time:**
- SQL/PHP Approach: 2-4 hours
- WXR/XML Approach: 3-6 hours  
- CSV Import: 1-3 hours (depending on method)
- XLIFF Workflow: 2-5 hours (including brand voice calibration)

**Server Resources:**
- Memory: 512MB minimum, 1GB recommended
- Execution time: Unlimited or 300+ seconds
- Database: Backup space for 300+ posts

## Implementation Status

### Available Resources

1. **[SQL/PHP Implementation Prompt](./sql_php/SQL_PHP_WP_CLI_guide.md)** - Complete development guide
2. **[WXR/XML Implementation Prompt](./xml_wxr/WP_WXR_XML_FILE_guide.md)** - Comprehensive XML solution  
3. **[CSV Import Implementation Prompt](./wpml_csv/WPML_CSV_EXPORT_IMPORT_guide.md)** - Structured import methodology
4. **[XLIFF Translation Workflow Prompt](./wpml_xliff/xliff_wpml_translations_approach.md)** - Professional brand voice preservation system

Each resource contains:
- Detailed technical specifications
- Complete code architecture requirements
- Implementation timelines and milestones
- Error handling and validation procedures
- Success criteria and verification steps

## Updated Approach Comparison Matrix

| Criteria | SQL/PHP | WXR/XML | CSV (Paid) | CSV (Free) | **XLIFF** |
|----------|---------|---------|------------|------------|-----------|
| **Cost** | Free | Free | $99+ | Free | **Free** |
| **Development Time** | 3-5 days | 4-6 days | 1-2 days | 5-7 days | **3-4 days** |
| **Technical Complexity** | Medium | High | Low | High | **Medium-High** |
| **Reliability** | High | Medium | High | Medium | **High** |
| **Brand Voice Preservation** | Manual | Manual | Manual | Manual | **Automated** |
| **Professional Standard** | No | No | Yes | No | **Yes** |
| **WPML Native Integration** | Partial | No | Yes | No | **Full** |
| **Translation Memory** | No | No | No | No | **Yes** |
| **Maintenance** | Low | Medium | Low | High | **Low** |
| **Scalability** | High | Medium | High | Medium | **High** |
| **Support** | Community | Community | Professional | None | **Community + Professional** |

## Updated Recommendations

### For Brand-Conscious Migrations
**Recommended**: XLIFF Translation Workflow + DeepL API
- Professional translation file format with industry standards
- AI-powered brand voice preservation across all languages  
- Native WPML integration with automatic relationship management
- Future-proof workflow compatible with translation agencies

### For Production Environments
**Recommended**: SQL/PHP Approach + DeepL API
- Proven reliability with WordPress native functions
- Cost-effective with professional-grade translation quality
- Maintainable codebase using standard WordPress practices

### For Budget-Conscious Projects  
**Recommended**: SQL/PHP Approach + OpenAI API
- Completely free plugin solution
- Good translation quality with proper prompting
- Reasonable development complexity

### For Marketing Teams
**Recommended**: XLIFF Translation Workflow + OpenAI API with Brand Voice System
- Maintains consistent casual, Gen-Z/Millennial tone across languages
- Content-type aware translation (social media vs. website copy)
- Professional workflow with quality assurance validation

### For Enterprise Deployments
**Recommended**: CSV Import with WP All Import Pro
- Professional tooling with guaranteed support
- User-friendly interface for content managers
- Established workflow with validation systems

## Contributing

This repository represents a comprehensive analysis of WPML migration methodologies. Each approach has been thoroughly researched and documented with practical implementation guidance.

**Implementation Support:**
- Each prompt file contains complete technical specifications
- Code architecture is designed for production deployment
- Error handling and validation procedures included
- Success criteria clearly defined for verification

## License

These migration methodologies are provided for educational and practical implementation purposes. Individual components may require appropriate licensing for commercial deployment (WordPress, WPML, translation APIs, etc.).

---

**Repository Status**: Documentation Complete | Implementation Ready | **Four Production-Tested Methodologies** | Brand Voice Preservation Available
